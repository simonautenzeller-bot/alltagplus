# Alltag+

Eine kleine Offline-App (PWA) für den Haushalt: tägliche, wöchentliche und monatliche Aufgaben zum Abhaken, ein Einkaufszettel und ein Merkzettel-Bereich für WLAN-Passwörter, Geschenkideen, Wunschlisten, Rezepte, Filme, Kleinanzeigen-Texte usw.

Reines HTML/CSS/JS ohne Build-Schritt, funktioniert direkt über GitHub Pages.

## Funktionen

- **Heute / Woche / Monat** – eigene Tabs, jeweils mit Fortschrittsbalken und Accordions pro Bereich (Küche, Bad, Finanzen …).
- **Automatischer Reset** – tägliche Häkchen fallen um Mitternacht weg, wöchentliche am Montag, monatliche am Monatsersten. Aufgaben bleiben erhalten.
- **Einkauf** – Schnelleingabe mit Mengen-Erkennung („2x Milch“), frei erweiterbare Abteilungen (Obst & Gemüse, Kühlregal, Drogerie …), Vorschläge aus dem, was oft gekauft wird, und ein „Im Wagen“-Bereich für Abgehaktes.
- **Gespeicherte Listen** – den aktuellen Zettel als Vorlage sichern (z. B. ein Rezept oder der Wocheneinkauf) und später per Tipp wieder auf den Zettel legen.
- **Merkzettel** – Notizen nach Kategorien gruppiert, mit Suche, Kategorie-Filter, Anpinnen und „Kopieren“-Button (praktisch für WLAN-Passwörter und Anzeigentexte). Zeilen mit `- ` werden als Aufzählung dargestellt, Zeilen mit `[ ] ` als abhakbare Checkliste inklusive Fortschrittsanzeige.
- **Backup** – Export/Import als JSON unter „Mehr“.
- **Wochen-Backup** – einmal pro Woche (beim Öffnen der App) ein Backup außerhalb des Browsers, denn gelöschte Browser-Daten nehmen sonst auch die App-Daten mit. Chrome/Edge am Computer: einmal einen Ordner wählen (z. B. im OneDrive-Ordner), danach schreibt die App automatisch `AlltagPlus-auto-JJJJ-MM-TT.json` hinein, die letzten 12 bleiben. Handy/andere Browser: Erinnerung, dann per „Teilen“ → OneDrive. Fehlen beim Start alle Daten, bietet die App direkt das Einspielen eines Backups an.
- **Demo-Modus** – unter „Mehr“ umschaltbar: zeigt die App mit gefüllten Beispiel-Inhalten. Demo- und echte Daten liegen in getrennten Speichern, nach dem Beenden ist alles wieder wie vorher.
- **Offline** – Service Worker cached die App, alle Daten liegen im `localStorage` des Geräts.

## Lokal testen

Wegen Service Worker und Manifest am besten über einen kleinen Webserver statt per Doppelklick:

```powershell
cd AlltagPlus
python -m http.server 8080
# danach http://localhost:8080 im Browser öffnen
```

## OneDrive-Upload einrichten (optional, privates Microsoft-Konto)

Ohne diese Einrichtung funktioniert das Wochen-Backup trotzdem: Am Handy erinnert die App einmal pro Woche, „Jetzt sichern“ öffnet das Teilen-Menü (Android: OneDrive wählen, iPhone: „In Dateien sichern“ → OneDrive), am Computer genügt der Backup-Ordner. Der direkte Upload braucht eine eigene App-Registrierung bei Microsoft. Dafür muss man sich bei entra.microsoft.com anmelden können; ein privates Microsoft-Konto allein reicht dafür oft nicht, es braucht zusätzlich ein (kostenloses) Azure-Konto unter azure.microsoft.com/free, bei dem eine Kreditkarte nur zur Prüfung hinterlegt wird.

Einmalig, am besten am Computer:

1. https://entra.microsoft.com öffnen und mit dem privaten Microsoft-Konto anmelden → **App-Registrierungen → Neue Registrierung**.
2. Name z. B. `Alltag+ Backup` (so heißt später der Ordner unter OneDrive/Apps). **Unterstützte Kontotypen: „Konten in einem beliebigen Organisationsverzeichnis (beliebiger Microsoft Entra-Mandant – mehrinstanzenfähig) und persönliche Microsoft-Konten“.** Die Standardauswahl („Nur dieses Organisationsverzeichnis“) führt zu der Meldung „Das Benutzerkonto ist im Mandanten ‚Microsoft Services‘ nicht vorhanden“. Eine bestehende Registrierung korrigierst du unter **Authentifizierung → Einstellungen → Unterstützte Kontotypen** (oder im **Manifest**: `"signInAudience": "AzureADandPersonalMicrosoftAccount"`). Melde dich beim Registrieren mit dem **privaten** Konto an, nicht mit dem Firmenkonto; sonst entsteht die Registrierung im Firmenmandanten.
3. **Umleitungs-URIs**: Plattform **Single-Page-Anwendung (SPA)**. Eine einzige Registrierung genügt für alle Apps, trage einfach jede Adresse ein (genau so, mit Schrägstrich am Ende):
   - `https://simonautenzeller-bot.github.io/alltagplus/`
   - `https://simonautenzeller-bot.github.io/hausplaner/`
   - `https://simonautenzeller-bot.github.io/finanzen-app/`
   - `https://simonautenzeller-bot.github.io/watchdex/`
   - `https://simonautenzeller-bot.github.io/questlog/`
4. Registrieren und die **Anwendungs-(Client-)ID** kopieren. Berechtigungen müssen nicht eingetragen werden, die Zustimmung kommt beim ersten Anmelden.
5. In der App: **Mehr → OneDrive (privat)** → Client-ID einfügen → „Mit OneDrive verbinden“. Die Client-ID merkt sich der Browser für alle Apps unter derselben Adresse, in den anderen Apps muss dann nur noch „Mit OneDrive verbinden“ getippt werden. Jede App schreibt ihre Dateien mit eigenem Namensanfang (`AlltagPlus-auto-…`, `Finanzen-auto-…` usw.) in denselben OneDrive-App-Ordner.

Danach lädt die App ein fälliges Wochen-Backup beim Öffnen hoch (kurzer Wechsel zu Microsoft und zurück). Klappt die stille Anmeldung nicht, erscheint der Hinweis „Backup fällig“ mit einem Tipp-Button. Die Dateien landen in `OneDrive/Apps/<App-Name>/`, es wird nur dieser App-Ordner freigegeben (Berechtigung `Files.ReadWrite.AppFolder`). Wer die ID fest einbauen will, trägt sie in `index.html` bei `ONEDRIVE_CLIENT_ID` ein, dann übersteht sie auch gelöschte Browserdaten.

### Gemeinsames Backup-Modul (`backup.js`)

Die komplette Backup-Logik liegt in `backup.js`, einer Datei ohne Abhängigkeiten, die identisch in alltagplus, hausplaner, finanzen-app, watchdex und questlog liegt. Jede App bindet sie mit `AppBackup.init({ ... })` ein und liefert nur:

| Option | Zweck |
| --- | --- |
| `appId`, `appName`, `filePrefix` | Namensräume für Speicher-Schlüssel und Dateinamen |
| `getData()` | liefert die zu sichernden Daten, `null` pausiert das Backup (z. B. im Demo-Modus) |
| `restore(text)` | spielt ein Backup ein, gibt `true`/`false` zurück |
| `isFresh()` | `true`, wenn die App ohne gespeicherte Daten gestartet ist |
| `bannerParent` / `bannerBefore` / `bannerClass` | wo der Hinweis erscheint |
| `btnClass`, `btnPrimaryClass`, `inputClass` | Klassen der App für Buttons und Eingabefeld |

`AppBackup.createPanel()` liefert das Einstellungs-Panel, `AppBackup.markExternal("Export")` verbucht ein manuelles Backup, `AppBackup.refresh()` aktualisiert die Anzeige. **Wird `backup.js` geändert, bitte die Kopie in alle Repos übernehmen.**

## Auf GitHub Pages veröffentlichen

1. Neues Repository anlegen, z. B. `AlltagPlus`.
2. Inhalt dieses Ordners committen und pushen:

   ```powershell
   git init
   git add .
   git commit -m "Alltag+"
   git branch -M main
   git remote add origin https://github.com/<user>/AlltagPlus.git
   git push -u origin main
   ```

3. Im Repo: **Settings → Pages → Source: Deploy from a branch → `main` / `root`**.
4. Nach ein paar Minuten liegt die App unter `https://<user>.github.io/AlltagPlus/`.

## Aufs Handy

Seite im Browser öffnen → Menü → **„Zum Startbildschirm hinzufügen“**. Danach startet sie wie eine native App im Vollbild und läuft auch ohne Internet.

## Dateien

| Datei | Zweck |
| --- | --- |
| `index.html` | Komplette App (Markup, Styles, Logik) |
| `backup.js` | Wochen-Backup (Ordner, OneDrive, Teilen), gemeinsames Modul aller Apps |
| `manifest.webmanifest` | PWA-Metadaten (Name, Icons, Farben) |
| `sw.js` | Service Worker für Offline-Betrieb |
| `icon.svg`, `icon-192.png`, `icon-512.png` | App-Icons |

## Datenhaltung

Alles bleibt lokal auf dem Gerät (`localStorage`, Schlüssel `AlltagPlus.v1`). Achtung: `localStorage` gehört zur Origin (`https://<user>.github.io`), nicht zum Pfad – alle Apps unter derselben GitHub-Pages-Adresse teilen sich also einen Speicher und verlieren ihn gemeinsam, wenn man „Websitedaten löschen“ wählt. Jede App braucht deshalb eigene Schlüsselnamen und ein eigenes Backup. Es gibt keine Synchronisierung zwischen Geräten – vor einem Gerätewechsel bitte unter „Mehr → Backup exportieren“ sichern. Die App ist ohne Anmeldung und ohne Verschlüsselung; sensible Passwörter also nur so speichern, wie du es auch in einer normalen Notiz-App tun würdest.

## Design

Warmes „Papier“-Thema: sandfarbener Hintergrund, Terracotta als Akzent, Serifen-Überschriften, kantige Karten mit versetztem Schatten. Bewusst anders als die dunkelblaue Finanzen-App. Dunkelmodus schaltet automatisch mit den Systemeinstellungen um.
