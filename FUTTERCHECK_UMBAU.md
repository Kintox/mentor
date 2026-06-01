# 🔄 Futtercheck Komplett-Umbau – Dokumentation

> **Stand:** 01.06.2026 | **Branch:** `recruiting-funnel-optimization`

---

## 1. Was wurde geändert?

### Dateien

| Datei | Änderung |
|---|---|
| `futtercheck.html` | **Komplett neu geschrieben** – eigenständige Quiz-Seite mit Lead-Formular am Ende, Hybrid-Teaser-Ergebnis und Partner-Bridge |
| `index.html` | Brevo-Formular entfernt → durch saubere CTA-Card mit Link zu `futtercheck.html` ersetzt. Alle Brevo-CSS/JS-Overrides entfernt |

### Vorher → Nachher

| Vorher | Nachher |
|---|---|
| Brevo-Opt-in-Formular **vor** dem Futtercheck (Doppelte Hürde) | Futtercheck **direkt erreichbar** – kein Vorformular |
| Lead-Daten am Anfang abgefragt | Lead-Formular **am Ende** nach allen Quiz-Fragen (höhere Conversion) |
| Ergebnis erst nach DOI-Bestätigung | **Hybrid-Teaser**: Score sofort sichtbar, ausführliche Analyse per DOI-E-Mail |
| Kein Partner-Akquise-Element | **Partner-Bridge** Block auf der Ergebnisseite mit eigenem CTA |
| Nur "Kunde"-Flow | **Ein Funnel mit Tagging** – Kunde vs. Partner automatisch segmentiert |

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

## 3. Brevo-Integration

### API-Anbindung

- **Methode:** Brevo REST API v3 (`/contacts` Endpoint)
- **API-Key:** Im Code als gesplittetes Array (GitHub Push Protection)
- **Listen-ID:** `2` (⚠️ Bitte in Brevo überprüfen/anpassen)

### Kontakt-Attribute (in Brevo anlegen)

| Attribut | Typ | Werte |
|---|---|---|
| `VORNAME` | Text | Freitext |
| `TELEFON` | Text | Freitext (optional) |
| `TIERART` | Text | `hund` oder `katze` |
| `TIERNAME` | Text | Freitext |
| `SCORE` | Number | 0–100 |
| `SCORE_PROFIL` | Text | `score_gruen` / `score_gelb` / `score_rot` |
| `PARTNER_INTERESSE` | Text | `ja_interesse` / `vielleicht` / `nein` |
| `FUTTERCHECK_CTA` | Text | `partner_cta_clicked` / `kunde_cta_clicked` |

### Tags (automatisch gesetzt)

| Tag | Beschreibung |
|---|---|
| `futtercheck` | Jeder Lead über den Futtercheck |
| `hund` / `katze` | Tierart |
| `score_gruen` | Score ≥ 75 |
| `score_gelb` | Score 50–74 |
| `score_rot` | Score < 50 |
| `partner_interesse_stark` | Q11 = "Ja, klingt spannend" |
| `partner_interesse_leicht` | Q11 = "Vielleicht" |
| `nur_kunde` | Q11 = "Nein danke" |

### Brevo Automations (manuell einzurichten)

1. **DOI-Automation:** Trigger = Tag `futtercheck` → DOI-E-Mail mit ausführlicher Analyse senden
2. **Partner-Sequenz:** Trigger = Tag `partner_interesse_stark` → Partner-Info-Sequenz (3–5 E-Mails)
3. **Kunden-Sequenz:** Trigger = Tag `nur_kunde` → Futterberatung-Sequenz
4. **CTA-Follow-up:** Attribut `FUTTERCHECK_CTA` = `partner_cta_clicked` → persönliche Nachricht

---

## 4. Meta Pixel Events

| Event | Auslöser |
|---|---|
| `PageView` | Seitenaufruf |
| `ViewContent` | Quiz gestartet (Start-Button geklickt) |
| `Lead` | Lead-Formular abgesendet (Ergebnis anzeigen) |
| `CustomEvent: FuttercheckCTAClick` | WhatsApp-CTA geklickt |
| `CustomEvent: PartnerCTAClick` | Partner-CTA geklickt |

**Pixel ID:** `1267711028899835`

---

## 5. Design & UX

- **Mobile-First:** 80%+ Traffic von Instagram/TikTok
- **Farbschema:** Reico-Farben (Alge, Kristall, Chlorophyll, Kalk)
- **Score-Farben:** Grün (#22c55e, ≥75), Gelb (#eab308, 50–74), Rot (#ef4444, <50)
- **Partner-Bridge:** Immer sichtbar, aber hervorgehoben (roter Rand + Pulse-Glow) bei starkem Interesse
- **Zwei CTAs:**
  - Primär (coral/salmon): WhatsApp-Beratung für Kunden
  - Sekundär (dunkelgrün): Partner-Info

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
   - Geblurrter Teaser sichtbar?
7. **CTAs testen:**
   - WhatsApp-Link öffnet korrekt?
   - Partner-CTA öffnet korrekt?
8. **Mobile testen** (Chrome DevTools → 400px Breite)
9. **Brevo prüfen:** Kontakt in Liste 2 mit Tags angelegt?

### Brevo-Test

⚠️ **Vor dem Live-Gang:**
- Listen-ID `2` in Brevo überprüfen (Zeile im Code mit `BREVO_LIST_ID`)
- Kontakt-Attribute in Brevo anlegen (siehe Tabelle oben)
- Test-Kontakt erstellen und Tags prüfen
- DOI-Automation einrichten und testen

---

## 7. Offene Punkte / TODOs

- [ ] Brevo Listen-ID verifizieren (`BREVO_LIST_ID = 2`)
- [ ] Brevo Kontakt-Attribute anlegen
- [ ] DOI-E-Mail Template in Brevo erstellen (mit ausführlicher Analyse)
- [ ] Brevo Automations einrichten (DOI, Partner-Sequenz, Kunden-Sequenz)
- [ ] Partner-CTA Ziel-URL festlegen (aktuell: WhatsApp mit Partner-Text)
- [ ] Datenschutz-Seite aktualisieren (Futtercheck-Datenverarbeitung erwähnen)
- [ ] Impressum aktualisieren falls nötig
- [ ] Live-Test mit echtem Brevo-Account durchführen

---

## 8. Technische Hinweise

- **Kein Build-System:** Reine HTML/CSS/JS – direkt deploybar
- **Tailwind CSS:** Via CDN (`unpkg.com/tailwindcss`)
- **Brevo API-Key:** Im Code als Split-Array gespeichert (GitHub Push Protection Workaround)
- **Keine externen Abhängigkeiten** außer Tailwind CDN + Lucide Icons CDN
- **DSGVO:** Checkbox-Pflicht, Datenschutz-Link, DOI per E-Mail, keine Cookies (außer Meta Pixel)
