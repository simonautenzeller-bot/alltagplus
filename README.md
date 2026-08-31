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
- **Demo-Modus** – unter „Mehr“ umschaltbar: zeigt die App mit gefüllten Beispiel-Inhalten. Demo- und echte Daten liegen in getrennten Speichern, nach dem Beenden ist alles wieder wie vorher.
- **Offline** – Service Worker cached die App, alle Daten liegen im `localStorage` des Geräts.

## Lokal testen

Wegen Service Worker und Manifest am besten über einen kleinen Webserver statt per Doppelklick:

```powershell
cd AlltagPlus
python -m http.server 8080
# danach http://localhost:8080 im Browser öffnen
```

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
| `manifest.webmanifest` | PWA-Metadaten (Name, Icons, Farben) |
| `sw.js` | Service Worker für Offline-Betrieb |
| `icon.svg`, `icon-192.png`, `icon-512.png` | App-Icons |

## Datenhaltung

Alles bleibt lokal auf dem Gerät (`localStorage`, Schlüssel `AlltagPlus.v1`). Es gibt keine Synchronisierung zwischen Geräten – vor einem Gerätewechsel bitte unter „Mehr → Backup exportieren“ sichern. Die App ist ohne Anmeldung und ohne Verschlüsselung; sensible Passwörter also nur so speichern, wie du es auch in einer normalen Notiz-App tun würdest.

## Design

Warmes „Papier“-Thema: sandfarbener Hintergrund, Terracotta als Akzent, Serifen-Überschriften, kantige Karten mit versetztem Schatten. Bewusst anders als die dunkelblaue Finanzen-App. Dunkelmodus schaltet automatisch mit den Systemeinstellungen um.
