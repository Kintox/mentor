# 📋 PROJEKT-ÜBERGABE: Reico Recruiting-Funnel Optimization

> **Stand:** 01.06.2026 | **Branch:** `recruiting-funnel-optimization`
> **Erstellt für:** Cedric Nitsch
> **Repository:** reico-funnel

---

## 1. Executive Summary

Im Rahmen dieses Projekts wurden **drei zusammenhängende Aufgaben** umgesetzt, die den Reico-Futtercheck-Funnel von der Datenerfassung über die Ergebnis-Zustellung bis zur automatisierten Partner-Akquise abdecken:

| # | Aufgabe | Status | Kernergebnis |
|---|---------|--------|--------------|
| **1** | Brevo-API-Übergabe optimieren | ✅ Erledigt | Neue Attribut-Struktur mit Kategorie-Feldern für saubere Segmentierung |
| **2** | Ergebnis-Mail erstellen | ✅ Erledigt | Dynamisches Template mit Conditional Content (grün/gelb/rot) |
| **3** | 5-Mail Partner-Sequenz schreiben | ✅ Erledigt | 5 Mails über 10 Tage – von Edukation bis Abschluss |

### Gesamter Funnel-Flow nach Umsetzung

```
Futtercheck ausfüllen (futtercheck.html)
    ↓ API: Kontakt in Brevo erstellt (Liste 5)
DOI-Mail (von Brevo automatisch)
    ↓ Kontakt bestätigt
Ergebnis-Mail (ergebnis_futtercheck.html) – je nach Score grün/gelb/rot
    ↓ 1 Tag warten
Mail 1 – Futter-Mythen & Vertrauen
    ↓ 2 Tage
Mail 2 – Reico-Vorteile & Social Proof
    ↓ 2 Tage
Mail 3 – Reico-Story & Partnerschaft vorstellen
    ↓ 2 Tage
Mail 4 – Business-Modell & Verdienst-Transparenz
    ↓ 3 Tage
Mail 5 – Urgency & Abschluss (letzte Mail)
```

---

## 2. Code-Änderungen (Aufgabe 1)

### 2.1 Geänderte Dateien

| Datei | Art der Änderung |
|-------|-----------------|
| `futtercheck.html` | Brevo-API-Payload komplett überarbeitet: neue Attribut-Namen, Kategorie-Mappings, Tag-Logik entfernt |
| `index.html` | Brevo-Formular entfernt → CTA-Card mit Link zu futtercheck.html |
| `FUTTERCHECK_UMBAU.md` | Technische Dokumentation der Änderungen |

### 2.2 Finale Brevo-Payload-Struktur (JSON)

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

### 2.3 Attribut-Mappings (Alt → Neu)

| Altes Attribut / Tag | Neues Attribut | Typ-Änderung |
|----------------------|----------------|-------------|
| `TIERART` | `HUND_KATZE` | Text → **Kategorie** |
| `FUTTERCHECK_ERGEBNIS` | `FUTTERCHECK_FARB_SCORE` | Text → **Kategorie** |
| `SCORE_PROFIL` | `FUTTERCHECK_FARB_SCORE` | Zusammengelegt |
| Tags: `hund`/`katze` | Attribut `HUND_KATZE` | Tag → Kategorie-Attribut |
| Tags: `score_gruen`/`gelb`/`rot` | Attribut `FUTTERCHECK_FARB_SCORE` | Tag → Kategorie-Attribut |
| Tags: `partner_interesse_stark`/`leicht`/`nur_kunde` | Attribut `PARTNER_INTERESSE` | Tag → Kategorie-Attribut |
| Tag: `FUTTERCHECK_CTA` | Entfernt | Tracking über Meta Pixel |
| *(neu)* | `PARTNER_INTERESSE` | **Kategorie** (neu hinzugefügt) |
| *(unverändert)* | `VORNAME`, `TIERNAME`, `FUTTERCHECK_SCORE`, `FUTTERCHECK_FUTTER`, `SMS` | Text / Zahl |

### 2.4 Quiz-Antwort → Brevo-Wert Mapping

| Quiz-Feld | Antwort im Quiz | → Brevo-Attribut | → Gesendeter Wert |
|-----------|----------------|------------------|-------------------|
| Tierart (Q1) | Hund | `HUND_KATZE` | `hund` |
| Tierart (Q1) | Katze | `HUND_KATZE` | `katze` |
| Partner-Frage (Q11) | "Ja, klingt spannend" | `PARTNER_INTERESSE` | `Stark` |
| Partner-Frage (Q11) | "Vielleicht" | `PARTNER_INTERESSE` | `Leicht` |
| Partner-Frage (Q11) | "Nein danke" | `PARTNER_INTERESSE` | `Kunde` |
| Score ≥ 75 | (automatisch) | `FUTTERCHECK_FARB_SCORE` | `gruen` |
| Score 40–74 | (automatisch) | `FUTTERCHECK_FARB_SCORE` | `gelb` |
| Score < 40 | (automatisch) | `FUTTERCHECK_FARB_SCORE` | `rot` |

### 2.5 Futter-Wert-Mapping

| Quiz-Value (intern) | → `FUTTERCHECK_FUTTER` (Brevo) |
|---------------------|-------------------------------|
| `nassfutter_premium` | Hochwertiges Nassfutter |
| `barf` | BARF / Rohfütterung |
| `selbstgekocht` | Selbstgekocht |
| `trockenfutter_premium` | Trockenfutter (Premium) |
| `trockenfutter_standard` | Trockenfutter (Standard) |
| `misch` | Mischfütterung (Nass + Trocken) |
| `unsicher` | Unsicher / wechselt oft |

---

## 3. Brevo-Konfiguration (⚠️ KRITISCH)

### 3.1 Kategorie-Attribute – EXAKT so anlegen!

> **⚠️ WARNUNG: Case-Sensitivity!**
> Brevo-Kategorie-Werte sind **case-sensitive**. Die Werte müssen **exakt** so angelegt werden, wie hier angegeben. Falsche Groß-/Kleinschreibung führt dazu, dass Conditional Content in E-Mails nicht funktioniert und Segmentierung fehlschlägt!

**Wo anlegen:** Brevo → Kontakte → Einstellungen → Kontakt-Attribute → Neues Attribut

### 3.2 Tabellarische Übersicht aller Attribute

| Attribut-ID | Typ | Kategorie-Werte (exakte Schreibweise!) | Beschreibung |
|-------------|-----|----------------------------------------|-------------|
| `VORNAME` | **Text** | *(Freitext)* | Vorname des Leads |
| `HUND_KATZE` | **Kategorie** | `hund`, `katze` | Tierart – **Kleinschreibung!** |
| `TIERNAME` | **Text** | *(Freitext)* | Name des Haustiers |
| `FUTTERCHECK_SCORE` | **Zahl** | *(0–100)* | Numerischer Futter-Score |
| `FUTTERCHECK_FUTTER` | **Text** | *(Freitext)* | Aktuelles Futter (lesbarer Text) |
| `PARTNER_INTERESSE` | **Kategorie** | `Stark`, `Leicht`, `Kunde` | Partner-Interesse – **Erster Buchstabe groß!** |
| `FUTTERCHECK_FARB_SCORE` | **Kategorie** | `gruen`, `gelb`, `rot` | Ampel-Bewertung – **Kleinschreibung, kein ü!** |
| `SMS` | **Text** | *(Freitext, optional)* | Telefonnummer |

### 3.3 Kategorie-Werte im Detail

#### `HUND_KATZE` (Kategorie)

| ID | Wert | ⚠️ Achtung |
|----|------|-----------|
| 1 | `hund` | Kleinschreibung! Nicht "Hund" |
| 2 | `katze` | Kleinschreibung! Nicht "Katze" |

#### `PARTNER_INTERESSE` (Kategorie)

| ID | Wert | ⚠️ Achtung |
|----|------|-----------|
| 1 | `Stark` | Großes S! Nicht "stark" |
| 2 | `Leicht` | Großes L! Nicht "leicht" |
| 3 | `Kunde` | Großes K! Nicht "kunde" |

#### `FUTTERCHECK_FARB_SCORE` (Kategorie)

| ID | Wert | ⚠️ Achtung |
|----|------|-----------|
| 1 | `gruen` | Kleinschreibung, **kein ü** → `gruen` nicht `grün` |
| 2 | `gelb` | Kleinschreibung |
| 3 | `rot` | Kleinschreibung |

---

## 4. Setup-Anleitungen

### 4.1 Ergebnis-Mail Setup

📄 **Vollständige Anleitung:** [`mails/BREVO_ERGEBNIS_MAIL_SETUP.md`](mails/BREVO_ERGEBNIS_MAIL_SETUP.md)

**Kurzfassung:**
- Template `ergebnis_futtercheck.html` in Brevo als E-Mail-Vorlage importieren
- Verwendet Conditional Content (`{% if contact.FUTTERCHECK_FARB_SCORE == "gruen" %}`)
- 3 Varianten: Grün (Score ≥ 75), Gelb (40–74), Rot (< 40)
- Personalisierung: Vorname, Tiername, Score, Hund/Katze
- Platzhalter `[CALENDLY-LINK]` durch echten Link ersetzen

### 4.2 Partner-Sequenz Setup

📄 **Vollständige Anleitung:** [`mails/BREVO_PARTNER_SEQUENZ_SETUP.md`](mails/BREVO_PARTNER_SEQUENZ_SETUP.md)

**Kurzfassung:**
- 5 HTML-Templates (Mail1–Mail5) in Brevo importieren
- Automation mit Wartezeiten einrichten (Tag 1 → 3 → 5 → 7 → 10)
- Conditional Content basierend auf `PARTNER_INTERESSE` (Stark/Leicht/Kunde)
- Platzhalter ersetzen: `[CALENDLY-LINK]`, `[IMPRESSUM-LINK]`, `[DATENSCHUTZ-LINK]`, `[SHOP-LINK]`, `[WHATSAPP-NUMMER]`

### 4.3 Gesamte Automation-Struktur

```
┌─────────────────────────────────────────────────────────┐
│                   BREVO AUTOMATIONS                     │
├─────────────────────────────────────────────────────────┤
│                                                         │
│  TRIGGER: Kontakt in Liste 5 ("Futtercheck") hinzugefügt│
│      ↓                                                  │
│  [DOI-Mail senden] (Brevo Standard)                     │
│      ↓                                                  │
│  [Warten auf DOI-Bestätigung]                           │
│      ↓                                                  │
│  [Wartezeit: 2 Minuten]                                 │
│      ↓                                                  │
│  ★ ERGEBNIS-MAIL ★ (ergebnis_futtercheck.html)          │
│      ↓   Conditional: gruen / gelb / rot                │
│      ↓                                                  │
│  [Wartezeit: 1 Tag]                                     │
│      ↓                                                  │
│  📧 Mail 1 – Futter-Mythen & Vertrauen                  │
│      ↓                                                  │
│  [Wartezeit: 2 Tage]                                    │
│      ↓                                                  │
│  📧 Mail 2 – Reico-Vorteile & Social Proof               │
│      ↓                                                  │
│  [Wartezeit: 2 Tage]                                    │
│      ↓                                                  │
│  📧 Mail 3 – Reico-Story & Partnerschaft                 │
│      ↓                                                  │
│  [Wartezeit: 2 Tage]                                    │
│      ↓                                                  │
│  📧 Mail 4 – Business-Modell & Verdienst                 │
│      ↓                                                  │
│  [Wartezeit: 3 Tage]                                    │
│      ↓                                                  │
│  📧 Mail 5 – Urgency & Abschluss (letzte Mail)           │
│      ↓                                                  │
│  [ENDE]                                                 │
│                                                         │
│  EXIT-BEDINGUNGEN:                                      │
│  • Kontakt hat Calendly-Termin gebucht                  │
│  • Kontakt hat sich abgemeldet                          │
│  • Kontakt hat sich als Partner registriert             │
└─────────────────────────────────────────────────────────┘
```

---

## 5. Testing-Anleitung (End-to-End)

### Voraussetzung

```bash
cd /pfad/zum/repo
python3 -m http.server 3000
# Browser: http://localhost:3000/futtercheck.html
```

### Schritt 1: Futtercheck ausfüllen

1. `futtercheck.html` im Browser öffnen
2. Alle Quiz-Fragen beantworten (Q1–Q11)
3. **Verschiedene Szenarien testen:**

| Testszenario | Tierart | Antworten | Erwarteter Score | Partner-Interesse |
|-------------|---------|-----------|-----------------|-------------------|
| Test Grün/Stark | Hund | Optimal-Antworten | ≥ 75 (grün) | "Ja, klingt spannend" → `Stark` |
| Test Gelb/Leicht | Katze | Mittel-Antworten | 40–74 (gelb) | "Vielleicht" → `Leicht` |
| Test Rot/Kunde | Hund | Schlecht-Antworten | < 40 (rot) | "Nein danke" → `Kunde` |

4. Vorname + E-Mail eingeben, DSGVO bestätigen
5. "Ergebnis anzeigen" klicken

**Prüfen:**
- ✅ Score-Kreis wird korrekt angezeigt (richtige Farbe?)
- ✅ Personalisierung funktioniert (Tiername, Vorname)
- ✅ Browser-Konsole (F12): "Kontakt erfolgreich in Brevo erstellt/aktualisiert"
- ✅ Keine JavaScript-Fehler in der Konsole

### Schritt 2: Brevo-Kontakt prüfen

1. In Brevo einloggen → Kontakte → Kontakt suchen (E-Mail)
2. **Alle Attribute kontrollieren:**

| Attribut | Erwarteter Wert | ✅/❌ |
|----------|----------------|------|
| `VORNAME` | Eingegebener Vorname | |
| `HUND_KATZE` | `hund` oder `katze` (Kleinschreibung!) | |
| `TIERNAME` | Eingegebener Tiername | |
| `FUTTERCHECK_SCORE` | Zahl 0–100 | |
| `FUTTERCHECK_FUTTER` | Lesbarer Text (z.B. "Trockenfutter (Standard)") | |
| `PARTNER_INTERESSE` | `Stark`, `Leicht` oder `Kunde` | |
| `FUTTERCHECK_FARB_SCORE` | `gruen`, `gelb` oder `rot` | |
| `SMS` | Telefonnummer (falls eingegeben) | |
| Liste | In Liste 5 ("Futtercheck") | |

### Schritt 3: DOI-Mail erhalten und bestätigen

1. E-Mail-Postfach prüfen → DOI-Mail erhalten?
2. Bestätigungslink klicken
3. **Prüfen:**
   - ✅ DOI-Bestätigung erfolgreich
   - ✅ Kontakt-Status in Brevo aktualisiert

### Schritt 4: Ergebnis-Mail erhalten

1. Nach DOI-Bestätigung + 2 Minuten: Ergebnis-Mail erhalten?
2. **Prüfen:**
   - ✅ **Richtige Farbvariante** angezeigt (grün/gelb/rot je nach Score)?
   - ✅ Personalisierung korrekt: `{{ contact.VORNAME }}`, `{{ contact.TIERNAME }}`
   - ✅ Score-Zahl korrekt: `{{ contact.FUTTERCHECK_SCORE }}`
   - ✅ Hund/Katze-Text richtig (`Hundefutter` vs. `Katzenfutter`)
   - ✅ Betreff personalisiert
   - ✅ Calendly-Link funktioniert
   - ✅ Mobile Darstellung OK
   - ✅ Abmelde-Link funktioniert

### Schritt 5: Partner-Sequenz (5 Mails über 10 Tage)

| Tag | Mail | Prüfpunkte |
|-----|------|-----------|
| Tag 1 | Mail 1 – Futter-Mythen | Personalisierung (Vorname, Tiername), alle Links, Mobile |
| Tag 3 | Mail 2 – Reico-Vorteile | Conditional Content je PARTNER_INTERESSE, Social Proof |
| Tag 5 | Mail 3 – Reico-Story | Partnerschaft-CTA angepasst an Interesse-Level |
| Tag 7 | Mail 4 – Business-Modell | Verdienst-Tabelle korrekt, starker CTA bei "Stark" |
| Tag 10 | Mail 5 – Abschluss | Urgency-Elemente, finale CTAs, "Letzte Mail"-Hinweis |

### Was bei jeder Mail geprüft werden muss

- [ ] **Personalisierung:** `{{ contact.VORNAME }}` und `{{ contact.TIERNAME }}` korrekt aufgelöst
- [ ] **Conditional Content:** Unterschiedliche Inhalte für `Stark` / `Leicht` / `Kunde`
- [ ] **Links:** Calendly, Impressum, Datenschutz, WhatsApp – alle klickbar und korrekt
- [ ] **Segmentierung:** Kontakte mit `PARTNER_INTERESSE = "Kunde"` erhalten sanftere CTAs
- [ ] **Mobile Darstellung:** E-Mails auf Handy/Tablet lesbar
- [ ] **Abmelde-Link:** `{{ unsubscribe }}` funktioniert
- [ ] **Absender:** "Cedric von Reico" / `info@cedricnitsch.de`
- [ ] **Kein Spam:** E-Mail landet im Posteingang (SPF/DKIM/DMARC prüfen)

---

## 6. Offene Punkte / Bestätigungen

### ⚠️ Annahmen, die noch bestätigt werden müssen

| # | Offener Punkt | Aktuelle Annahme | Bestätigt? |
|---|--------------|------------------|-----------|
| 1 | **Brevo-Listen-ID** | Aktuell: `5` – Ist das die richtige Liste für Futtercheck-Leads? | ❓ |
| 2 | **API-Key funktionsfähig** | API-Key ist im Code als Split-Array hinterlegt – funktioniert er noch? | ❓ |
| 3 | **Kategorie-Attribute in Brevo angelegt** | `HUND_KATZE`, `PARTNER_INTERESSE`, `FUTTERCHECK_FARB_SCORE` müssen mit exakten Werten angelegt werden | ❓ |
| 4 | **Bestehende Automations** | Gibt es bereits Automation-Workflows, die angepasst/ersetzt werden müssen? | ❓ |
| 5 | **DOI-Template** | Existiert bereits ein DOI-E-Mail-Template in Brevo? | ❓ |
| 6 | **Absender-Domain verifiziert** | Ist `cedricnitsch.de` in Brevo als Absender-Domain verifiziert (SPF/DKIM)? | ❓ |
| 7 | **Calendly-Link** | Welcher Calendly-Link soll in den Mails verwendet werden? Platzhalter: `[CALENDLY-LINK]` | ❓ |
| 8 | **Shop-Link** | Welcher Reico-Shop-Link soll in Mail 5 verwendet werden? Platzhalter: `[SHOP-LINK]` | ❓ |
| 9 | **WhatsApp-Nummer** | Welche WhatsApp-Nummer soll verwendet werden? Platzhalter: `[WHATSAPP-NUMMER]` | ❓ |
| 10 | **Impressum/Datenschutz-URLs** | Welche URLs sollen für Impressum und Datenschutz verwendet werden? | ❓ |
| 11 | **Meta Pixel ID** | Ist `1267711028899835` die korrekte Pixel-ID? | ❓ |
| 12 | **Bestehende Kontakt-Attribute** | Gibt es bereits Attribute mit alten Namen (TIERART, FUTTERCHECK_ERGEBNIS), die kollidieren könnten? | ❓ |

---

## 7. Nächste Schritte (Priorisiert)

### 🔴 SOFORT (vor Go-Live)

| Prio | Aktion | Geschätzter Aufwand |
|------|--------|-------------------|
| **1** | **Kategorie-Attribute in Brevo anlegen** (siehe Abschnitt 3) – HUND_KATZE, PARTNER_INTERESSE, FUTTERCHECK_FARB_SCORE mit exakten Werten | 10 Min |
| **2** | **Restliche Text-/Zahl-Attribute anlegen** (VORNAME, TIERNAME, FUTTERCHECK_SCORE, FUTTERCHECK_FUTTER, SMS) | 5 Min |
| **3** | **Brevo-Listen-ID bestätigen** – Ist Liste 5 korrekt? | 2 Min |
| **4** | **API-Key testen** – Einen Test-Kontakt über den Futtercheck erstellen | 5 Min |
| **5** | **Platzhalter in allen Mail-Templates ersetzen** ([CALENDLY-LINK], [IMPRESSUM-LINK], etc.) | 10 Min |

### 🟡 DANACH (Automation aufbauen)

| Prio | Aktion | Geschätzter Aufwand |
|------|--------|-------------------|
| **6** | **Ergebnis-Mail Template in Brevo importieren** (siehe `BREVO_ERGEBNIS_MAIL_SETUP.md`) | 15 Min |
| **7** | **5 Partner-Sequenz Templates in Brevo importieren** (Mail1–Mail5) | 30 Min |
| **8** | **DOI-Automation prüfen/einrichten** – Trigger: Kontakt in Liste 5 | 15 Min |
| **9** | **Ergebnis-Mail in DOI-Automation einhängen** – nach DOI-Bestätigung + 2 Min | 10 Min |
| **10** | **Partner-Sequenz-Automation erstellen** – 5 Mails mit Wartezeiten | 20 Min |
| **11** | **Exit-Bedingungen definieren** (Calendly-Buchung, Abmeldung) | 10 Min |

### 🟢 TESTEN

| Prio | Aktion | Geschätzter Aufwand |
|------|--------|-------------------|
| **12** | **3 Test-Kontakte anlegen** (grün/gelb/rot) und End-to-End testen | 20 Min |
| **13** | **Conditional Content verifizieren** für alle 3 Varianten × 3 Interesse-Level | 30 Min |
| **14** | **Mobile-Test** aller Mails | 15 Min |
| **15** | **Absender-Domain prüfen** (SPF/DKIM/DMARC für cedricnitsch.de) | 15 Min |

### ⚪ OPTIONAL / EMPFOHLEN

| Prio | Aktion |
|------|--------|
| **16** | Datenschutz-Seite aktualisieren (neue Datenverarbeitung dokumentieren) |
| **17** | KPI-Dashboard in Brevo einrichten (Öffnungsrate, Klickrate, Conversions) |
| **18** | A/B-Tests für Betreffzeilen planen |

---

## 8. Dateiübersicht

### Alle wichtigen Dateien im Repository

| Datei | Pfad | Beschreibung |
|-------|------|-------------|
| **Futtercheck** | `futtercheck.html` | Quiz + Lead-Formular + Ergebnis + Brevo-API (Hauptdatei) |
| **Landing Page** | `index.html` | CTA → Futtercheck |
| **Danke-Seite** | `danke.html` | Bestätigungsseite |
| **Stylesheet** | `style.css` | Globale Styles |
| **Ergebnis-Mail** | `mails/ergebnis_futtercheck.html` | Brevo-Template mit Conditional Content (grün/gelb/rot) |
| **Mail 1** | `mails/Mail1.html` | Futter-Mythen & Vertrauen (Tag 1) |
| **Mail 2** | `mails/Mail2.html` | Reico-Vorteile & Social Proof (Tag 3) |
| **Mail 3** | `mails/Mail3.html` | Reico-Story & Partnerschaft (Tag 5) |
| **Mail 4** | `mails/Mail4.html` | Business-Modell & Verdienst (Tag 7) |
| **Mail 5** | `mails/Mail5.html` | Urgency & Abschluss (Tag 10) |
| **Doku: Umbau** | `FUTTERCHECK_UMBAU.md` | Technische Doku der Futtercheck-Änderungen |
| **Doku: Ergebnis-Mail** | `mails/BREVO_ERGEBNIS_MAIL_SETUP.md` | Setup-Anleitung für Ergebnis-Mail |
| **Doku: Sequenz** | `mails/BREVO_PARTNER_SEQUENZ_SETUP.md` | Setup-Anleitung für 5-Mail Partner-Sequenz |
| **Doku: Übergabe** | `PROJEKT_UEBERGABE.md` | Dieses Dokument – Gesamtübersicht |
| **Datenschutz** | `datenschutz.html` | Datenschutzerklärung |
| **Impressum** | `impressum.html` | Impressum |
| **DNS** | `CNAME` | GitHub Pages Domain-Config |
| **README** | `README.md` | Projekt-README |

---

## Anhang: Brevo API-Endpunkt

```
POST https://api.brevo.com/v3/contacts
Content-Type: application/json
api-key: [API-KEY]
```

Der API-Key ist im Code (`futtercheck.html`) als gesplittetes Array hinterlegt, um GitHub Push Protection zu umgehen.

---

*Erstellt am 01.06.2026 | Projekt: Reico Recruiting-Funnel Optimization*
