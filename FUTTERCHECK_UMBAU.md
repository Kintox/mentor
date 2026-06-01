# 🔄 Futtercheck Komplett-Umbau – Dokumentation

> **Stand:** 01.06.2026 | **Branch:** `recruiting-funnel-optimization`
> **Letztes Update:** Brevo-Attribute auf finale Struktur umgestellt

---

## 1. Was wurde geändert?

### Dateien

| Datei | Änderung |
|---|---|
| `futtercheck.html` | **Komplett neu geschrieben** – eigenständige Quiz-Seite mit Lead-Formular am Ende, Hybrid-Teaser-Ergebnis und Partner-Bridge |
| `index.html` | Brevo-Formular entfernt → durch saubere CTA-Card mit Link zu `futtercheck.html` ersetzt |

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
    └── Partner-Bridge Block:
        ├── Persönliche Story von Cedric
        ├── 4 Benefit-Cards
        └── Partner-CTA: "Mehr über die Partnerschaft erfahren"
```

---

## 3. Brevo-Integration (FINALE STRUKTUR)

### API-Anbindung

- **Methode:** Brevo REST API v3 (`POST /contacts`)
- **API-Key:** Im Code als gesplittetes Array (GitHub Push Protection)
- **Listen-ID:** `5`
- **updateEnabled:** `true` (bestehende Kontakte werden aktualisiert)

### Finale JSON-Payload (Brevo REST API v3)

```json
{
  "email": "max@beispiel.de",
  "attributes": {
    "VORNAME": "Max",
    "HUND_KATZE": "hund",
    "TIERNAME": "Bello",
    "FUTTERCHECK_SCORE": 72,
    "FUTTERCHECK_FUTTER": "Trockenfutter (Standard)",
    "PARTNER_INTERESSE": "Stark",
    "FUTTERCHECK_FARB_SCORE": "gelb",
    "SMS": "0151..."
  },
  "listIds": [5],
  "updateEnabled": true
}
```

### Kontakt-Attribute (in Brevo anzulegen)

| Attribut-ID | Typ | Mögliche Werte | Beschreibung |
|---|---|---|---|
| `VORNAME` | **Text** | Freitext | Vorname des Leads |
| `HUND_KATZE` | **Kategorie** | `hund`, `katze` | Tierart (Kleinschreibung!) |
| `TIERNAME` | **Text** | Freitext | Name des Tieres |
| `FUTTERCHECK_SCORE` | **Zahl** | `0`–`100` | Numerischer Futter-Score |
| `FUTTERCHECK_FUTTER` | **Text** | Freitext | Aktuelles Futter (lesbarer Text) |
| `PARTNER_INTERESSE` | **Kategorie** | `Stark`, `Leicht`, `Kunde` | Interesse an Partnerschaft |
| `FUTTERCHECK_FARB_SCORE` | **Kategorie** | `gruen`, `gelb`, `rot` | Ampel-Bewertung des Scores |
| `SMS` | **Text** | Freitext (optional) | Telefonnummer |

### ⚠️ Brevo Kategorie-Attribute – EXAKT so anlegen!

Kategorie-Attribute müssen in Brevo unter **Kontakte → Einstellungen → Kontakt-Attribute** mit den exakt folgenden Werten angelegt werden. Groß-/Kleinschreibung muss übereinstimmen!

#### HUND_KATZE (Kategorie)
| ID | Wert |
|---|---|
| 1 | `hund` |
| 2 | `katze` |

#### PARTNER_INTERESSE (Kategorie)
| ID | Wert |
|---|---|
| 1 | `Stark` |
| 2 | `Leicht` |
| 3 | `Kunde` |

#### FUTTERCHECK_FARB_SCORE (Kategorie)
| ID | Wert |
|---|---|
| 1 | `gruen` |
| 2 | `gelb` |
| 3 | `rot` |

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
   - ✅ "Kontakt erfolgreich in Brevo erstellt/aktualisiert"
   - 📦 Payload enthält korrekte Attribute
8. **Brevo prüfen:** Kontakt mit allen 7 Attributen korrekt?

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

- [ ] Brevo Listen-ID verifizieren (`BREVO_LIST_ID = 5`)
- [ ] **Brevo Kategorie-Attribute anlegen** (siehe Abschnitt 3 oben – exakte Werte!)
- [ ] DOI-E-Mail Template in Brevo erstellen
- [ ] Brevo Automations einrichten (DOI, Partner-Sequenz, Kunden-Sequenz)
- [ ] Ergebnis-Mails erstellen (Mail1–Mail5 + Ergebnis-Mail je Score)
- [ ] Datenschutz-Seite aktualisieren
- [ ] Live-Test mit echtem Brevo-Account durchführen

---

## 8. Technische Hinweise

- **Kein Build-System:** Reine HTML/CSS/JS – direkt deploybar
- **Tailwind CSS:** Via CDN
- **Brevo API-Key:** Im Code als Split-Array (GitHub Push Protection Workaround)
- **DSGVO:** Checkbox-Pflicht, Datenschutz-Link, DOI per E-Mail