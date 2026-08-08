# DOI-Integration – Finale Dokumentation

> **Stand:** 01.06.2026 | **Branch:** `recruiting-funnel-optimization`
> **Letztes Update:** Natives Brevo-Formular für DOI – keine API mehr

---

## Überblick

Die DOI (Double-Opt-In) Integration nutzt ein **natives Brevo-Formular** (sibforms.com), das unsichtbar im DOM eingebettet ist. Das Formular enthält alle 8 Custom-Attribute als Felder und wird nach Quiz-Abschluss programmatisch befüllt und abgesendet.

### Warum Brevo-Formular statt API?

| Ansatz | Status | Bewertung |
|--------|--------|-----------|
| Brevo API `/contacts/doubleOptinConfirmation` | ❌ Verworfen | Template-ID-Setup komplex, CORS-Probleme |
| Brevo API `POST /contacts` (updateEnabled: false) | ❌ Unzuverlässig | DOI wurde nicht immer ausgelöst |
| **Natives Brevo-Formular (sibforms)** | ✅ **Aktuelle Lösung** | Offizieller Brevo-Weg, alle Attribute als Felder, zuverlässiger DOI |

### Vorteile

- **Kein API-Key im Frontend** – keine Sicherheitsbedenken
- **Offizieller Brevo-Mechanismus** – sibforms handled DOI, Validierung, Duplikate
- **Alle 8 Custom-Attribute** direkt als Formularfelder im Brevo-Formular konfiguriert
- **DOI-Template** wird in den Brevo-Formular-Einstellungen konfiguriert (kein Template-ID im Code)

---

## Technischer Ablauf

```
Nutzer füllt Futtercheck aus
    ↓ Klickt "Ergebnis anzeigen"
    ↓
┌─────────────────────────────────────────────────┐
│ sendToBrevo(score, profile)                      │
│                                                  │
│ 1. Verstecktes Brevo-Formular (#sib-form) finden │
│ 2. Alle Felder programmatisch befüllen:          │
│    EMAIL, VORNAME, HUND_KATZE, TIERNAME,         │
│    FUTTERCHECK_SCORE, FUTTERCHECK_FUTTER,        │
│    FUTTERCHECK_FARB_SCORE, PARTNER_INTERESSE,    │
│    SMS, SMS__COUNTRY_CODE (+49)                  │
│ 3. Honeypot (email_address_check) leer lassen    │
│ 4. Submit-Button programmatisch klicken          │
└─────────────────────────────────────────────────┘
    ↓
┌─────────────────────────────────────────────────┐
│ sibforms main.js (extern, von Brevo)             │
│                                                  │
│ → Fängt den Submit ab                            │
│ → Validiert die Felder                           │
│ → AJAX POST an sibforms.com                      │
│ → Brevo verarbeitet: Kontakt + Attribute + DOI   │
└─────────────────────────────────────────────────┘
    ↓
Nutzer sieht sofort das Ergebnis (Score-Kreis)
    ↓
Nutzer erhält DOI-Mail → klickt Bestätigungslink
    ↓
Kontakt wird in Brevo aktiviert → Automation startet
```

---

## Verstecktes Brevo-Formular

Das Formular ist unsichtbar im DOM eingebettet (`position:absolute; left:-9999px`).

### Formular-Action
```
https://527052f3.sibforms.com/serve/MUIFAEsoUsu2lQEwZbEGWqu3v6u9wL2SIPPUymK-A_Vc0QPh5brKiyc__E3MTjN9YYhO3Tqv-YtqYTJeAH6h3IV0Mt5ECrXtrWv73icuhdF0rgKNrVbQDxNTrQLccP0-cvUBbw9TcGjJobMlR1757jpWbQmfb6FPlxkFczQhk0UQ5mCi5IIPD5GZPTV7gIyGgCy2PSFGJ-ObCxEZ1g==
```

### Formularfelder

| Feld | Name | Typ | Required | Beschreibung |
|------|------|-----|----------|-------------|
| E-Mail | `EMAIL` | text | ✅ | Aus Lead-Formular |
| Vorname | `VORNAME` | text | ✅ | Aus Quiz-Antwort |
| Tierart | `HUND_KATZE` | text | ✅ | "hund" oder "katze" |
| Tiername | `TIERNAME` | text | ✅ | Aus Quiz-Antwort |
| Score | `FUTTERCHECK_SCORE` | text (numeric) | ✅ | 0–100 |
| Farb-Score | `FUTTERCHECK_FARB_SCORE` | text | ✅ | "gruen"/"gelb"/"rot" |
| Futter | `FUTTERCHECK_FUTTER` | text | ✅ | Lesbarer Text |
| Partner | `PARTNER_INTERESSE` | text | ✅ | "Stark"/"Leicht"/"Kunde" |
| WhatsApp | `SMS` | tel | ❌ | Telefonnummer (optional) |
| Ländervorwahl | `SMS__COUNTRY_CODE` | select | – | Default: `+49 DE` |
| Honeypot | `email_address_check` | hidden | – | Anti-Bot (MUSS leer bleiben!) |
| Locale | `locale` | hidden | – | `de` |

### Externe Abhängigkeiten

| Ressource | URL | Zweck |
|-----------|-----|-------|
| sibforms CSS | `sibforms.com/forms/end-form/build/sib-styles.css` | Formular-Styles (minimal, da versteckt) |
| sibforms JS | `sibforms.com/forms/end-form/build/main.js` | AJAX-Form-Handler für DOI-Submit |

---

## Code-Implementierung

### `sendToBrevo(score, profile)`

```javascript
function sendToBrevo(score, profile) {
    const form = document.getElementById('sib-form');
    
    // Felder befüllen
    form.querySelector('#EMAIL').value = answers['email'];
    form.querySelector('#VORNAME').value = answers['vorname'];
    form.querySelector('#HUND_KATZE').value = tierart;
    form.querySelector('#TIERNAME').value = tiername;
    form.querySelector('#FUTTERCHECK_SCORE').value = String(score);
    form.querySelector('#FUTTERCHECK_FUTTER').value = futterLabel;
    form.querySelector('#FUTTERCHECK_FARB_SCORE').value = profile;
    form.querySelector('#PARTNER_INTERESSE').value = partnerLabel;
    form.querySelector('#SMS').value = phoneNumber;
    form.querySelector('[name="SMS__COUNTRY_CODE"]').value = '+49';
    form.querySelector('[name="email_address_check"]').value = '';
    
    // Submit auslösen → sibforms main.js macht AJAX
    form.querySelector('[type="submit"]').click();
}
```

---

## Kontakt-Attribute (alle TEXT in Brevo)

| Attribut-ID | Typ | Mögliche Werte | Beschreibung |
|---|---|---|---|
| `VORNAME` | Text | Freitext | Vorname des Leads |
| `HUND_KATZE` | Text | `hund`, `katze` | Tierart |
| `TIERNAME` | Text | Freitext | Name des Tieres |
| `FUTTERCHECK_SCORE` | Zahl | `0`–`100` | Numerischer Futter-Score |
| `FUTTERCHECK_FUTTER` | Text | Freitext | Aktuelles Futter (lesbarer Text) |
| `PARTNER_INTERESSE` | Text | `Stark`, `Leicht`, `Kunde` | Interesse an Partnerschaft |
| `FUTTERCHECK_FARB_SCORE` | Text | `gruen`, `gelb`, `rot` | Ampel-Bewertung |
| `SMS` | Text | Freitext (optional) | Telefonnummer |

---

## Was wurde entfernt?

| Was | Warum |
|-----|-------|
| `BREVO_API_KEY` (gesplittetes Array) | Kein API-Key mehr nötig – Formular authentifiziert sich über Action-URL |
| `BREVO_LIST_ID` Konstante | Liste wird im Brevo-Formular konfiguriert |
| `POST /v3/contacts` API-Call | Ersetzt durch Formular-Submit |
| `PUT /v3/contacts/{email}` Fallback | sibforms handled Duplikate intern |
| `updateBrevoContact()` Funktion | Nicht mehr nötig |

---

## Testanleitung

### 1. Lokal testen
```bash
cd /pfad/zum/repo
python3 -m http.server 3000
# Browser: http://localhost:3000/futtercheck.html
```

### 2. Quiz durchlaufen
1. Alle Fragen beantworten
2. Lead-Formular ausfüllen (echte E-Mail verwenden!)
3. "Ergebnis anzeigen" klicken

### 3. Browser-Konsole prüfen (F12)
Erwartete Logs:
```
📤 Brevo-Formular wird abgesendet …
📦 Formulardaten: { EMAIL: "...", VORNAME: "...", HUND_KATZE: "hund", ... }
✅ Brevo: Formular-Submit ausgelöst – DOI-Mail wird versendet
```

### 4. E-Mail prüfen
- DOI-Bestätigungsmail erhalten?
- Bestätigungslink klicken → `bestaetigung-danke.html`?
- In Brevo: Kontakt jetzt aktiv mit allen Attributen?

---

## Brevo-Formular-Einstellungen

DOI-Template und Redirect-URL werden **in den Brevo-Formular-Einstellungen** konfiguriert (nicht im Code):

1. Brevo → Kontakte → Formulare → das Formular bearbeiten
2. **DOI-Bestätigungsmail:** Template dort auswählen
3. **Redirect nach Bestätigung:** `https://cedricnitsch.de/bestaetigung-danke.html`
4. **Liste:** Im Formular ist die Ziel-Liste bereits konfiguriert

---

## Offene Punkte

- [ ] Live-Test: Quiz → DOI-Mail erhalten → Bestätigen → Kontakt prüfen
- [ ] Brevo-Automations einrichten (Trigger: Kontakt bestätigt → Ergebnis-Mail + Sequenz)
- [ ] Prüfen ob Redirect-URL im Brevo-Formular korrekt konfiguriert ist
