# BREVO Partner-Sequenz – Setup-Dokumentation

## 📋 Übersicht

Diese Dokumentation beschreibt die Einrichtung der 5-teiligen E-Mail-Sequenz zur **Reico-Partnergewinnung** in Brevo (ehemals Sendinblue). Die Sequenz startet nach dem Double-Opt-In und der Ergebnis-Mail des Futterchecks.

---

## 📧 E-Mail-Übersicht mit Betreff & Preheader

| Mail | Tag | Datei | Betreff | Preheader |
|------|-----|-------|---------|-----------|
| **1** | Tag 1 | `Mail1.html` | `{{ contact.VORNAME }}, das glauben die meisten über Hundefutter (und liegen falsch)` | `5 Futter-Mythen, die deinem Vierbeiner schaden könnten – und was wirklich zählt.` |
| **2** | Tag 3 | `Mail2.html` | `Was macht Reico eigentlich anders, {{ contact.VORNAME }}?` | `Natürliche Zutaten, mineralisches Gleichgewicht – und warum Tierhalter begeistert sind.` |
| **3** | Tag 5 | `Mail3.html` | `Die Geschichte hinter Reico – und eine besondere Möglichkeit für dich` | `Warum ein Familienbetrieb aus dem Allgäu die Tiernahrung revolutioniert – und wie du Teil davon werden kannst.` |
| **4** | Tag 7 | `Mail4.html` | `Empfehlen statt verkaufen – so funktioniert das Reico-Business, {{ contact.VORNAME }}` | `Kein Lager, kein Risiko, keine Kaltakquise – echte Partner erzählen, wie es wirklich läuft.` |
| **5** | Tag 10 | `Mail5.html` | `{{ contact.VORNAME }}, deine Entscheidung – Partner werden oder erstmal Kunde bleiben?` | `Alles auf einen Blick: Nebeneinkommen, Community, Sinn – und warum jetzt der richtige Zeitpunkt ist.` |

---

## 🎯 Dramaturgie & Ziel pro Mail

| Mail | Fokus | Ziel |
|------|-------|------|
| **1** | Vertrauen & Edukation (Futter-Mythen) | Expertise aufbauen, erster sanfter Hinweis auf Empfehlungen |
| **2** | Reico-Vorteile & Social Proof | Produktvertrauen vertiefen, Business-Chance andeuten |
| **3** | Reico-Story & Partnerschaft vorstellen | Erstmals konkret: Was ist ein Reico-Partner? Vorteile, Kosten |
| **4** | Business-Modell & Verdienst-Transparenz | "Empfehlen statt Verkaufen" erklären, Partner-Stories, starker CTA |
| **5** | Urgency & Abschluss | Zusammenfassung, Haupt-CTA "Jetzt Partner werden" + Alternative "Erstmal Kunde" |

---

## 🔀 Segmentierung nach PARTNER_INTERESSE

### Kontakt-Attribut einrichten

1. **Brevo → Kontakte → Kontaktattribute verwalten**
2. Neues Attribut erstellen:
   - **Name:** `PARTNER_INTERESSE`
   - **Typ:** Text (oder Dropdown-Kategorie)
   - **Mögliche Werte:** `Stark`, `Leicht`, `Kunde`

### Werte zuweisen

Das Attribut `PARTNER_INTERESSE` wird idealerweise automatisch gesetzt, basierend auf:

#### Option A: Über den Futtercheck-Funnel (empfohlen)
- Im Funnel eine Frage einbauen: *"Interessierst du dich auch für ein Nebeneinkommen mit Tiernahrung?"*
  - **Ja, sehr** → `PARTNER_INTERESSE = "Stark"`
  - **Vielleicht, erzähl mir mehr** → `PARTNER_INTERESSE = "Leicht"`
  - **Nein, nur Futter** → `PARTNER_INTERESSE = "Kunde"`
- Per API/Webhook das Attribut beim Kontakt in Brevo setzen

#### Option B: Über Engagement-basierte Automation
- Wer auf Partner-Links klickt → automatisch auf `"Stark"` oder `"Leicht"` hochstufen
- Standard für neue Kontakte: `"Kunde"`

#### Option C: Manuelle Zuweisung
- Nach dem Erstgespräch manuell setzen
- Für kleinere Listen zu Beginn praktikabel

### Wie Conditional Content in den Mails funktioniert

Die Mails verwenden Brevo/Jinja2-Template-Syntax:

```html
{% if contact.PARTNER_INTERESSE == "Stark" %}
  <!-- Stärkerer CTA, z.B. "Jetzt Partner werden" -->
{% elsif contact.PARTNER_INTERESSE == "Leicht" %}
  <!-- Sanfterer CTA, z.B. "Unverbindlich informieren" -->
{% else %}
  <!-- Nur Kunden-CTA, z.B. "Kostenlose Beratung buchen" -->
{% endif %}
```

**Wichtig:** Falls das Attribut nicht gesetzt ist (z.B. bei Kontakten ohne Funnel-Daten), greift der `{% else %}`-Block – also die Kunden-Variante. So ist sichergestellt, dass niemand einen kaputten oder leeren Block sieht.

---

## ⏱️ Timing-Empfehlung

### Gesamter Funnel-Ablauf

```
Tag 0:  Futtercheck ausgefüllt → DOI-Mail (automatisch von Brevo)
Tag 0:  DOI bestätigt → Ergebnis-Mail (ergebnis_futtercheck.html)
Tag 1:  Mail 1 – Futter-Mythen & Vertrauen
Tag 3:  Mail 2 – Reico-Vorteile & Social Proof
Tag 5:  Mail 3 – Reico-Philosophie & Partnerschaft
Tag 7:  Mail 4 – Business-Modell & Verdienst
Tag 10: Mail 5 – Urgency & Abschluss
```

### Warum diese Abstände?

| Abstand | Begründung |
|---------|------------|
| Tag 1 (1 Tag nach Ergebnis) | Kontakt ist noch "warm", Edukation vertieft Vertrauen |
| Tag 3 (2 Tage Pause) | Genug Zeit zum Verdauen, aber nicht vergessen |
| Tag 5 (2 Tage Pause) | Partnerschaft erstmals vorstellen, nachdem Vertrauen aufgebaut |
| Tag 7 (2 Tage Pause) | Business-Details, wenn Interesse geweckt |
| Tag 10 (3 Tage Pause) | Längere Pause vor Abschluss-Mail = "letzte Chance"-Effekt |

### Optimale Sendezeit

- **Empfohlen:** 08:00–09:00 Uhr oder 18:00–19:00 Uhr (lokale Zeit)
- **Vermeiden:** Montags (voller Posteingang), Freitags nachmittags (Wochenende-Modus)
- **Best Practice:** Di–Do vormittags

---

## 🔧 Automation in Brevo einrichten

### Schritt-für-Schritt-Anleitung

#### 1. Neue Automation erstellen
- **Brevo → Automations → Neue Automation erstellen**
- Name: `Partner-Sequenz nach Futtercheck`

#### 2. Trigger/Einstiegspunkt definieren
- **Trigger:** "E-Mail wurde gesendet" ODER "Kontakt betritt eine Liste"
- **Bedingung:** Ergebnis-Mail (`ergebnis_futtercheck.html`) wurde erfolgreich versendet
- **Alternative:** Kontakt wird zur Liste "Futtercheck-Abschlüsse" hinzugefügt

#### 3. Wartezeiten & E-Mails einrichten

```
[Trigger: Ergebnis-Mail versendet]
    ↓
[Warte: 1 Tag]
    ↓
[Sende: Mail 1 (Mail1.html)]
    ↓
[Warte: 2 Tage]
    ↓
[Sende: Mail 2 (Mail2.html)]
    ↓
[Warte: 2 Tage]
    ↓
[Sende: Mail 3 (Mail3.html)]
    ↓
[Warte: 2 Tage]
    ↓
[Sende: Mail 4 (Mail4.html)]
    ↓
[Warte: 3 Tage]
    ↓
[Sende: Mail 5 (Mail5.html)]
    ↓
[Ende]
```

#### 4. Exit-Bedingungen (wichtig!)
- **Kontakt hat Calendly-Termin gebucht** → aus Automation entfernen (falls technisch trackbar)
- **Kontakt hat sich abgemeldet** → automatisch (Brevo Standard)
- **Kontakt hat sich als Partner registriert** → aus Automation entfernen

#### 5. E-Mail-Templates anlegen
Für jede Mail:
1. Brevo → Kampagnen → E-Mail-Templates → Neues Template
2. **"Code einfügen"** wählen (nicht den Drag-and-Drop-Editor)
3. HTML-Code der jeweiligen Mail einfügen
4. Betreffzeile und Preheader aus der Tabelle oben eintragen
5. Absender: `Cedric von Reico <info@cedricnitsch.de>`
6. Template speichern und in der Automation verknüpfen

---

## 📝 Verfügbare Brevo-Variablen

| Variable | Beschreibung | Verwendet in |
|----------|-------------|-------------|
| `{{ contact.VORNAME }}` | Vorname des Kontakts | Alle Mails (Anrede, Betreff) |
| `{{ contact.TIERNAME }}` | Name des Haustiers | Mail 1, 2, 3, 5 |
| `{{ contact.HUND_KATZE }}` | Tierart (Hund/Katze) | Optional nutzbar |
| `{{ contact.FUTTERCHECK_SCORE }}` | Ergebnis-Score | Optional nutzbar |
| `{{ contact.FUTTERCHECK_FUTTER }}` | Aktuelles Futter | Optional nutzbar |
| `{{ contact.FUTTERCHECK_FARB_SCORE }}` | Farbcode des Scores | Optional nutzbar |
| `{{ contact.PARTNER_INTERESSE }}` | Partner-Interesse-Segment | Mail 2, 3, 4, 5 (Conditional Content) |
| `{{ unsubscribe }}` | Abmelde-Link | Alle Mails (Footer) |

---

## ✅ Checkliste vor Go-Live

- [ ] Kontaktattribut `PARTNER_INTERESSE` in Brevo angelegt
- [ ] Alle 5 HTML-Templates in Brevo als Code-Templates erstellt
- [ ] Betreffzeilen und Preheader korrekt eingetragen
- [ ] Absender-Adresse verifiziert (`info@cedricnitsch.de`)
- [ ] Platzhalter-Links ersetzt:
  - [ ] `[CALENDLY-LINK]` → echte Calendly-URL
  - [ ] `[IMPRESSUM-LINK]` → echte Impressum-URL
  - [ ] `[DATENSCHUTZ-LINK]` → echte Datenschutz-URL
  - [ ] `[SHOP-LINK]` → echte Reico-Shop-URL (Mail 5)
  - [ ] `[WHATSAPP-NUMMER]` / `[WHATSAPP-NUMMER-OHNE-PLUS]` → echte WhatsApp-Nummer (Mail 5)
- [ ] Automation erstellt und mit korrekten Wartezeiten konfiguriert
- [ ] Exit-Bedingungen definiert
- [ ] Test-Versand an eigene E-Mail-Adresse für alle 5 Mails durchgeführt
- [ ] Mobile-Darstellung geprüft
- [ ] Conditional Content mit Test-Kontakten (Stark/Leicht/Kunde) verifiziert
- [ ] Automation aktiviert

---

## ⚠️ Rechtliche Hinweise

1. **Keine Heilversprechen** – Die Mails enthalten keine Aussagen, die eine medizinische Wirkung des Futters suggerieren
2. **Keine Einkommensgarantien** – Alle Verdienst-Angaben sind als Orientierungswerte gekennzeichnet mit dem Disclaimer: *"Ergebnisse sind individuell und hängen vom persönlichen Engagement ab. Es handelt sich nicht um eine Einkommensgarantie."*
3. **DSGVO-konform** – Alle Mails enthalten Abmelde-Link, Impressum und Datenschutz-Link
4. **DOI-Pflicht** – Die Sequenz startet erst nach bestätigtem Double-Opt-In
5. **Letzte-Mail-Hinweis** – Mail 5 kommuniziert klar, dass es die letzte automatische Nachricht ist

---

## 📊 Empfohlene KPIs zum Tracking

| KPI | Zielwert | Wo tracken |
|-----|----------|-----------|
| Öffnungsrate | > 40% | Brevo Analytics |
| Klickrate | > 5% | Brevo Analytics |
| Calendly-Buchungen | > 3% der Kontakte | Calendly + Brevo Events |
| Partner-Conversions | > 1% der Kontakte | Manuell / CRM |
| Abmelderate | < 2% pro Mail | Brevo Analytics |

---

*Erstellt am: Juni 2026 | Für: Cedric Nitsch – Reico-Recruiting-Funnel*
