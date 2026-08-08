# Brevo Setup-Anleitung: Ergebnis-Mail (Futtercheck)

## Übersicht

Die Datei `ergebnis_futtercheck.html` ist eine **personalisierte Ergebnis-Mail** mit Brevo Conditional Content. Sie zeigt dem Kontakt sein individuelles Futtercheck-Ergebnis an – farblich abgestuft nach Score (grün/gelb/rot).

**Voraussetzung:** Die Brevo-Kontaktattribute sind bereits angelegt und werden vom Futtercheck-System befüllt.

---

## Vorgeschlagener Betreff & Preheader

| Element | Text |
|---------|------|
| **Betreff** | `{{ contact.VORNAME }}, hier ist dein Futtercheck-Ergebnis für {{ contact.TIERNAME }} 🐾` |
| **Preheader** | `{{ contact.FUTTERCHECK_SCORE }} von 100 Punkten – was das für {{ contact.TIERNAME }} bedeutet.` |
| **Absender** | Cedric Nitsch · info@cedricnitsch.de |

---

## Schritt 1: Kontakt-Attribute prüfen

Stelle sicher, dass folgende Attribute in Brevo existieren (unter **Kontakte → Attribute**):

| Attribut | Typ | Werte |
|----------|-----|-------|
| `VORNAME` | Text | – |
| `TIERNAME` | Text | – |
| `HUND_KATZE` | Text | `hund` oder `katze` |
| `FUTTERCHECK_SCORE` | Zahl | 0–100 |
| `FUTTERCHECK_FUTTER` | Text | z.B. „Hochwertiges Nassfutter" |
| `FUTTERCHECK_FARB_SCORE` | Text | `gruen`, `gelb` oder `rot` |
| `PARTNER_INTERESSE` | Text | `Stark`, `Leicht` oder `Kunde` |

> Diese Attribute werden automatisch über die API beim Absenden des Futterchecks gesetzt.

---

## Schritt 2: E-Mail-Template in Brevo importieren

### Option A: Drag & Drop Editor (empfohlen)

1. Gehe zu **Kampagnen → E-Mail-Vorlagen → Neue Vorlage**
2. Wähle **Drag & Drop Editor**
3. Wähle ein leeres Layout
4. Klicke oben rechts auf **< > Code bearbeiten** (HTML-Ansicht)
5. **Lösche den gesamten bestehenden Code**
6. Öffne `ergebnis_futtercheck.html` in einem Texteditor
7. **Kopiere den gesamten HTML-Code** und füge ihn ein
8. Klicke auf **Speichern**
9. Benenne die Vorlage: `Futtercheck – Ergebnis-Mail (Variante B)`

### Option B: Klassischer HTML-Editor

1. Gehe zu **Kampagnen → E-Mail-Vorlagen → Neue Vorlage**
2. Wähle **Rich Text / HTML Editor**
3. Wechsle in die **HTML-Quellcode-Ansicht**
4. Füge den kompletten HTML-Code ein
5. Speichere als `Futtercheck – Ergebnis-Mail (Variante B)`

### Platzhalter anpassen

Ersetze vor dem Speichern folgende Platzhalter:

| Platzhalter | Ersetzen durch |
|-------------|----------------|
| `[CALENDLY-LINK]` | Dein tatsächlicher Calendly-Link (z.B. `https://calendly.com/cedricnitsch/futterberatung`) |

> Die `{{ contact.* }}` und `{{ unsubscribe }}` Platzhalter **nicht** ändern – diese werden von Brevo automatisch befüllt.

---

## Schritt 3: Test-Versand

Bevor du die Mail in eine Automation einbaust, teste sie:

1. Öffne die gespeicherte Vorlage
2. Klicke auf **Vorschau & Test**
3. Wähle **An eine Test-Adresse senden**
4. **Wichtig:** Teste mit 3 verschiedenen Test-Kontakten:

| Test-Kontakt | FUTTERCHECK_FARB_SCORE | FUTTERCHECK_SCORE | HUND_KATZE |
|--------------|------------------------|-------------------|------------|
| Test Grün | `gruen` | `82` | `hund` |
| Test Gelb | `gelb` | `55` | `katze` |
| Test Rot | `rot` | `28` | `hund` |

### Test-Kontakte anlegen:

1. Gehe zu **Kontakte → Kontakt hinzufügen**
2. Erstelle 3 Test-Kontakte mit den obigen Werten
3. Verwende deine eigene E-Mail-Adresse
4. Sende die Test-Mail an jeden Kontakt einzeln
5. Prüfe:
   - ✅ Personalisierung korrekt (Vorname, Tiername, Score)
   - ✅ Richtige Farb-Variante wird angezeigt (grün/gelb/rot)
   - ✅ Hund/Katze-Text korrekt
   - ✅ Mobilansicht testen (am Handy oder im Vorschau-Tool)
   - ✅ Alle Links funktionieren
   - ✅ Abmelde-Link funktioniert
   - ✅ Footer vollständig

---

## Schritt 4: Automation einrichten

### 4.1 Bestehende Automation erweitern

Die Ergebnis-Mail wird **nach der DOI-Bestätigung** automatisch verschickt:

1. Gehe zu **Automationen → Deine Futtercheck-Automation**
2. Öffne den Workflow-Editor

### 4.2 Workflow-Aufbau

Der gesamte Automations-Flow sieht so aus:

```
[Trigger: Kontakt in Liste "Futtercheck" hinzugefügt]
    │
    ▼
[DOI-Mail senden]
    │
    ▼
[Warten auf: Kontakt hat DOI bestätigt]
    │          (Event: Kontakt in Liste "DOI-bestätigt" verschoben)
    │
    ▼
[Wartezeit: 2 Minuten]
    │
    ▼
[★ ERGEBNIS-MAIL SENDEN ★]  ← Die neue Mail
    │
    ▼
[Wartezeit: 2 Tage]
    │
    ▼
[Folge-Mails (Mail 2–5)]
```

### 4.3 Ergebnis-Mail-Schritt hinzufügen

1. Klicke nach dem DOI-Bestätigungs-Schritt auf **+ Schritt hinzufügen**
2. Wähle **E-Mail senden**
3. Konfiguriere:
   - **Vorlage:** `Futtercheck – Ergebnis-Mail (Variante B)`
   - **Betreff:** `{{ contact.VORNAME }}, hier ist dein Futtercheck-Ergebnis für {{ contact.TIERNAME }} 🐾`
   - **Absendername:** `Cedric Nitsch`
   - **Absender-E-Mail:** `info@cedricnitsch.de`
4. Klicke auf **Speichern**

### 4.4 Wartezeit vor Ergebnis-Mail

Füge **vor** dem Ergebnis-Mail-Schritt eine kurze Wartezeit ein:

1. Klicke auf **+ Schritt hinzufügen** → **Wartezeit**
2. Setze auf **2 Minuten**
3. Das sorgt dafür, dass die Ergebnis-Mail nicht sofort nach DOI-Bestätigung kommt, was natürlicher wirkt

---

## Schritt 5: Conditional Content – So funktioniert er

### Brevo-Syntax

Die Mail verwendet Brevo Jinja2-Syntax für Conditional Content:

```jinja2
{% if contact.FUTTERCHECK_FARB_SCORE == "gruen" %}
  ... Inhalt für grünen Bereich ...
{% elsif contact.FUTTERCHECK_FARB_SCORE == "gelb" %}
  ... Inhalt für gelben Bereich ...
{% else %}
  ... Inhalt für roten Bereich (Fallback) ...
{% endif %}
```

### Was wird bedingt angezeigt?

| Bereich | Grün | Gelb | Rot |
|---------|------|------|-----|
| Score-Box Farbe | ✅ Grün (#16A34A) | ⚠️ Orange (#D97706) | 🔴 Rot (#DC2626) |
| Score-Box Rahmen | Grün | Orange | Rot |
| Bewertungstext | „Sehr gute Ernährungssituation" | „Verbesserungspotenzial vorhanden" | „Handlungsbedarf bei der Ernährung" |
| Auswertungstext | Optimierungs-Tipps | Konkrete Schwachpunkte | Dringender Handlungsbedarf |
| CTA-Text | „Das letzte Quäntchen rausholen" | „Konkrete Schritte" | „Schnell und unkompliziert verbessern" |

### Zusätzliche Bedingungen (Hund/Katze):

```jinja2
{% if contact.HUND_KATZE == "katze" %}Katzenfutter{% else %}Hundefutter{% endif %}
```

Diese steuern, ob „Hundefutter" oder „Katzenfutter" im Text steht.

---

## Schritt 6: Aktivierung & Go-Live

### Checkliste vor der Aktivierung:

- [ ] Alle 3 Farbvarianten getestet (grün, gelb, rot)
- [ ] Hund- und Katze-Variante getestet
- [ ] Mobile Darstellung geprüft
- [ ] Alle Links korrekt (Calendly, Impressum, Datenschutz)
- [ ] Abmelde-Link funktioniert
- [ ] Betreff mit Personalisierung getestet
- [ ] Preheader korrekt angezeigt
- [ ] Kein Disclaimer/Heilversprechen im Text

### Automation aktivieren:

1. Überprüfe den gesamten Workflow
2. Klicke auf **Aktivieren** (oben rechts)
3. Bestätige die Aktivierung

---

## Troubleshooting

### Problem: Conditional Content wird nicht richtig angezeigt

**Ursache:** Attribut `FUTTERCHECK_FARB_SCORE` ist leer oder hat einen unerwarteten Wert.

**Lösung:**
- Prüfe in der API/Webhook-Integration, ob das Attribut korrekt gesetzt wird
- Erlaubte Werte: exakt `gruen`, `gelb`, `rot` (Kleinschreibung, keine Umlaute)
- Der `else`-Block (rot) greift als Fallback, wenn der Wert weder `gruen` noch `gelb` ist

### Problem: Personalisierung zeigt leere Felder

**Ursache:** Kontakt-Attribute wurden nicht rechtzeitig befüllt.

**Lösung:**
- Stelle sicher, dass die API die Attribute **vor** dem Automation-Trigger setzt
- Die 2-Minuten-Wartezeit nach DOI hilft, Race Conditions zu vermeiden

### Problem: Mail landet im Spam

**Lösung:**
- SPF, DKIM und DMARC für die Absender-Domain konfigurieren
- Brevo Sender-Verifizierung durchführen
- Vermeidung von Spam-Trigger-Wörtern (bereits im Template berücksichtigt)

---

## Dateiübersicht

| Datei | Beschreibung |
|-------|-------------|
| `ergebnis_futtercheck.html` | HTML-Template der Ergebnis-Mail mit Brevo Conditional Content |
| `BREVO_ERGEBNIS_MAIL_SETUP.md` | Diese Setup-Anleitung |

---

*Letzte Aktualisierung: Juni 2026*
