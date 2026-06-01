# 🔄 Futtercheck Komplett-Umbau – Dokumentation

> **Stand:** 01.06.2026 | **Branch:** `recruiting-funnel-optimization`
> **Letztes Update:** Natives Brevo-Formular für DOI – API entfernt, alle Attribute als Formularfelder

---

## 1. Was wurde geändert?

### Dateien

| Datei | Änderung |
|---|---|
| `futtercheck.html` | **Komplett neu geschrieben** – Quiz + Lead-Formular + DOI via natives Brevo-Formular (sibforms, versteckt im DOM) |
| `index.html` | Brevo-Formular entfernt → durch saubere CTA-Card mit Link zu `futtercheck.html` ersetzt |
| `bestaetigung-danke.html` | **Neu** – DOI-Bestätigungsseite nach Klick auf den E-Mail-Link (Reico-Design) |
| `mails/doi_bestaetigung.html` | **Neu** – DOI-Mail-Vorlage für Brevo (Transactional Template) |
| `.nojekyll` | **Neu** – Deaktiviert Jekyll für GitHub Pages (behebt Liquid-Syntax-Fehler) |
| `DOI_INTEGRATION_FINAL.md` | **Neu** – Vollständige technische Doku der finalen DOI-Integration |

### Vorher → Nachher

| Vorher | Nachher |
|---|---|
| Brevo-Opt-in-Formular **vor** dem Futtercheck | Futtercheck **direkt erreichbar** – kein Vorformular |
| Lead-Daten am Anfang abgefragt | Lead-Formular **am Ende** nach allen Quiz-Fragen (höhere Conversion) |
| Ergebnis erst nach DOI-Bestätigung | **Hybrid-Teaser**: Score sofort sichtbar, ausführliche Analyse per DOI-E-Mail |
| Kein Partner-Akquise-Element | **Partner-Bridge** Block auf der Ergebnisseite mit eigenem CTA |
| Alte Attributnamen (TIERART, FUTTERCHECK_ERGEBNIS) | **Finale Brevo-Attribute** (HUND_KATZE, FUTTERCHECK_FARB_SCORE, etc.) |

---

## 2. Neuer Flow (Schritt für Schritt)

```
Landing Page (index.html)
    ↓ "Jetzt Futtercheck starten" Button
Futtercheck (futtercheck.html)
    ↓ Startseite mit Paw-Icon
    ↓ Frage 1:  Tierart (Hund/Katze)
    ↓ Frage 2:  Name des Tieres
    ↓ Frage 3:  Alter
    ↓ Frage 4:  Gewicht
    ↓ Frage 5:  Aktivitätslevel
    ↓ Frage 6:  Futterart
    ↓ Frage 7:  Symptome (Multi-Select)
    ↓ Frage 8:  Allergien
    ↓ Frage 9:  Leckerlis
    ↓ Frage 10: Zufriedenheit
    ↓ Frage 11: Partner-Interesse ("Noch eine letzte Frage...")
    ↓ Frage 12: Vorname
    ↓ Frage 13: Lead-Formular (E-Mail*, Telefon optional, DSGVO*)
    ↓ "Ergebnis anzeigen" Button
Ergebnisseite (gleiche Datei, dynamisch)
    ├── Score-Kreis (farbcodiert: grün/gelb/rot)
    ├── Kurzbewertung + Tipps
    ├── Geblurrte "Ausführliche Auswertung" (DOI-Teaser)
    ├── WhatsApp-CTA (Kunde): "Jetzt per WhatsApp beraten lassen"
    └── Partner-Bridge Block
        ├── Persönliche Story von Cedric
        ├── 4 Benefit-Cards
        └── Partner-CTA: "Mehr über die Partnerschaft erfahren"

    → Verstecktes Brevo-Formular wird programmatisch befüllt
    → sibforms main.js macht AJAX POST an sibforms.com
    → Brevo sendet DOI-Bestätigungsmail
    → Nutzer klickt Bestätigungslink → Kontakt wird aktiviert → Automation startet
```

---

## 3. Brevo-Integration (FINALE STRUKTUR)

### API-Anbindung (Natives Brevo-Formular)

- **Verstecktes Brevo-Formular** (sibforms.com): Wird programmatisch befüllt und abgesendet
- **sibforms main.js**: Fängt Submit ab, macht AJAX POST, löst DOI aus
- **Kein API-Key im Frontend** – Authentifizierung läuft über die Formular-Action-URL
- **Alle 8 Custom-Attribute** direkt als Formularfelder konfiguriert
- **DOI-Template** wird in den Brevo-Formular-Einstellungen konfiguriert

> ℹ️ Für die vollständige technische Dokumentation siehe `DOI_INTEGRATION_FINAL.md`

### Formular-Daten (werden programmatisch gesetzt)

```
EMAIL          = max@beispiel.de
VORNAME        = Max
HUND_KATZE     = hund
TIERNAME       = Bello
FUTTERCHECK_SCORE      = 72
FUTTERCHECK_FUTTER     = Trockenfutter (Standard)
PARTNER_INTERESSE      = Stark
FUTTERCHECK_FARB_SCORE = gelb
SMS            = 015678516818
SMS__COUNTRY_CODE = +49
email_address_check =          (Honeypot – leer!)
locale         = de

```

### Kontakt-Attribute (in Brevo anzulegen)

> ℹ️ **Alle Attribute sind jetzt TEXT** – keine Kategorie-Attribute mehr nötig. Das vereinfacht die Einrichtung erheblich.

| Attribut-ID | Typ | Mögliche Werte | Beschreibung |
|---|---|---|---|
| `VORNAME` | **Text** | Freitext | Vorname des Leads |
| `HUND_KATZE` | **Text** | `hund`, `katze` | Tierart (Kleinschreibung!) |
| `TIERNAME` | **Text** | Freitext | Name des Tieres |
| `FUTTERCHECK_SCORE` | **Zahl** | `0`–`100` | Numerischer Futter-Score |
| `FUTTERCHECK_FUTTER` | **Text** | Freitext | Aktuelles Futter (lesbarer Text) |
| `PARTNER_INTERESSE` | **Text** | `Stark`, `Leicht`, `Kunde` | Interesse an Partnerschaft |
| `FUTTERCHECK_FARB_SCORE` | **Text** | `gruen`, `gelb`, `rot` | Ampel-Bewertung des Scores |
| `SMS` | **Text** | Freitext (optional) | Telefonnummer |

### Mapping: Quiz-Antwort → Brevo-Wert

| Quiz-Feld | Quiz-Antwort | → Brevo-Attribut | → Brevo-Wert |
|---|---|---|---|
| Tierart (Q1) | Hund / Katze | `HUND_KATZE` | `hund` / `katze` |
| Partner-Interesse (Q11) | "Ja, klingt spannend" | `PARTNER_INTERESSE` | `Stark` |
| Partner-Interesse (Q11) | "Vielleicht" | `PARTNER_INTERESSE` | `Leicht` |
| Partner-Interesse (Q11) | "Nein danke" | `PARTNER_INTERESSE` | `Kunde` |
| Score ≥ 75 | automatisch | `FUTTERCHECK_FARB_SCORE` | `gruen` |
| Score 40–74 | automatisch | `FUTTERCHECK_FARB_SCORE` | `gelb` |
| Score < 40 | automatisch | `FUTTERCHECK_FARB_SCORE` | `rot` |
| Aktuelles Futter (Q5) | z.B. "trockenfutter_standard" | `FUTTERCHECK_FUTTER` | `Trockenfutter (Standard)` |

### Futter-Wert-Mapping (intern → Brevo)

| Quiz-Value | → FUTTERCHECK_FUTTER |
|---|---|
| `nassfutter_premium` | Hochwertiges Nassfutter |
| `barf` | BARF / Rohfütterung |
| `selbstgekocht` | Selbstgekocht |
| `trockenfutter_premium` | Trockenfutter (Premium) |
| `trockenfutter_standard` | Trockenfutter (Standard) |
| `misch` | Mischfütterung (Nass + Trocken) |
| `unsicher` | Unsicher / wechselt oft |

### Tag-Logik (bereinigt)

Tags werden **nicht** mehr über die Brevo-API gesendet (Brevo REST v3 `/contacts` unterstützt kein `tags`-Feld).
Die Segmentierung erfolgt ausschließlich über die **Kontakt-Attribute**:

| Altes Tag | Ersetzt durch Attribut |
|---|---|
| `hund` / `katze` | `HUND_KATZE` = `hund` / `katze` |
| `score_gruen` / `score_gelb` / `score_rot` | `FUTTERCHECK_FARB_SCORE` = `gruen` / `gelb` / `rot` |
| `partner_interesse_stark` | `PARTNER_INTERESSE` = `Stark` |
| `partner_interesse_leicht` | `PARTNER_INTERESSE` = `Leicht` |
| `nur_kunde` | `PARTNER_INTERESSE` = `Kunde` |
| `FUTTERCHECK_CTA` | Entfernt – CTA-Tracking über Meta Pixel Events |

### Entfernte Attribute (nicht mehr gesendet)

| Attribut (alt) | Grund |
|---|---|
| `TIERART` | Ersetzt durch `HUND_KATZE` |
| `FUTTERCHECK_ERGEBNIS` | Ersetzt durch `FUTTERCHECK_FARB_SCORE` |
| `SCORE_PROFIL` | Ersetzt durch `FUTTERCHECK_FARB_SCORE` |
| `FUTTERCHECK_CTA` | Entfernt – CTA-Tracking über Meta Pixel |

---

## 4. Meta Pixel Events

| Event | Auslöser |
|---|---|
| `PageView` | Seitenaufruf |
| `ViewContent` | Quiz gestartet |
| `Lead` | Lead-Formular abgesendet |
| `PartnerInteresse` (Custom) | Partner-CTA geklickt |
| `KundeInteresse` (Custom) | WhatsApp-CTA geklickt |

**Pixel ID:** `1267711028899835`

---

## 5. Design & UX

- **Mobile-First:** 80%+ Traffic von Instagram/TikTok
- **Farbschema:** Reico-Farben (Alge, Kristall, Chlorophyll, Kalk)
- **Score-Farben:** Grün (#22c55e, ≥75), Gelb (#eab308, 40–74), Rot (#ef4444, <40)
- **Partner-Bridge:** Immer sichtbar, hervorgehoben bei starkem Interesse

---

## 6. End-to-End Test Anleitung

### Lokaler Test

```bash
cd /pfad/zum/repo
python3 -m http.server 3000
# Browser: http://localhost:3000/futtercheck.html
```

### Testschritte

1. **Startseite** → "Futtercheck starten" klicken
2. **Alle 10 Quiz-Fragen** durchklicken (verschiedene Antworten testen)
3. **Partner-Frage (Q11)** → alle 3 Optionen testen
4. **Vorname** eingeben
5. **Lead-Formular** ausfüllen (E-Mail + DSGVO)
6. **Ergebnis** prüfen:
   - Score-Kreis korrekt?
   - Farbe passend zur Punktzahl?
   - Personalisierung (Tiername + Vorname)?
7. **Browser-Konsole öffnen** (F12) und prüfen:
   - 📤 "Brevo-Formular wird abgesendet …"
   - 📦 Formulardaten korrekt (EMAIL, HUND_KATZE, FUTTERCHECK_SCORE, etc.)
   - ✅ "Brevo: Formular-Submit ausgelöst – DOI-Mail wird versendet"
8. **DOI-Mail prüfen:** E-Mail mit Bestätigungslink erhalten?
9. **Bestätigungslink klicken** → Kontakt in Brevo aktiviert?
10. **Brevo prüfen:** Kontakt jetzt **aktiv** mit allen 7 Attributen korrekt?

### Brevo-Prüfung (Kontakt-Details)

Nach dem Test sollte der Kontakt in Brevo folgende Attribute haben:

| Attribut | Erwarteter Wert |
|---|---|
| `VORNAME` | Eingegebener Vorname |
| `HUND_KATZE` | `hund` oder `katze` |
| `TIERNAME` | Eingegebener Tiername |
| `FUTTERCHECK_SCORE` | Zahl 0–100 |
| `FUTTERCHECK_FUTTER` | Lesbarer Text (z.B. "Trockenfutter (Standard)") |
| `PARTNER_INTERESSE` | `Stark`, `Leicht` oder `Kunde` |
| `FUTTERCHECK_FARB_SCORE` | `gruen`, `gelb` oder `rot` |

---

## 7. Offene Punkte / TODOs

- [x] ~~`DOI_TEMPLATE_ID` ersetzen~~ – Entfällt
- [x] ~~Reine API-Lösung~~ – Ersetzt durch natives Brevo-Formular (zuverlässiger)
- [x] ~~Kategorie-Attribute~~ – Auf TEXT umgestellt
- [x] ~~API-Key im Frontend~~ – Entfernt, Formular braucht keinen
- [ ] **Brevo-Formular DOI-Settings prüfen** (Template, Redirect-URL)
- [ ] Brevo TEXT-Attribute anlegen (siehe Abschnitt 3 – einfach Typ "Text" wählen)
- [ ] Brevo Automations einrichten (Trigger: Kontakt bestätigt → Ergebnis-Mail + Partner-Sequenz)
- [ ] Ergebnis-Mails erstellen (Mail1–Mail5 + Ergebnis-Mail je Score)
- [ ] Datenschutz-Seite aktualisieren
- [ ] End-to-End DOI-Test: Quiz → DOI-Mail erhalten → Bestätigen → Kontakt aktiv + alle Attribute korrekt

---

## 8. Technische Hinweise

- **Kein Build-System:** Reine HTML/CSS/JS – direkt deploybar
- **Tailwind CSS:** Via CDN
- **Brevo-Formular:** Versteckt im DOM, sibforms.com JS/CSS für AJAX-Submit (kein API-Key nötig)
- **DSGVO:** Checkbox-Pflicht, Datenschutz-Link, DOI per E-Mail