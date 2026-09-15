# Tom Holland Challenge · Tracker mit Duell

Der vollständige Tracker für die **Tom Holland Challenge** (20 Min AMRAP: 5 Pull-Ups,
10 Push-Ups, 15 Air Squats pro Runde) – Dashboard, Workout-Erfassung, Aufwärm-Assistent,
AMRAP-Timer, Charts, Kalender, Rekorde, Abzeichen, Seasons – erweitert um einen
**Duell-Modus**, in dem sich zwei Athleten live vergleichen.

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

Im Tab **⚔️ Duell**: *Neuen Code erzeugen* → Athlet A wählen → beitreten. Den Code dem
Gegner schicken, der wählt Athlet B. Die bisherigen Workouts werden automatisch mitgenommen.
Über *Einladung teilen* gibt es einen Link mit Code (`…/index.html#THXXXX`), der das Feld
beim Gegner schon ausfüllt.

## Mehrere Geräte

Handy und Laptop melden sich **mit demselben Code, demselben Athleten-Slot und demselben
Namen** an – dann gehören sie zum selben Athleten und gleichen sich gegenseitig ab:

- Beide Geräte schreiben in dasselbe Athleten-Dokument und **führen ihre Workouts zusammen**
  (Vereinigung über die Workout-ID), statt sich zu überschreiben.
- Ein auf dem Handy erfasstes Workout taucht auf dem Laptop auf, sobald der die Seite offen
  hat oder neu lädt – und umgekehrt.
- Löschungen werden über eine Liste gelöschter IDs übertragen, damit ein gelöschtes Workout
  nicht vom anderen Gerät zurückkommt.

Nicht synchronisiert werden **Seasons, Abzeichen und Einstellungen** – die bleiben pro Gerät.
Workouts, die zu einer auf dem anderen Gerät unbekannten Season gehören, landen dort in der
aktuellen Season.

Wichtig: Wählt der Gegner denselben Slot wie du, überschreibt ihr euch gegenseitig.
A und B müssen unterschiedlich vergeben sein.

## Datenmodell

```
duels/{code}/athletes/{a|b}   { name, workouts:[{i,d,r,n,c,s}], removed:[id], updated }
```

`i` = Workout-ID, `d` = Datum, `r` = Runden, `n` = Notiz, `c` = Erstellzeit, `s` = Season-ID.

## Sicherheit

Der Challenge-Code ist das einzige Geheimnis. Die Regeln verhindern, dass Fremde die
Datenbank durchsuchen oder Unsinn hineinschreiben – wer den Code kennt, kann die Workouts
des Duells aber lesen und ändern. Für Rundenzahlen zweier Kumpels ist das angemessen.

Der API-Key im Quelltext ist kein Passwort: Firebase-Web-Keys sind öffentlich und dürfen
im Repo stehen. Den Schutz leisten allein die Firestore-Regeln.
