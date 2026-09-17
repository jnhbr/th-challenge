# Tom Holland Challenge · Tracker mit Duell

Der vollständige Tracker für die **Tom Holland Challenge** (20 Min AMRAP: 5 Pull-Ups,
10 Push-Ups, 15 Air Squats pro Runde) – Dashboard, Workout-Erfassung, Aufwärm-Assistent,
AMRAP-Timer, Charts, Kalender, Rekorde, Abzeichen, Seasons – erweitert um einen
**Duell-Modus**, in dem sich bis zu sechs Athleten live vergleichen.

Eine einzelne `index.html`, kein Build. Gehostet auf GitHub Pages, das Duell läuft über
Firebase Firestore.

## Ohne Firebase

Der Tracker funktioniert vollständig ohne jede Einrichtung – alle Daten liegen wie bisher
im LocalStorage des Browsers. Nur der Duell-Tab bleibt gesperrt, bis eine Firebase-Config
hinterlegt ist.

## Setup für das Duell

1. **Projekt anlegen** – [console.firebase.google.com](https://console.firebase.google.com) →
   *Projekt hinzufügen* (z. B. `th-challenge`). Google Analytics kann man abwählen.
   Der kostenlose Spark-Tarif reicht vollständig aus.

2. **Web-App registrieren** – im Projekt auf `</>` klicken, Name vergeben,
   **kein** Firebase Hosting aktivieren. Die Console zeigt dann den `firebaseConfig`-Block.

3. **Config eintragen** – die sechs Werte in [`index.html`](index.html) bei
   `FIREBASE_CONFIG` ersetzen (ganz oben im `<script>`, ausführlich kommentiert).

4. **Anonyme Anmeldung einschalten** – *Authentication* → *Sign-in method* → **Anonym**.
   Davon merkt man auf der Seite nichts; sie sorgt nur dafür, dass die Regeln greifen.

5. **Firestore anlegen und Regeln setzen** – *Firestore Database* → *Datenbank erstellen*
   (Produktionsmodus, Region `eur3`) → Reiter *Regeln* → Inhalt von
   [`firestore.rules`](firestore.rules) einfügen → *Veröffentlichen*.

Danach committen; GitHub Pages baut automatisch neu.

## Duell benutzen

Im Tab **⚔️ Duell**: *Neuen Code erzeugen* → Athlet A wählen → beitreten. Den Code den
Kumpels schicken; bis zu sechs Athleten (A–F) passen in ein Duell. Die bisherigen Workouts
werden automatisch mitgenommen.

Die Athleten-Plätze sind gesperrt, bis ein Code eingegeben ist. Danach zeigt die Seite, welche
Plätze schon vergeben sind; wählbar sind nur freie – oder der eigene, wenn derselbe Name
eingegeben wird (= zweites Gerät derselben Person). Beim Beitreten wird der Platz in einer
Firestore-Transaktion geprüft und belegt: treten zwei Personen gleichzeitig auf denselben Platz
bei, bekommt ihn nur eine, die andere erhält eine Meldung und einen freien Platz vorgeschlagen.

Bei zwei Athleten erscheint oben das klassische VS, ab drei eine Rangliste nach Bestleistung.
Direktvergleich, Weg zu Tom Holland, Verlauf und letzte Workouts zeigen alle Teilnehmer.
Über *Einladung teilen* gibt es einen Link mit Code (`…/index.html#THXXXX`), der das Feld
beim Gegner schon ausfüllt.

## Angefangene Runden

Eine Runde sind 5 Pull-Ups, 10 Push-Ups und 15 Air Squats – und wenn der Gong kommt, ist man
selten genau am Rundenende. Beim Erfassen stehen deshalb neben den **vollen Runden** drei
kleine Felder für die **angefangene Runde**: was nach der letzten vollen Runde noch geschafft
wurde (z. B. 5 Pull-Ups und 3 Push-Ups in der 11. Runde). Derselbe Block steckt im Dialog,
der nach dem AMRAP-Timer aufgeht.

Notiert wird das wie im CrossFit als `10+8` – zehn volle Runden plus acht Wiederholungen.
Gerechnet wird damit überall: Wiederholungen zählen exakt, und für Rekord, Schnitt, Charts
und Duell zählt eine angefangene Runde anteilig (8 von 30 Wiederholungen = 0.27 Runden).
Ältere Einträge mit halben Runden (`11.5`) bleiben gültig und werden unverändert
weitergerechnet.

## Workouts nachträglich korrigieren

Jeder Eintrag im Verlauf hat neben dem Papierkorb einen Stift: der öffnet denselben
Dialog wie beim Erfassen, nur mit den gespeicherten Werten – Datum, volle Runden,
angefangene Runde und Notiz lassen sich ändern. ID, Season und Erstellzeit bleiben,
damit es derselbe Eintrag bleibt.

Eine Korrektur trägt einen Zeitstempel (`u`), und beim Zusammenführen gewinnt der
neuere Stand derselben ID. Ohne das bliebe eine Korrektur auf dem Gerät hängen, auf
dem sie gemacht wurde, weil beim Abgleich sonst nur unbekannte IDs dazukommen.

## Konto: ein Code für die ganze App

Beim ersten Öffnen fragt die Seite nach einem **Zugangscode** – selbst ausgedacht (mindestens
6 Zeichen) oder per Knopfdruck zufällig erzeugt. Der Code ist Login und Passwort in einem.

Mit demselben Code auf Handy und Laptop angemeldet, gleicht sich **der gesamte Tracker** ab:
Workouts, Seasons, Abzeichen, Einstellungen und das Farbschema. Wer keinen Code will, wählt
*Ohne Konto, nur auf diesem Gerät* – dann verhält sich die App wie vorher, rein lokal.

Zusammengeführt wird so:

- **Workouts** – Vereinigung über die Workout-ID. Gelöschte bleiben gelöscht (Liste gelöschter
  IDs). Gleiches Datum bei gleicher Rundenzahl gilt als derselbe Eintrag, damit ein versehentlich
  doppelt erfasstes Training nicht zweimal erscheint. Bei gleicher ID gewinnt die neuere
  Korrektur (siehe oben).
- **Abzeichen** – Vereinigung, das frühere Freischaltdatum gewinnt.
- **Seasons** – fehlende werden ergänzt.
- **Einstellungen, aktuelle Season, Theme** – der zuletzt gespeicherte Stand gewinnt.

Der Zugangscode ist **privat**. Der Duell-Code ist ein anderer und wird mit dem Gegner geteilt –
über ihn sieht der andere nur die gespiegelten Workouts, nicht dein Konto.

## Mehrere Geräte

Für das Duell melden sich Handy und Laptop **mit demselben Duell-Code und demselben
Athleten-Slot** an – dann gehören sie zum selben Athleten:

- Beide Geräte schreiben in dasselbe Athleten-Dokument und **führen ihre Workouts zusammen**
  (Vereinigung über die Workout-ID), statt sich zu überschreiben.
- Ein auf dem Handy erfasstes Workout taucht auf dem Laptop auf, sobald der die Seite offen
  hat oder neu lädt – und umgekehrt.
- Löschungen werden über eine Liste gelöschter IDs übertragen, damit ein gelöschtes Workout
  nicht vom anderen Gerät zurückkommt.

Seasons, Abzeichen und Einstellungen laufen über das Konto (siehe oben), nicht über das Duell.
Workouts, die zu einer auf dem anderen Gerät unbekannten Season gehören, landen dort in der
aktuellen Season.

Wichtig: Zwei verschiedene Personen im selben Slot würden sich gegenseitig überschreiben.
Die Seite verhindert das beim Beitreten (siehe oben).

## Datenmodell

```
athletes/{zugangscode}        { name, updated, workouts:[{i,d,r,n,c,s,p,u}], removed:[id],
                                seasons, currentSeasonId, badges, settings, theme }
duels/{duellcode}/athletes/{a|b|c|d|e|f}
                              { name, workouts:[{i,d,r,n,c,s,p,u}], removed:[id], updated }
```

`i` = Workout-ID, `d` = Datum, `r` = volle Runden, `n` = Notiz, `c` = Erstellzeit,
`s` = Season-ID, `p` = angefangene Runde als `[Pull-Ups, Push-Ups, Squats]` (fehlt, wenn
die Runde voll ausging), `u` = Zeitpunkt der letzten Korrektur (fehlt, solange nie
bearbeitet wurde).

## Sicherheit

Der Challenge-Code ist das einzige Geheimnis. Die Regeln verhindern, dass Fremde die
Datenbank durchsuchen oder Unsinn hineinschreiben – wer den Code kennt, kann die Workouts
des Duells aber lesen und ändern. Für Rundenzahlen zweier Kumpels ist das angemessen.

Der API-Key im Quelltext ist kein Passwort: Firebase-Web-Keys sind öffentlich und dürfen
im Repo stehen. Den Schutz leisten allein die Firestore-Regeln.
