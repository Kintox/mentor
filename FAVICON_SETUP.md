# 🐾 Favicon-Setup – Anleitung für Cedric

Diese Anleitung erklärt, wie das Reico-Favicon (Tierpfote mit Herz) korrekt in allen
Browser-Tabs von **cedricnitsch.de** angezeigt wird.

---

## 1. Welche Dateien müssen auf den Server?

Alle Favicon-Dateien sind bereits im Projekt enthalten. Beim Upload auf den
Production-Server (bzw. beim Deploy über GitHub Pages) müssen sie an die **richtige
Stelle**:

| Datei | Ziel auf dem Server | Öffentliche URL |
|-------|--------------------|-----------------|
| `favicon.ico` | **Root-Verzeichnis** | `https://cedricnitsch.de/favicon.ico` |
| `site.webmanifest` | **Root-Verzeichnis** | `https://cedricnitsch.de/site.webmanifest` |
| `images/favicon-16.png` | Ordner `/images/` | `https://cedricnitsch.de/images/favicon-16.png` |
| `images/favicon-32.png` | Ordner `/images/` | `https://cedricnitsch.de/images/favicon-32.png` |
| `images/favicon-180.png` | Ordner `/images/` | `https://cedricnitsch.de/images/favicon-180.png` |
| `images/favicon-192.png` | Ordner `/images/` | `https://cedricnitsch.de/images/favicon-192.png` |
| `images/favicon-512.png` | Ordner `/images/` | `https://cedricnitsch.de/images/favicon-512.png` |

> **Wichtig:** `favicon.ico` **muss** direkt im Root liegen – nicht im `/images/`-Ordner.
> Browser suchen automatisch nach `https://cedricnitsch.de/favicon.ico`.

Bei **GitHub Pages** passiert das automatisch: Ein `git push` deployt alle Dateien an
die richtige Stelle. Es ist kein manueller Upload nötig.

---

## 2. Einbindung im HTML (bereits erledigt)

In **allen** HTML-Seiten steht direkt nach `<meta charset="UTF-8">` folgender Block –
mit **absoluten Pfaden** (führender `/`) für maximale Kompatibilität:

```html
<!-- Favicons (absolute Pfade für maximale Kompatibilität) -->
<link rel="icon" type="image/x-icon" href="/favicon.ico">
<link rel="icon" type="image/png" sizes="32x32" href="/images/favicon-32.png">
<link rel="icon" type="image/png" sizes="16x16" href="/images/favicon-16.png">
<link rel="icon" type="image/png" sizes="192x192" href="/images/favicon-192.png">
<link rel="apple-touch-icon" href="/images/favicon-180.png">
<link rel="manifest" href="/site.webmanifest">
```

**Warum absolute Pfade (`/favicon.ico`) statt relativer (`favicon.ico`)?**
Absolute Pfade funktionieren zuverlässig von jeder Unterseite aus und vermeiden Fehler,
wenn Seiten in Unterordnern liegen.

---

## 3. Test der Einbindung

Es gibt eine eigene Test-Seite:

**➡️ https://cedricnitsch.de/favicon-test.html**

Diese Seite zeigt:
- eine direkte Vorschau aller Favicon-Bilder,
- einen automatischen Erreichbarkeits-Check (grün = OK, rot = Fehler),
- die Reico-Pfote im Browser-Tab.

Sind alle Punkte grün, sind die Dateien korrekt auf dem Server.

---

## 4. Häufigstes Problem: Browser-Cache

Favicons werden von Browsern **sehr aggressiv zwischengespeichert**. Nach einem Update
kann es sein, dass das alte (oder gar kein) Favicon angezeigt wird.

**Lösung:**
1. **Hard-Reload** der Seite:
   - Windows / Linux: **`Strg` + `F5`** oder **`Strg` + `Shift` + `R`**
   - Mac: **`Cmd` + `Shift` + `R`**
2. Seite in einem **Inkognito-/Privat-Fenster** öffnen (nutzt keinen Cache).
3. Notfalls Browser-Cache komplett leeren (Einstellungen → Verlauf/Cache löschen).
4. Manchmal hilft es, den Tab komplett zu schließen und neu zu öffnen.

> Nach einem Deploy kann es **einige Minuten bis Stunden** dauern, bis Browser und CDN
> das neue Favicon zeigen. Das ist normal.

---

## 5. Troubleshooting

| Symptom | Ursache | Lösung |
|---------|---------|--------|
| Kein Favicon im Tab | Cache | Hard-Reload (Strg+F5), Inkognito-Fenster |
| `favicon-test.html` zeigt rote Fehler | Dateien nicht (am richtigen Ort) hochgeladen | Upload prüfen (siehe Tabelle Abschnitt 1) |
| `favicon.ico` liefert 404 | Datei nicht im Root | `favicon.ico` ins Root-Verzeichnis legen |
| Favicon auf iPhone-Homescreen fehlt | `apple-touch-icon` fehlt/404 | `images/favicon-180.png` prüfen |
| Altes Favicon bleibt sichtbar | Aggressiver Browser-Cache | Cache leeren, ggf. 24h warten |

---

## 6. Design-Info

- **Motiv:** Tierpfote mit Herz
- **Farben:** Alge `#3A3D0C`, Kristall `#FF9888`, Hintergrund `#F7FAE0`
- **Hintergrund:** transparent (funktioniert auf hellen & dunklen Browser-Themes)
- **Formate:** `.ico` (Legacy-Browser) + `.png` in mehreren Größen (moderne Browser, PWA)
