# Tom Holland Duell

Zwei Athleten, eine Challenge: **20 Minuten AMRAP** mit 5 Pull-Ups, 10 Push-Ups und 15 Air Squats
pro Runde. Beide tragen ihre Workouts ein und sehen den Stand des anderen in Echtzeit.

Eine einzelne `index.html` – kein Build, kein npm. Gehostet auf GitHub Pages,
Daten in Firebase Firestore.

## Setup in 5 Schritten

Solange keine Firebase-Konfiguration hinterlegt ist, läuft die Seite im **Offline-Modus**:
alles funktioniert, die Daten bleiben aber im eigenen Browser. Für das Duell braucht es Firebase.

1. **Projekt anlegen** – [console.firebase.google.com](https://console.firebase.google.com) →
   *Projekt hinzufügen* (z. B. `th-challenge`). Google Analytics kann man abwählen.
   Der kostenlose Spark-Tarif reicht vollständig aus.

2. **Web-App registrieren** – im Projekt auf das Symbol `</>` klicken, Name vergeben,
   **kein** Firebase Hosting aktivieren. Danach zeigt die Console den `firebaseConfig`-Block.

3. **Config eintragen** – die sechs Werte oben in [`index.html`](index.html) bei
   `FIREBASE_CONFIG` ersetzen (Zeile ~471, gut sichtbar kommentiert).

4. **Anonyme Anmeldung einschalten** – *Authentication* → *Sign-in method* →
   **Anonym** aktivieren. Davon merkt man auf der Seite nichts; sie sorgt nur dafür,
   dass die Sicherheitsregeln greifen.

5. **Firestore anlegen und Regeln setzen** – *Firestore Database* → *Datenbank erstellen*
   (Produktionsmodus, Region `eur3`) → Reiter *Regeln* → Inhalt von
   [`firestore.rules`](firestore.rules) einfügen → *Veröffentlichen*.

Danach `index.html` committen. GitHub Pages baut automatisch neu.

## Bedienung

- Beim ersten Öffnen: **Neue Challenge starten** erzeugt einen Code (z. B. `THNQNJ`).
  Den Code dem Gegner schicken – er gibt ihn ein, wählt den anderen Athleten-Slot und ist dabei.
- Über *Mehr → Einladung teilen* gibt es einen Link mit Code (`…/index.html#THNQNJ`),
  der das Feld beim Gegner schon ausfüllt.
- Workouts aus dem lokalen Tracker lassen sich unter *Mehr → JSON importieren* übernehmen;
  doppelte Einträge (gleiches Datum, gleiche Rundenzahl) werden übersprungen.
- Löschen darf jeder nur die eigenen Einträge.

## Datenmodell

```
duels/{code}                    { nameA, nameB, touched }
duels/{code}/workouts/{id}      { slot: "a"|"b", date: "YYYY-MM-DD",
                                  rounds: number, notes: string, ts: number }
```

## Sicherheit – was das Setup leistet und was nicht

Der Challenge-Code ist das einzige Geheimnis. Die Regeln verhindern, dass Fremde die
Datenbank durchsuchen oder Unsinn hineinschreiben, aber wer den Code kennt, kann die
Workouts des Duells lesen und ändern. Für Rundenzahlen zweier Kumpels ist das angemessen –
für alles Persönlichere wäre ein echter Login nötig.

Der API-Key im Quelltext ist kein Passwort: Firebase-Web-Keys sind öffentlich und
dürfen im Repo stehen. Den Schutz leisten allein die Firestore-Regeln.
