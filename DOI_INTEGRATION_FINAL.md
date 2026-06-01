# DOI-Integration – Finale Dokumentation

> **Stand:** 01.06.2026 | **Branch:** `recruiting-funnel-optimization`

---

## Überblick

Die DOI (Double-Opt-In) Integration nutzt das **offizielle Brevo-Formular** (sibforms.com) für die Bestätigungsmail, kombiniert mit der **Brevo REST API** für Custom-Attribute.

### Warum dieser Hybrid-Ansatz?

| Ansatz | Status | Problem |
|--------|--------|---------|
| Brevo API `/contacts/doubleOptinConfirmation` | ❌ Hat nicht funktioniert | Template-ID-Setup komplex, CORS-Probleme |
| Brevo DOI-Formular (sibforms) | ✅ Funktioniert | Kann nur Basis-Felder (Email, Vorname, SMS) |
| **Hybrid: API + Formular** | ✅ **Aktuelle Lösung** | API setzt Attribute, Formular löst DOI aus |

---

## Technischer Ablauf

```
Nutzer füllt Futtercheck aus
    ↓ Klickt "Ergebnis anzeigen"
    ↓
┌─────────────────────────────────────────────────┐
│ SCHRITT 1: Brevo REST API (/v3/contacts)        │
│ → Kontakt anlegen/aktualisieren                 │
│ → ALLE Custom-Attribute setzen:                 │
│   VORNAME, HUND_KATZE, TIERNAME,                │
│   FUTTERCHECK_SCORE, FUTTERCHECK_FUTTER,        │
│   PARTNER_INTERESSE, FUTTERCHECK_FARB_SCORE     │
│ → updateEnabled: true                           │
│ → listIds: [5]                                  │
└─────────────────────────────────────────────────┘
    ↓
┌─────────────────────────────────────────────────┐
│ SCHRITT 2: Brevo DOI-Formular (sibforms)        │
│ → Verstecktes <form> wird programmatisch        │
│   ausgefüllt und abgesendet                     │
│ → sibforms main.js fängt Submit ab              │
│ → AJAX POST an sibforms.com                     │
│ → Brevo sendet DOI-Bestätigungsmail             │
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

Das offizielle Brevo-Formular ist unsichtbar im DOM eingebettet (`position:absolute; left:-9999px`).

### Formular-Action
```
https://527052f3.sibforms.com/serve/MUIFAEsoUsu2lQEwZbEGWqu3v6u9wL2SIPPUymK-A_Vc0QPh5brKiyc__E3MTjN9YYhO3Tqv-YtqYTJeAH6h3IV0Mt5ECrXtrWv73icuhdF0rgKNrVbQDxNTrQLccP0-cvUBbw9TcGjJobMlR1757jpWbQmfb6FPlxkFczQhk0UQ5mCi5IIPD5GZPTV7gIyGgCy2PSFGJ-ObCxEZ1g==
```

### Formulardaten

| Feld | Name | Typ | Beschreibung |
|------|------|-----|-------------|
| Vorname | `VORNAME` | text, required | Aus Quiz-Antwort |
| E-Mail | `EMAIL` | text, required | Aus Lead-Formular |
| WhatsApp | `SMS` | tel, optional | Ohne Ländervorwahl |
| Ländervorwahl | `SMS__COUNTRY_CODE` | select | Default: `+49 DE` |
| Honeypot | `email_address_check` | hidden, leer | Anti-Bot (muss leer bleiben!) |
| Locale | `locale` | hidden | `de` |

### Externe Abhängigkeiten

| Ressource | URL | Zweck |
|-----------|-----|-------|
| sibforms CSS | `https://sibforms.com/forms/end-form/build/sib-styles.css` | Formular-Styles (minimal, da versteckt) |
| sibforms JS | `https://sibforms.com/forms/end-form/build/main.js` | AJAX-Form-Handler für DOI |

---

## Geänderte Dateien

| Datei | Änderung |
|-------|----------|
| `futtercheck.html` | API-Endpoint von `/contacts/doubleOptinConfirmation` auf `/contacts` umgestellt + verstecktes Brevo-Formular für DOI + sibforms JS |
| `.nojekyll` | Neu – deaktiviert Jekyll für GitHub Pages (behebt Liquid-Syntax-Fehler) |
| `mails/BREVO_ERGEBNIS_MAIL_SETUP.md` | Bare Liquid-Tag escaped |
| `mails/BREVO_PARTNER_SEQUENZ_SETUP.md` | Bare Liquid-Tag escaped |

### Entfernt

| Was | Grund |
|-----|-------|
| `DOI_TEMPLATE_ID` Platzhalter | Nicht mehr benötigt – DOI über sibforms |
| `DOI_REDIRECTION_URL` | Nicht mehr benötigt – Redirect wird von Brevo-Formular gesteuert |
| `sendToBrevoFallback()` | Ersetzt durch 2-Schritt-Ansatz (API + Formular) |

---

## Brevo-Attribute (unverändert)

Die API setzt weiterhin alle Custom-Attribute:

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
    "SMS": "015678516818"
  },
  "listIds": [5],
  "updateEnabled": true
}
```

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
✅ Brevo API: Custom-Attribute gesetzt
📦 Attribute: { VORNAME: "Max", HUND_KATZE: "hund", ... }
📨 DOI-Formular wird abgesendet …
```

### 4. E-Mail prüfen
- DOI-Bestätigungsmail erhalten?
- Bestätigungslink klicken
- In Brevo: Kontakt jetzt aktiv mit allen Attributen?

---

## GitHub Pages Fix

### Problem
GitHub Pages nutzt Jekyll, das Liquid-Syntax (`{% if %}`, `{% else %}`, etc.) in Markdown-Dateien interpretiert. Brevo-Code-Beispiele in der Dokumentation haben Jekyll-Build-Fehler verursacht.

### Lösung
`.nojekyll` Datei im Root-Verzeichnis. Diese deaktiviert Jekyll vollständig – ideal für reine HTML/CSS/JS-Seiten ohne Jekyll-Template-Engine.

---

## Offene Punkte

- [ ] Live-Test: Quiz ausfüllen → DOI-Mail erhalten → Bestätigen → Kontakt prüfen
- [ ] Brevo-Automations einrichten (Trigger: Kontakt bestätigt → Ergebnis-Mail + Sequenz)
- [ ] Prüfen, ob die Brevo-Formular-Redirect-URL korrekt konfiguriert ist (in Brevo-Formular-Settings)
