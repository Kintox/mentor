# DOI-Integration – Finale Dokumentation

> **Stand:** 01.06.2026 | **Branch:** `recruiting-funnel-optimization`
> **Letztes Update:** Reine API-Lösung – sibforms/Hybrid komplett entfernt

---

## Überblick

Die DOI (Double-Opt-In) Integration nutzt die **Brevo REST API v3** (`POST /contacts`) mit `updateEnabled: false`. Dies ist die sauberste Lösung: ein einziger API-Call erstellt den Kontakt, setzt alle Attribute und löst die DOI-Bestätigungsmail aus.

### Warum reine API-Lösung?

| Ansatz | Status | Bewertung |
|--------|--------|-----------|
| Brevo API `/contacts/doubleOptinConfirmation` | ❌ Verworfen | Template-ID-Setup komplex, CORS-Probleme |
| Brevo DOI-Formular (sibforms) | ❌ Entfernt | Externe Abhängigkeit, nur Basis-Felder, fragil |
| Hybrid: API + Formular | ❌ Entfernt | Überflüssig komplex, zwei Schritte statt einem |
| **Reine API (`POST /contacts`)** | ✅ **Aktuelle Lösung** | Ein Call, alle Attribute, DOI über Listeneinstellung |

### Voraussetzungen in Brevo

1. **DOI in Liste aktivieren:** Kontakte → Listen → Liste 5 → Einstellungen → "Double Opt-In aktivieren"
2. **DOI-Template konfigurieren:** In den Listeneinstellungen das DOI-Bestätigungs-Template auswählen
3. **Redirect-URL setzen:** `https://cedricnitsch.de/bestaetigung-danke.html`
4. **TEXT-Attribute anlegen:** Alle Custom-Attribute als Typ "Text" (siehe Tabelle unten)

---

## Technischer Ablauf

```
Nutzer füllt Futtercheck aus
    ↓ Klickt "Ergebnis anzeigen"
    ↓
┌─────────────────────────────────────────────────┐
│ Brevo REST API: POST /v3/contacts               │
│                                                  │
│ → Kontakt anlegen mit updateEnabled: false       │
│ → ALLE Custom-Attribute setzen:                  │
│   VORNAME, HUND_KATZE, TIERNAME,                 │
│   FUTTERCHECK_SCORE, FUTTERCHECK_FUTTER,         │
│   PARTNER_INTERESSE, FUTTERCHECK_FARB_SCORE, SMS │
│ → listIds: [5]                                   │
│ → Brevo erkennt: neuer Kontakt → sendet DOI-Mail │
└─────────────────────────────────────────────────┘
    ↓
    ├── ✅ Erfolg (201) → Kontakt erstellt, DOI-Mail wird gesendet
    │
    └── ⚠️ Duplikat (400 duplicate_parameter)
        ↓
    ┌─────────────────────────────────────────────┐
    │ Fallback: PUT /v3/contacts/{email}           │
    │ → Attribute aktualisieren                    │
    │ → Kontakt bleibt in bestehendem Status       │
    └─────────────────────────────────────────────┘
    ↓
Nutzer sieht sofort das Ergebnis (Score-Kreis)
    ↓
Nutzer erhält DOI-Mail → klickt Bestätigungslink
    ↓
Kontakt wird in Brevo aktiviert → Automation startet
```

---

## Code-Implementierung

### `sendToBrevo(email, vorname, attributes)` 

Hauptfunktion – erstellt Kontakt via `POST /contacts`:

```javascript
async function sendToBrevo(email, vorname, attributes) {
    const payload = {
        email: email,
        attributes: {
            VORNAME: vorname,
            HUND_KATZE: attributes.hundKatze,
            TIERNAME: attributes.tiername,
            FUTTERCHECK_SCORE: attributes.score,
            FUTTERCHECK_FUTTER: attributes.futter,
            PARTNER_INTERESSE: attributes.partnerInteresse,
            FUTTERCHECK_FARB_SCORE: attributes.farbScore,
            ...(attributes.sms && { SMS: attributes.sms })
        },
        listIds: [BREVO_LIST_ID],
        updateEnabled: false  // ← Wichtig für DOI!
    };
    // POST /v3/contacts
    // Bei 400 duplicate_parameter → updateBrevoContact()
}
```

### `updateBrevoContact(email, attributes)`

Fallback bei Duplikat – aktualisiert nur Attribute:

```javascript
async function updateBrevoContact(email, attributes) {
    const payload = { attributes: { ... } };
    // PUT /v3/contacts/{encodeURIComponent(email)}
}
```

### Schlüssel: `updateEnabled: false`

- **`updateEnabled: false`** = Brevo behandelt den Kontakt als NEU → DOI wird ausgelöst
- **`updateEnabled: true`** = Brevo würde bestehende Kontakte stillschweigend aktualisieren → KEIN DOI

---

## Kontakt-Attribute (alle TEXT)

> ℹ️ Alle Attribute sind Typ **TEXT** – keine Kategorie-Attribute mehr nötig.

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

## Entfernte Komponenten (Hybrid-Ansatz)

| Was | Warum entfernt |
|-----|----------------|
| Verstecktes `<form id="sib-form">` | Nicht mehr nötig – reine API |
| `sibforms.com/forms/.../sib-styles.css` | Externe Abhängigkeit entfernt |
| `sibforms.com/forms/.../main.js` | Externe Abhängigkeit entfernt |
| `BREVO_DOI_FORM_ACTION` Konstante | Formular-URL nicht mehr benötigt |
| `submitBrevoDOIForm()` Funktion | Ersetzt durch `sendToBrevo()` |
| `window.REQUIRED_CODE_ERROR_MESSAGE` | sibforms-spezifisch |
| `window.LOCALE` / `window.EMAIL_INVALID_MESSAGE` | sibforms-spezifisch |

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
📤 Sende an Brevo API... {email: "...", attributes: {...}}
✅ Brevo API: Kontakt erstellt
```
Oder bei Duplikat:
```
⚠️ Brevo: Kontakt existiert bereits – Attribute werden aktualisiert...
✅ Brevo: Attribute aktualisiert
```

### 4. E-Mail prüfen
- DOI-Bestätigungsmail erhalten?
- Bestätigungslink klicken → `bestaetigung-danke.html`?
- In Brevo: Kontakt jetzt aktiv mit allen Attributen?

---

## Fehlerbehandlung

| Fehler | Verhalten |
|--------|-----------|
| `201` Created | ✅ Kontakt erstellt, DOI-Mail unterwegs |
| `400` duplicate_parameter | ⚠️ Fallback: `PUT /contacts/{email}` |
| `400` anderer Fehler | ❌ Console Error, Ergebnis wird trotzdem angezeigt |
| Netzwerkfehler | ❌ Console Error, Ergebnis wird trotzdem angezeigt |

> **Wichtig:** Brevo-Fehler blockieren NICHT die Ergebnisanzeige. Der Nutzer sieht immer sein Ergebnis.

---

## Offene Punkte

- [ ] DOI in Brevo-Listeneinstellungen aktivieren + Template konfigurieren
- [ ] Live-Test: Quiz → DOI-Mail erhalten → Bestätigen → Kontakt prüfen
- [ ] Brevo-Automations einrichten (Trigger: Kontakt bestätigt → Ergebnis-Mail + Sequenz)
