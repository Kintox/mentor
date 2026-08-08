# DOI Setup Komplett-Anleitung für Reico Futtercheck

**Für: Cedric Nitsch**  
**Stand: Juni 2026**  
**Version: 1.0**

---

## 📋 INHALTSVERZEICHNIS

1. [Übersicht - Was ist DOI und warum brauchen wir es?](#1-übersicht)
2. [Schritt-für-Schritt Brevo-Konfiguration](#2-schritt-für-schritt-brevo-konfiguration)
3. [Testing-Checkliste](#3-testing-checkliste)
4. [Häufige Fehler & Troubleshooting](#4-häufige-fehler--troubleshooting)
5. [Go-Live Checkliste](#5-go-live-checkliste)
6. [Support & Weitere Ressourcen](#6-support--weitere-ressourcen)

---

## 1. ÜBERSICHT

### Was ist DOI und warum ist es wichtig?

**DOI** steht für **Double Opt-In** – ein zweistufiges Anmeldeverfahren, bei dem der Nutzer seine E-Mail-Adresse zweimal bestätigen muss:

1. **Erste Anmeldung**: Nutzer füllt den Futtercheck aus
2. **Zweite Bestätigung**: Nutzer klickt auf einen Link in einer Bestätigungs-E-Mail

**Warum ist das wichtig?**
- ✅ **DSGVO-Konformität**: DOI ist in Deutschland rechtlich erforderlich für Newsletter und Marketing-E-Mails
- ✅ **Schutz vor Spam-Anmeldungen**: Nur echte E-Mail-Adressen werden aktiviert
- ✅ **Bessere Listen-Qualität**: Nur wirklich interessierte Kontakte in deiner Liste
- ✅ **Höhere Zustellbarkeit**: E-Mail-Provider bewerten DOI-Listen besser

---

### Der neue Prozess im Überblick

```
┌─────────────────────────────────────────────────────────────────┐
│ SCHRITT 1: NUTZER FÜLLT FUTTERCHECK AUS                        │
│ (futtercheck.html)                                              │
│                                                                 │
│ - Name, E-Mail, Telefon                                         │
│ - Kategorien (Tierart, Fütterung, etc.)                        │
└────────────────────┬────────────────────────────────────────────┘
                     │
                     ▼
┌─────────────────────────────────────────────────────────────────┐
│ SCHRITT 2: DATEN WERDEN AN BREVO GESENDET                      │
│ (über Brevo API)                                                │
│                                                                 │
│ - Kontakt wird als "NICHT BESTÄTIGT" angelegt                  │
│ - DOI-Mail wird automatisch verschickt                         │
│ - Alle Kategorie-Daten werden gespeichert                      │
└────────────────────┬────────────────────────────────────────────┘
                     │
                     ▼
┌─────────────────────────────────────────────────────────────────┐
│ SCHRITT 3: NUTZER ERHÄLT DOI-BESTÄTIGUNGS-MAIL                 │
│ (doi_bestaetigung.html Template)                               │
│                                                                 │
│ Betreff: "Bitte bestätige deine E-Mail-Adresse"                │
│ Inhalt: Freundliche Aufforderung mit Bestätigungs-Button       │
└────────────────────┬────────────────────────────────────────────┘
                     │
                     ▼
┌─────────────────────────────────────────────────────────────────┐
│ SCHRITT 4: NUTZER KLICKT AUF BESTÄTIGUNGSLINK                  │
│                                                                 │
│ - Link enthält eindeutigen DOI-Token                           │
│ - Brevo registriert die Bestätigung                            │
│ - Kontakt wird auf "BESTÄTIGT" gesetzt                         │
└────────────────────┬────────────────────────────────────────────┘
                     │
                     ▼
┌─────────────────────────────────────────────────────────────────┐
│ SCHRITT 5: WEITERLEITUNG ZUR DANKE-SEITE                       │
│ (bestaetigung-danke.html)                                       │
│                                                                 │
│ - Nutzer sieht Bestätigung                                     │
│ - Info: "Du erhältst gleich deine Auswertung"                  │
└────────────────────┬────────────────────────────────────────────┘
                     │
                     ▼
┌─────────────────────────────────────────────────────────────────┐
│ SCHRITT 6: BREVO AUTOMATION WIRD AUSGELÖST                     │
│ (Trigger: "Kontakt hat E-Mail bestätigt")                      │
│                                                                 │
│ ┌─────────────────────────────────────────────────────────┐   │
│ │ Warte 1 Minute (System-Synchronisation)                 │   │
│ └────────────┬────────────────────────────────────────────┘   │
│              ▼                                                  │
│ ┌─────────────────────────────────────────────────────────┐   │
│ │ Sende ERGEBNIS-MAIL (ergebnis_futtercheck.html)        │   │
│ │ → Personalisierte Auswertung mit Conditional Content   │   │
│ └────────────┬────────────────────────────────────────────┘   │
│              ▼                                                  │
│ ┌─────────────────────────────────────────────────────────┐   │
│ │ Warte 1 Tag                                             │   │
│ └────────────┬────────────────────────────────────────────┘   │
│              ▼                                                  │
│ ┌─────────────────────────────────────────────────────────┐   │
│ │ PARTNER-SEQUENZ STARTET (Mail 1-5)                     │   │
│ │ → Aufbau Vertrauen → Mehrwert → Partner-Angebot        │   │
│ └─────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────┘
```

**Wichtige Punkte:**
- ⏱️ **Timing**: Ergebnis-Mail kommt erst NACH der DOI-Bestätigung (nicht sofort)
- 🔒 **Sicherheit**: Nur bestätigte Kontakte erhalten Marketing-Mails
- 📊 **Tracking**: Du siehst in Brevo genau, wer bestätigt hat und wer nicht

---

## 2. SCHRITT-FÜR-SCHRITT BREVO-KONFIGURATION

### SCHRITT 1: DOI-Template in Brevo anlegen

**Dauer: ca. 10 Minuten**

#### 1.1 Template erstellen

1. **Melde dich bei Brevo an** und gehe zu:
   ```
   Hauptmenü → Transactional → Email Templates
   ```

2. **Klicke oben rechts auf "Create Template"**

3. **Wähle "Rich Text Editor"** (NICHT "Drag & Drop Editor")
   - Du brauchst die Möglichkeit, HTML-Code einzufügen

4. **Template-Name eingeben**:
   ```
   Reico Futtercheck - DOI Bestätigung
   ```

5. **Betreff-Zeile (Subject)**:
   ```
   {{params.VORNAME}}, bitte bestätige deine E-Mail-Adresse
   ```
   ⚠️ **Wichtig**: Der Platzhalter `{{params.VORNAME}}` wird automatisch ersetzt

#### 1.2 HTML-Code einfügen

1. **Wechsle zum "HTML-Modus"** (Button oben rechts im Editor)

2. **Öffne die Datei** `doi_bestaetigung.html` aus deinem Repository

3. **Kopiere den GESAMTEN Inhalt** und füge ihn in den Brevo-Editor ein

4. **Prüfe, dass folgende Platzhalter vorhanden sind**:
   - `{{ doiURL }}` → Der Bestätigungslink (wird von Brevo automatisch generiert)
   - `{{ params.VORNAME }}` → Personalisierung mit Vornamen

#### 1.3 Vorschau & Test

1. **Klicke auf "Preview"** (Vorschau)
   - Prüfe, ob das Design korrekt angezeigt wird
   - Der Button "Jetzt E-Mail bestätigen" sollte gut sichtbar sein

2. **Speichern**: Klicke auf "Save" unten rechts

3. **WICHTIG: Template-ID notieren!**
   
   Nach dem Speichern wirst du zur Template-Übersicht weitergeleitet.
   
   - **Klicke auf dein gerade erstelltes Template**
   - **Schau in die Browser-URL**:
     ```
     https://app.brevo.com/template/edit/123
                                           ^^^
                                        Das ist deine Template-ID!
     ```
   
   - **Schreibe diese Zahl auf** (z.B. `123`)
   - Du brauchst sie im nächsten Schritt!

#### 1.4 Testversand durchführen (OPTIONAL aber empfohlen)

1. **Gehe zu**: `Transactional → Email API → Overview`

2. **Klicke auf "Send a test email"**

3. **Fülle aus**:
   - Template: "Reico Futtercheck - DOI Bestätigung"
   - Empfänger: Deine eigene E-Mail
   - Parameters:
     ```json
     {
       "VORNAME": "Test"
     }
     ```

4. **Sende ab** und prüfe dein Postfach
   - ⚠️ Der `{{ doiURL }}` Platzhalter wird im Test NICHT ersetzt (ist normal)
   - Prüfe nur Design und Personalisierung

---

### SCHRITT 2: Template-ID im Code einsetzen

**Dauer: 2 Minuten**

#### 2.1 Datei öffnen

1. **Öffne** `futtercheck.html` in deinem Code-Editor (z.B. VS Code)

2. **Suche nach Zeile ~227** (oder drücke `Strg+F` und suche nach `DOI_TEMPLATE_ID`)

#### 2.2 Template-ID ersetzen

Du siehst diesen Code:
```javascript
templateId: DOI_TEMPLATE_ID, // ← HIER ECHTE TEMPLATE-ID EINTRAGEN
```

**Ersetze `DOI_TEMPLATE_ID` durch deine echte Template-ID** (die Zahl aus Schritt 1.3):

```javascript
templateId: 123, // ← Beispiel: Template-ID 123
```

**Vollständiges Beispiel**:
```javascript
// DOI-E-Mail über Brevo versenden
const emailResponse = await fetch('https://api.brevo.com/v3/smtp/email', {
  method: 'POST',
  headers: {
    'accept': 'application/json',
    'api-key': BREVO_API_KEY,
    'content-type': 'application/json'
  },
  body: JSON.stringify({
    sender: { name: 'Reico Futtercheck', email: 'noreply@cedricnitsch.de' },
    to: [{ email: formData.email }],
    templateId: 123, // ← DEINE Template-ID
    params: {
      VORNAME: formData.vorname
    }
  })
});
```

#### 2.3 Speichern und hochladen

1. **Speichere die Datei** (`Strg+S`)

2. **Lade die Datei auf deinen Server hoch**
   - Via FTP/SFTP zu deinem Webspace
   - Oder über dein Hosting-Panel (z.B. Webgo, All-Inkl, etc.)

3. **Teste die URL**:
   ```
   https://cedricnitsch.de/futtercheck.html
   ```
   (oder wo auch immer dein Futtercheck liegt)

---

### SCHRITT 3: Weiterleitungs-URL konfigurieren

**Dauer: 5 Minuten**

#### 3.1 Bestätigungsseite hochladen

1. **Nimm die Datei** `bestaetigung-danke.html` aus deinem Repository

2. **Lade sie auf deinen Server hoch** (in den gleichen Ordner wie `futtercheck.html`)

3. **Teste die URL im Browser**:
   ```
   https://cedricnitsch.de/bestaetigung-danke.html
   ```
   
   Du solltest eine Seite mit der Überschrift sehen:
   ```
   ✅ E-Mail erfolgreich bestätigt!
   ```

#### 3.2 URL in futtercheck.html prüfen

1. **Öffne nochmal** `futtercheck.html`

2. **Suche nach** `redirectionUrl` (Zeile ~230)

3. **Stelle sicher, dass die URL korrekt ist**:
   ```javascript
   redirectionUrl: 'https://cedricnitsch.de/bestaetigung-danke.html'
   ```

4. **Falls du eine andere Domain verwendest**, passe die URL an

5. **Speichern & hochladen** (falls geändert)

---

### SCHRITT 4: Brevo-Attribute prüfen/anlegen

**Dauer: 15 Minuten**

Die Futtercheck-Kategorien werden als **Contact Attributes** (Kontaktfelder) in Brevo gespeichert. Du musst diese Felder VORHER anlegen, damit Brevo die Daten korrekt speichern kann.

#### 4.1 Zu den Contact Attributes navigieren

1. **Gehe in Brevo zu**:
   ```
   Hauptmenü → Contacts → Settings → Contact Attributes
   ```

2. **Klicke auf** "Create a new attribute"

#### 4.2 Alle Attribute anlegen

Lege nacheinander folgende Attribute an. **WICHTIG**: Die Namen müssen EXAKT so geschrieben werden (Groß-/Kleinschreibung beachten!)

---

**ATTRIBUT 1: VORNAME**

- **Category**: NORMAL
- **Type**: Text
- **Name**: `VORNAME`
- **Beschreibung** (optional): "Vorname des Kontakts"

**Klicke auf "Create"**

---

**ATTRIBUT 2: TIERART**

- **Category**: CATEGORY
- **Type**: Text
- **Name**: `TIERART`
- **Beschreibung** (optional): "Tierart (Hund, Katze, Pferd)"

**Klicke auf "Create"**

**Nach dem Erstellen**: Klicke auf das Attribut und füge folgende **erlaubte Werte** hinzu:
- `Hund`
- `Katze`
- `Pferd`

---

**ATTRIBUT 3: GROESSE**

- **Category**: CATEGORY
- **Type**: Text
- **Name**: `GROESSE`
- **Beschreibung** (optional): "Größe des Hundes"

**Erlaubte Werte**:
- `Klein (bis 10 kg)`
- `Mittel (10-25 kg)`
- `Groß (über 25 kg)`

---

**ATTRIBUT 4: ALTER**

- **Category**: CATEGORY
- **Type**: Text
- **Name**: `ALTER`

**Erlaubte Werte**:
- `Welpe (0-1 Jahr)`
- `Erwachsen (1-7 Jahre)`
- `Senior (7+ Jahre)`

---

**ATTRIBUT 5: AKTIVITAET**

- **Category**: CATEGORY
- **Type**: Text
- **Name**: `AKTIVITAET`

**Erlaubte Werte**:
- `Wenig`
- `Mittel`
- `Sehr aktiv`

---

**ATTRIBUT 6: GESUNDHEITSZUSTAND**

- **Category**: CATEGORY
- **Type**: Text
- **Name**: `GESUNDHEITSZUSTAND`

**Erlaubte Werte**:
- `Gesund`
- `Empfindlicher Magen`
- `Allergien`
- `Übergewicht`
- `Gelenkprobleme`
- `Sonstige`

---

**ATTRIBUT 7: FUETTERUNGSART**

- **Category**: CATEGORY
- **Type**: Text
- **Name**: `FUETTERUNGSART`

**Erlaubte Werte**:
- `Trockenfutter`
- `Nassfutter`
- `BARF`
- `Selbstgekocht`
- `Mix`

---

**ATTRIBUT 8: ZUFRIEDENHEIT**

- **Category**: CATEGORY
- **Type**: Text
- **Name**: `ZUFRIEDENHEIT`

**Erlaubte Werte**:
- `Sehr zufrieden`
- `Zufrieden`
- `Geht so`
- `Unzufrieden`

---

**ATTRIBUT 9: PARTNER_INTERESSE**

- **Category**: CATEGORY
- **Type**: Text
- **Name**: `PARTNER_INTERESSE`
- **Beschreibung**: "Interesse an Reico-Partnerschaft (wird über Links in Sequenz-Mails gesetzt)"

**Erlaubte Werte**:
- `Hoch` (hat auf "Ja"-Link geklickt)
- `Niedrig` (hat auf "Nein"-Link geklickt)
- `Unbekannt` (noch nicht geklickt)

---

#### 4.3 Attribute überprüfen

Nach dem Anlegen solltest du **9 Attribute** in der Liste sehen:

```
✅ VORNAME
✅ TIERART
✅ GROESSE
✅ ALTER
✅ AKTIVITAET
✅ GESUNDHEITSZUSTAND
✅ FUETTERUNGSART
✅ ZUFRIEDENHEIT
✅ PARTNER_INTERESSE
```

**Tipp**: Mache einen Screenshot dieser Liste für deine Unterlagen!

---

### SCHRITT 5: Ergebnis-Mail & Partner-Sequenz importieren

**Dauer: 30-45 Minuten**

#### 5.1 Ergebnis-Mail erstellen

1. **Gehe zu**:
   ```
   Campaigns → Email Campaigns → Create Campaign
   ```

2. **Wähle**: "Regular email campaign"

3. **Name der Kampagne**:
   ```
   Futtercheck Ergebnis-Mail (Automation)
   ```

4. **Subject (Betreff)**:
   ```
   {{contact.VORNAME}}, dein persönliches Futtercheck-Ergebnis ist da! 🐾
   ```

5. **Wähle**: "Rich Text Editor" (HTML-Modus)

6. **Öffne die Datei** `ergebnis_futtercheck.html` und kopiere den kompletten Code

7. **Füge den Code ein** und klicke auf "Save & Next"

8. **Absender einstellen**:
   - Name: `Cedric von Reico`
   - E-Mail: `noreply@cedricnitsch.de` (oder deine Haupt-E-Mail)

9. **Preview-Text** (optional):
   ```
   Individuelle Fütterungsempfehlung für {{contact.TIERART}}
   ```

10. **WICHTIG: Platzhalter ersetzen**
    
    Durchsuche das HTML nach folgenden Platzhaltern und ersetze sie:
    
    - `CALENDLY_LINK_HIER` → Dein echter Calendly-Link (z.B. `https://calendly.com/cedricnitsch/beratung`)
    - `SHOP_LINK_HIER` → Link zu deinem Reico-Shop (z.B. `https://www.reico-vital.com/cedricnitsch`)
    - `WHATSAPP_NUMMER_HIER` → Deine WhatsApp-Nummer im Format `491234567890` (ohne +, Leerzeichen oder Bindestriche)

11. **Speichere die Kampagne**

12. **NICHT ABSENDEN!** Diese Mail wird über die Automation verschickt

---

#### 5.2 Partner-Sequenz-Mails importieren (Mail 1-5)

**Wiederhole für JEDE der 5 Mails:**

**MAIL 1: Einführung & Vertrauensaufbau**
- Datei: `Mail1.html`
- Name: `Partner-Sequenz Mail 1 - Einführung`
- Betreff: `{{contact.VORNAME}}, wie geht es [Tiername] mit der neuen Fütterung?`

**MAIL 2: Meine Geschichte**
- Datei: `Mail2.html`
- Name: `Partner-Sequenz Mail 2 - Meine Geschichte`
- Betreff: `Warum ich Partner bei Reico wurde...`

**MAIL 3: Erfolgsgeschichten**
- Datei: `Mail3.html`
- Name: `Partner-Sequenz Mail 3 - Erfolgsgeschichten`
- Betreff: `Diese 3 Partner-Typen sind bei Reico erfolgreich`

**MAIL 4: Interesse abfragen**
- Datei: `Mail4.html`
- Name: `Partner-Sequenz Mail 4 - Interesse abfragen`
- Betreff: `{{contact.VORNAME}}, eine Frage...`

**MAIL 5: Finale Einladung**
- Datei: `Mail5.html`
- Name: `Partner-Sequenz Mail 5 - Finale Einladung`
- Betreff: `Deine Chance: Werde Teil unseres Teams`

**Für jede Mail**:
1. Create Campaign → Regular email campaign
2. Name eingeben (siehe oben)
3. Betreff eingeben (siehe oben)
4. Rich Text Editor → HTML-Code einfügen
5. Platzhalter ersetzen (Calendly, Shop, WhatsApp)
6. Speichern (NICHT absenden!)

---

#### 5.3 Tracking-Links in Mail 4 einrichten

**WICHTIG**: Mail 4 enthält zwei Buttons, mit denen das `PARTNER_INTERESSE` gesetzt wird.

**Button 1: "Ja, erzähl mir mehr!"**
- Dieser Link muss auf eine URL zeigen, die das Attribut `PARTNER_INTERESSE` auf `Hoch` setzt
- Brevo-Syntax:
  ```
  https://app.brevo.com/update-contact/{{contact.id}}?PARTNER_INTERESSE=Hoch&redirect=https://cedricnitsch.de/interesse-danke.html
  ```

**Button 2: "Nein, nicht interessiert"**
- Setzt `PARTNER_INTERESSE` auf `Niedrig`
- Brevo-Syntax:
  ```
  https://app.brevo.com/update-contact/{{contact.id}}?PARTNER_INTERESSE=Niedrig&redirect=https://cedricnitsch.de/keine-interesse.html
  ```

**Optional**: Erstelle die Weiterleitungs-Seiten (`interesse-danke.html`, `keine-interesse.html`) oder verwende deine Hauptseite als Redirect.

---

### SCHRITT 6: Automation einrichten

**Dauer: 20 Minuten**

Dies ist der wichtigste Schritt! Die Automation sorgt dafür, dass nach der DOI-Bestätigung automatisch die Ergebnis-Mail und die Partner-Sequenz verschickt werden.

#### 6.1 Neue Automation erstellen

1. **Gehe zu**:
   ```
   Automation → Workflows → Create Workflow
   ```

2. **Wähle**: "Start from scratch" (oder "Leeres Template")

3. **Name der Automation**:
   ```
   Futtercheck DOI-Bestätigung → Ergebnis & Partner-Sequenz
   ```

---

#### 6.2 TRIGGER konfigurieren (KRITISCH!)

**Dies ist der wichtigste Teil!**

1. **Klicke auf** "Add a trigger"

2. **Wähle**: "Contact confirms email (DOI)"
   
   ⚠️ **NICHT "Contact added to list"** oder "Contact created"!
   
   ⚠️ **WICHTIG**: Nur "Contact confirms email (DOI)" stellt sicher, dass die Mails erst NACH der Bestätigung verschickt werden!

3. **Optional: Liste auswählen**
   - Falls du eine spezielle Liste für Futtercheck-Kontakte hast, wähle sie hier aus
   - Sonst: "All contacts"

4. **Speichern**

---

#### 6.3 Aktion 1: Warte 1 Minute

**Warum?** Das gibt Brevo Zeit, die DOI-Bestätigung vollständig zu verarbeiten und alle Attribute zu synchronisieren.

1. **Klicke auf** "+" (Aktion hinzufügen)

2. **Wähle**: "Wait"

3. **Einstellung**:
   - Duration: `1 minute`

4. **Speichern**

---

#### 6.4 Aktion 2: Ergebnis-Mail senden

1. **Klicke auf** "+" (nächste Aktion)

2. **Wähle**: "Send an email"

3. **Wähle die Kampagne**:
   ```
   Futtercheck Ergebnis-Mail (Automation)
   ```
   (Die Mail, die du in Schritt 5.1 erstellt hast)

4. **Speichern**

---

#### 6.5 Aktion 3: Warte 1 Tag

**Warum?** Wir geben dem Nutzer Zeit, die Ergebnis-Mail zu lesen und ggf. zu bestellen. Dann startet die Partner-Sequenz.

1. **Klicke auf** "+"

2. **Wähle**: "Wait"

3. **Einstellung**:
   - Duration: `1 day`

4. **Speichern**

---

#### 6.6 Aktion 4-8: Partner-Sequenz (Mail 1-5)

**Jetzt bauen wir die 5er-Sequenz:**

**Aktion 4: Mail 1 senden**
- "Send an email" → `Partner-Sequenz Mail 1 - Einführung`

**Aktion 5: Warte 3 Tage**
- "Wait" → `3 days`

**Aktion 6: Mail 2 senden**
- "Send an email" → `Partner-Sequenz Mail 2 - Meine Geschichte`

**Aktion 7: Warte 3 Tage**
- "Wait" → `3 days`

**Aktion 8: Mail 3 senden**
- "Send an email" → `Partner-Sequenz Mail 3 - Erfolgsgeschichten`

**Aktion 9: Warte 3 Tage**
- "Wait" → `3 days`

**Aktion 10: Mail 4 senden**
- "Send an email" → `Partner-Sequenz Mail 4 - Interesse abfragen`

**Aktion 11: Warte 4 Tage**
- "Wait" → `4 days`

**Aktion 12: Mail 5 senden**
- "Send an email" → `Partner-Sequenz Mail 5 - Finale Einladung`

---

#### 6.7 Optional: Verzweigung nach PARTNER_INTERESSE

**Fortgeschritten**: Du kannst nach Mail 4 eine Verzweigung einbauen:

1. **Nach Aktion 10** (Mail 4): Klicke auf "+" → "Add a condition"

2. **Bedingung**:
   - IF `PARTNER_INTERESSE` is `Hoch`
   - THEN: Sende eine zusätzliche Mail mit mehr Infos oder lade zu einem Call ein
   - ELSE: Normale Sequenz fortsetzen

**Das kannst du später optimieren!** Für den Start reicht die lineare Sequenz.

---

#### 6.8 Automation aktivieren

1. **Klicke oben rechts auf** "Activate workflow"

2. **Bestätige**: "Yes, activate"

3. **Fertig!** 🎉

**Deine Automation ist jetzt aktiv und läuft automatisch für jeden neuen DOI-bestätigten Kontakt!**

---

## 3. TESTING-CHECKLISTE

**Bevor du live gehst, teste ALLES!**

### Vorbereitungen

- [ ] **DOI-Template-ID im Code eingesetzt** (futtercheck.html, Zeile ~227)
- [ ] **Alle 9 Brevo-Attribute angelegt** (Case-Sensitivity geprüft: `VORNAME`, `TIERART`, etc.)
- [ ] **Ergebnis-Mail importiert** und Platzhalter ersetzt (Calendly, Shop, WhatsApp)
- [ ] **Partner-Sequenz importiert** (Mail 1-5) und Platzhalter ersetzt
- [ ] **Automation erstellt** mit richtigem Trigger ("Contact confirms email")
- [ ] **Automation aktiviert**

---

### Vollständiger Test-Durchlauf

**Test 1: DOI-Prozess**

1. [ ] **Öffne den Futtercheck** in einem Inkognito-Fenster:
   ```
   https://cedricnitsch.de/futtercheck.html
   ```

2. [ ] **Fülle das Formular aus** mit ECHTEN Test-Daten:
   - Vorname: `Test`
   - E-Mail: **Deine eigene E-Mail** (die du kontrollieren kannst)
   - Telefon: `0123456789`
   - Wähle Kategorien aus (z.B. Hund, Mittel, Erwachsen, etc.)

3. [ ] **Klicke auf "Futtercheck starten"**

4. [ ] **Prüfe dein Postfach** (auch Spam!):
   - [ ] DOI-Mail erhalten? (Betreff: "Test, bitte bestätige deine E-Mail-Adresse")
   - [ ] Design korrekt?
   - [ ] Name korrekt personalisiert?

5. [ ] **Klicke auf den Bestätigungslink** in der Mail

6. [ ] **Wirst du weitergeleitet** zu `bestaetigung-danke.html`?
   - [ ] Seite lädt korrekt?
   - [ ] Überschrift: "✅ E-Mail erfolgreich bestätigt!"

---

**Test 2: Brevo-Kontakt prüfen**

7. [ ] **Gehe in Brevo zu**: `Contacts → All contacts`

8. [ ] **Suche nach deiner Test-E-Mail**

9. [ ] **Öffne den Kontakt** und prüfe:
   - [ ] Status: "✅ Confirmed" (nicht "Unconfirmed")
   - [ ] Attribut `VORNAME`: `Test`
   - [ ] Attribut `TIERART`: (deine Auswahl, z.B. `Hund`)
   - [ ] Alle anderen Attribute gefüllt?

---

**Test 3: Ergebnis-Mail**

10. [ ] **Warte ca. 2 Minuten** (die Automation hat 1 Minute Delay)

11. [ ] **Prüfe dein Postfach**:
    - [ ] Ergebnis-Mail erhalten? (Betreff: "Test, dein persönliches Futtercheck-Ergebnis ist da! 🐾")
    - [ ] Richtige Personalisierung? (`Hallo Test,`)
    - [ ] **Conditional Content korrekt?**
      - Zeigt die Mail nur Inhalte für DEINE gewählte Tierart? (z.B. nur Hund-Tipps)
      - Gesundheitstipps passend zu deiner Auswahl?
    - [ ] Alle Links funktionieren? (Calendly, Shop, WhatsApp)

---

**Test 4: Partner-Sequenz (Verkürzt für Test)**

12. [ ] **Option A: Warte 1 Tag** und prüfe, ob Mail 1 der Partner-Sequenz ankommt

13. [ ] **Option B: Beschleunigter Test** (empfohlen):
    - Gehe in Brevo zu deiner Automation
    - Ändere temporär die "Wait"-Zeiten:
      - 1 Tag → 2 Minuten
      - 3 Tage → 5 Minuten
    - Speichern & Test nochmal durchführen
    - ⏱️ Du kannst so die komplette Sequenz in ~30 Minuten testen
    - **WICHTIG**: Zeiten danach wieder zurücksetzen!

14. [ ] **Prüfe jede Mail der Sequenz**:
    - [ ] Mail 1 erhalten?
    - [ ] Mail 2 erhalten?
    - [ ] Mail 3 erhalten?
    - [ ] Mail 4 erhalten? (mit Tracking-Buttons?)
    - [ ] Mail 5 erhalten?

---

**Test 5: Tracking-Links in Mail 4**

15. [ ] **Öffne Mail 4** ("Eine Frage...")

16. [ ] **Klicke auf "Ja, erzähl mir mehr!"**

17. [ ] **Wirst du weitergeleitet?** (zu `interesse-danke.html` oder deiner konfigurierten URL)

18. [ ] **Gehe in Brevo** zu deinem Kontakt

19. [ ] **Prüfe Attribut** `PARTNER_INTERESSE`:
    - [ ] Steht dort jetzt `Hoch`?

---

### Fehlerbehandlung während des Tests

**Problem: DOI-Mail kommt nicht an**
- → Prüfe Spam-Ordner
- → Prüfe Template-ID im Code
- → Prüfe Brevo-Dashboard: `Transactional → Logs` für Fehler

**Problem: Ergebnis-Mail kommt sofort (ohne DOI-Bestätigung)**
- → Automation-Trigger falsch! Muss "Contact confirms email" sein, nicht "Contact added"

**Problem: Attribute leer in Brevo**
- → Namen falsch geschrieben (Groß-/Kleinschreibung!)
- → Im Code nachschauen: werden Attribute korrekt übergeben?

**Problem: Conditional Content zeigt alles an**
- → Brevo-Syntax prüfen: `{% if contact.TIERART == "Hund" %}` (mit Anführungszeichen!)

---

## 4. HÄUFIGE FEHLER & TROUBLESHOOTING

### ❌ Fehler 1: DOI-Mail wird nicht verschickt

**Symptom**: Nutzer füllt Futtercheck aus, erhält aber keine DOI-Mail.

**Mögliche Ursachen**:

1. **Template-ID vergessen oder falsch**
   - **Lösung**: Öffne `futtercheck.html`, Zeile ~227
   - Prüfe: Steht dort `templateId: 123` (mit echter Zahl)?
   - Prüfe: Ist die Zahl identisch mit deiner Template-ID in Brevo?

2. **Brevo API-Key fehlt oder ungültig**
   - **Lösung**: Prüfe im Code, ob `BREVO_API_KEY` korrekt eingesetzt ist
   - Test: Gehe zu Brevo → `Settings → API Keys` und erstelle einen neuen Key falls nötig

3. **E-Mail-Adresse ungültig**
   - **Lösung**: Prüfe, ob das Formular valide E-Mails akzeptiert
   - Tipp: Im Code ist bereits eine Validierung eingebaut (`type="email"`)

4. **DOI-Template ist "draft"**
   - **Lösung**: In Brevo muss das Template "active" sein
   - Gehe zu `Transactional → Email Templates` und prüfe den Status

**Debugging**:
- Öffne die Browser-Konsole (`F12`) beim Absenden des Formulars
- Prüfe auf Fehler wie "Template not found" oder "Invalid API key"
- Gehe zu Brevo → `Transactional → Logs` und schaue nach fehlgeschlagenen E-Mails

---

### ❌ Fehler 2: Ergebnis-Mail wird sofort verschickt (VOR DOI-Bestätigung)

**Symptom**: Nutzer erhält sofort die Ergebnis-Mail, ohne auf den Bestätigungslink geklickt zu haben.

**Ursache**: Falscher Trigger in der Automation!

**Lösung**:
1. Gehe zu `Automation → Workflows`
2. Öffne deine Automation
3. Klicke auf den **Trigger** (erster Block)
4. **Prüfe**: Steht dort "Contact confirms email (DOI)"?
5. **Falls NICHT**: Lösche den Trigger und füge den richtigen hinzu:
   - Lösche den aktuellen Trigger
   - Klicke auf "Add trigger"
   - Wähle "Contact confirms email (DOI)"
   - Speichern & Automation neu aktivieren

---

### ❌ Fehler 3: Attribute in Brevo bleiben leer

**Symptom**: Kontakt wird angelegt, aber Felder wie `TIERART`, `GROESSE` etc. sind leer.

**Mögliche Ursachen**:

1. **Attribute falsch geschrieben (Case-Sensitivity!)**
   - **Lösung**: Prüfe in Brevo → `Contacts → Settings → Contact Attributes`
   - Die Namen müssen EXAKT so sein:
     ```
     VORNAME (nicht vorname oder Vorname)
     TIERART (nicht tierart oder Tier_Art)
     ```

2. **Attribute wurden nicht angelegt**
   - **Lösung**: Gehe zu Schritt 4 und lege alle Attribute an

3. **Formular sendet Daten falsch**
   - **Lösung**: Öffne `futtercheck.html` und prüfe Zeile ~200
   - Stelle sicher, dass die Attribute im `attributes`-Objekt korrekt benannt sind:
     ```javascript
     attributes: {
       VORNAME: formData.vorname,
       TIERART: formData.tierart,
       GROESSE: formData.groesse,
       // ...
     }
     ```

**Debugging**:
- Öffne Browser-Konsole (`F12`)
- Sende Formular ab
- Prüfe das `fetch`-Request: Werden die Attribute mitgeschickt?
- Gehe zu Brevo → `Contacts → Logs` und prüfe, was ankommt

---

### ❌ Fehler 4: Weiterleitung nach DOI-Klick führt zu 404-Fehler

**Symptom**: Nutzer klickt auf DOI-Link und landet auf einer Fehlerseite ("Seite nicht gefunden").

**Ursache**: Die Datei `bestaetigung-danke.html` existiert nicht auf dem Server oder die URL ist falsch.

**Lösung**:
1. **Prüfe**: Ist `bestaetigung-danke.html` auf deinem Server hochgeladen?
2. **Teste die URL direkt** im Browser:
   ```
   https://cedricnitsch.de/bestaetigung-danke.html
   ```
3. **Falls 404**: Lade die Datei hoch (via FTP/SFTP)
4. **Falls URL falsch**: Öffne `futtercheck.html` und ändere `redirectionUrl` (Zeile ~230)

---

### ❌ Fehler 5: DOI-Mail landet im Spam

**Symptom**: DOI-Mail wird verschickt, aber landet immer im Spam-Ordner.

**Mögliche Ursachen**:

1. **Absender-Domain nicht verifiziert**
   - **Lösung**: Gehe zu Brevo → `Settings → Senders & IPs → Domains`
   - Füge deine Domain hinzu und verifiziere sie (SPF, DKIM)
   - Anleitung: [Brevo Domain Authentication](https://help.brevo.com/hc/en-us/articles/360008712180)

2. **E-Mail-Inhalt triggert Spam-Filter**
   - **Lösung**: Vermeide Spam-Wörter wie "Kostenlos", "Jetzt kaufen", zu viele Großbuchstaben
   - Das DOI-Template ist bereits optimiert, aber prüfe ob du Änderungen gemacht hast

3. **IP-Reputation niedrig** (bei neuem Brevo-Account)
   - **Lösung**: Braucht Zeit! Versende anfangs nur kleine Mengen
   - Brevo "wärmt" automatisch deine IP auf (Warm-up)

**Workaround**:
- Bitte Test-Empfänger, die Mail als "Kein Spam" zu markieren
- Nach einigen erfolgreichen Zustellungen verbessert sich die Reputation

---

### ❌ Fehler 6: Conditional Content zeigt ALLE Inhalte (nicht nur relevante)

**Symptom**: In der Ergebnis-Mail werden Hund-, Katzen- UND Pferde-Tipps angezeigt.

**Ursache**: Brevo-Syntax für Conditional Content ist falsch.

**Lösung**:
1. Öffne die Ergebnis-Mail in Brevo
2. Wechsle zum HTML-Modus
3. Prüfe die Conditional-Syntax:
   ```handlebars
   {% if contact.TIERART == "Hund" %}
     <!-- Hund-spezifischer Inhalt -->
   {% elsif contact.TIERART == "Katze" %}
     <!-- Katzen-spezifischer Inhalt -->
   {% elsif contact.TIERART == "Pferd" %}
     <!-- Pferd-spezifischer Inhalt -->
   {% endif %}
   ```

**Wichtige Punkte**:
- **Anführungszeichen** um Werte: `"Hund"` (nicht `Hund`)
- **Doppeltes Gleichheitszeichen**: `==` (nicht `=`)
- **elsif** (nicht `else if`)
- **Closing-Tag**: `{% endif %}`

---

### ❌ Fehler 7: Partner-Sequenz startet nicht nach 1 Tag

**Symptom**: Ergebnis-Mail kommt an, aber Mail 1 der Partner-Sequenz kommt nie.

**Mögliche Ursachen**:

1. **Automation ist nicht aktiviert**
   - **Lösung**: Gehe zu `Automation → Workflows`
   - Prüfe: Steht dort "Active" oder "Paused"?
   - Falls "Paused": Klicke auf "Activate"

2. **Kontakt wurde aus der Automation genommen**
   - **Lösung**: Öffne den Kontakt in Brevo
   - Gehe zu "Workflow History"
   - Prüfe: Ist der Kontakt in der Automation oder hat er sie verlassen?

3. **Wait-Zeit falsch konfiguriert**
   - **Lösung**: Öffne die Automation
   - Prüfe die "Wait"-Aktion nach der Ergebnis-Mail
   - Sollte sein: `1 day` (nicht 1 week, 1 hour, etc.)

**Debugging**:
- Gehe zu `Automation → Workflows → [Deine Automation]`
- Klicke auf "Analytics"
- Prüfe: Wie viele Kontakte sind in welchem Schritt?
- Wenn Kontakte bei der "Wait"-Aktion hängen → Wait für Test verkürzen (z.B. 5 Minuten)

---

## 5. GO-LIVE CHECKLISTE

**Letzte Checks, bevor du den Futtercheck scharf schaltest:**

### Technische Checks

- [ ] **Template-ID ist korrekt** im Code eingesetzt (futtercheck.html)
- [ ] **Alle Dateien hochgeladen**:
  - [ ] futtercheck.html
  - [ ] bestaetigung-danke.html
- [ ] **Alle URLs funktionieren**:
  - [ ] https://cedricnitsch.de/futtercheck.html
  - [ ] https://cedricnitsch.de/bestaetigung-danke.html

### Brevo-Checks

- [ ] **Alle 9 Attribute angelegt** (Case-Sensitivity geprüft)
- [ ] **DOI-Template gespeichert** und "Active"
- [ ] **Ergebnis-Mail erstellt** und alle Platzhalter ersetzt:
  - [ ] Calendly-Link
  - [ ] Shop-Link
  - [ ] WhatsApp-Nummer
- [ ] **Partner-Sequenz erstellt** (Mail 1-5) und alle Platzhalter ersetzt
- [ ] **Automation erstellt** mit korrektem Trigger ("Contact confirms email")
- [ ] **Automation aktiviert** (Status: "Active")

### Testing

- [ ] **Vollständiger Test-Durchlauf durchgeführt** (siehe Abschnitt 3)
- [ ] **DOI-Mail erhalten und bestätigt**
- [ ] **Ergebnis-Mail erhalten** (korrekte Personalisierung & Conditional Content)
- [ ] **Partner-Sequenz getestet** (mindestens Mail 1-2)
- [ ] **Tracking-Links in Mail 4 getestet** (PARTNER_INTERESSE wird gesetzt)

### Backup & Dokumentation

- [ ] **Backup der alten Konfiguration gemacht** (falls du etwas ersetzt hast)
- [ ] **Screenshot der Automation-Struktur** gemacht (für deine Unterlagen)
- [ ] **Template-ID notiert** (z.B. in einer Textdatei)
- [ ] **Diese Anleitung durchgelesen** 😊

### Go-Live!

- [ ] **Alles grün?** → Dann schalte den Futtercheck live!
- [ ] **Teile den Link**:
  - Social Media
  - E-Mail-Signatur
  - Website
  - Wo auch immer deine Zielgruppe ist

### Post-Launch Monitoring (erste 24 Stunden)

- [ ] **Prüfe Brevo-Dashboard** regelmäßig:
  - Wie viele Kontakte füllen den Futtercheck aus?
  - Wie hoch ist die DOI-Bestätigungsrate? (sollte >50% sein)
  - Werden die Ergebnis-Mails zugestellt? (Bounce-Rate?)
  - Landen Mails im Spam? (teste mit verschiedenen E-Mail-Anbietern)

- [ ] **Prüfe Automation-Analytics**:
  - Gehe zu `Automation → Workflows → Analytics`
  - Wie viele Kontakte sind in welchem Schritt?
  - Verlassen Kontakte die Automation frühzeitig? (dann: warum?)

- [ ] **Sammle erstes Feedback**:
  - Frag Freunde/Familie, ob sie den Futtercheck testen
  - Gibt es Verständnisprobleme?
  - Sind die Mails hilfreich?

---

## 6. SUPPORT & WEITERE RESSOURCEN

### 📚 Weitere Dokumentationen in diesem Repository

Diese Anleitung ist Teil einer umfassenden Dokumentation. Für tiefere Einblicke:

- **`DOI_DOKUMENTATION.md`**
  - Technische Hintergründe zum DOI-System
  - API-Calls im Detail
  - Code-Erklärungen

- **`DOI_UEBERSICHT.md`**
  - Kompakte Zusammenfassung des DOI-Prozesses
  - Schnellreferenz für erfahrene Nutzer

- **`BREVO_ERGEBNIS_MAIL_SETUP.md`**
  - Detaillierte Anleitung zur Conditional Content Logik
  - Optimierung der Ergebnis-Mail
  - Personalisierungs-Tipps

- **`BREVO_PARTNER_SEQUENZ_SETUP.md`**
  - Aufbau der 5er-Sequenz im Detail
  - Timing-Strategien
  - A/B-Testing-Ideen

- **`FUTTERCHECK_UMBAU.md`**
  - Umbau vom alten System zum neuen DOI-System
  - Was hat sich geändert?
  - Migration bestehender Kontakte

---

### 🔍 Wo finde ich was in Brevo?

**Template-IDs finden**:
1. Gehe zu `Transactional → Email Templates`
2. Klicke auf dein Template
3. Schau in die Browser-URL: `https://app.brevo.com/template/edit/123`
4. Die Zahl am Ende (`123`) ist die Template-ID

**API-Key erstellen/finden**:
1. Gehe zu `Settings → API Keys`
2. Klicke auf "Create a new API Key"
3. Name: "Futtercheck DOI"
4. Kopiere den Key (WICHTIG: nur einmal sichtbar!)

**Kontakt-Listen verwalten**:
1. Gehe zu `Contacts → Lists`
2. Erstelle eine neue Liste: "Futtercheck DOI-bestätigt"
3. Optional: Füge diese Liste als Filter in der Automation hinzu

**E-Mail-Logs prüfen**:
1. Gehe zu `Transactional → Logs`
2. Filtere nach E-Mail-Adresse oder Zeitraum
3. Sieh dir Details zu jeder E-Mail an (zugestellt, geöffnet, geklickt, gebounced)

**Automation-Analytics**:
1. Gehe zu `Automation → Workflows`
2. Klicke auf deine Automation
3. Tab "Analytics"
4. Sieh dir Durchlaufzeiten, Abbruchraten, Conversion an

---

### 🛟 Brevo Support kontaktieren

**Wann solltest du Brevo kontaktieren?**
- API-Fehler, die du nicht lösen kannst (z.B. "500 Internal Server Error")
- DOI-Token werden nicht generiert
- E-Mails werden trotz korrekter Konfiguration nicht verschickt
- Attribute verhalten sich unerwartet

**Wie kontaktierst du Brevo?**
1. Gehe zu [help.brevo.com](https://help.brevo.com)
2. Klicke rechts unten auf den Chat-Button
3. Oder: Schreibe eine E-Mail an `support@brevo.com`

**Was solltest du bereithalten?**
- Deine Brevo Account-E-Mail
- Template-ID(s)
- Fehlermeldungen (Screenshots)
- Zeitpunkt des Fehlers (Datum/Uhrzeit)
- Betroffene E-Mail-Adressen (für Tests)

---

### 📖 Weiterführende Links

**Brevo Dokumentation**:
- [DOI Setup Guide](https://help.brevo.com/hc/en-us/articles/360000946299)
- [Transactional Email API](https://developers.brevo.com/reference/sendtransacemail)
- [Conditional Content Syntax](https://help.brevo.com/hc/en-us/articles/360000267730)
- [Automation Workflows](https://help.brevo.com/hc/en-us/articles/360000267190)

**DSGVO & Rechtliches**:
- [DSGVO-konforme E-Mail-Marketing](https://dsgvo-gesetz.de/art-6-dsgvo/)
- [Double Opt-In Pflicht in Deutschland](https://www.e-recht24.de/artikel/datenschutz/11227-newsletter-abmahnungen-double-opt-in-pflicht.html)

**Reico Vital**:
- [Reico Partner werden](https://www.reico-vital.com/partner-werden)
- [Reico Produktkatalog](https://www.reico-vital.com/produkte)

---

### 💡 Optimierungs-Ideen (für später)

Sobald dein System läuft, kannst du optimieren:

**A/B-Testing**:
- Teste verschiedene Betreff-Zeilen in der DOI-Mail
- Teste verschiedene Button-Texte ("Jetzt bestätigen" vs. "E-Mail verifizieren")
- Teste verschiedene Wartezeiten in der Partner-Sequenz

**Segmentierung**:
- Erstelle separate Automationen für Hunde-, Katzen- und Pferdebesitzer
- Passe die Sequenz basierend auf `GESUNDHEITSZUSTAND` an
- Erstelle eine VIP-Sequenz für Kontakte mit `PARTNER_INTERESSE = Hoch`

**Lead-Scoring**:
- Vergib Punkte für jede Interaktion (E-Mail geöffnet, Link geklickt)
- Verschiebe hochengagierte Kontakte in eine separate Liste
- Automatische Benachrichtigung wenn ein Lead "heiß" ist

**Retargeting**:
- Erstelle eine Automation für Kontakte, die DOI NICHT bestätigt haben
- Sende nach 3 Tagen eine Erinnerungs-Mail
- "Hast du unsere letzte Mail übersehen?"

**Analytics & Reporting**:
- Erstelle ein Dashboard mit Google Data Studio
- Verknüpfe Brevo-Daten mit Google Analytics
- Tracke Conversion-Rate vom Futtercheck bis zum Kauf

---

## 🎉 GESCHAFFT!

Wenn du diese Anleitung befolgt hast, hast du jetzt ein **vollständig automatisiertes, DSGVO-konformes DOI-System** mit:

✅ **Rechtssicherer Double Opt-In Prozess**  
✅ **Personalisierte Ergebnis-Mails** mit Conditional Content  
✅ **Automatische 5er-Partner-Sequenz** über 2 Wochen  
✅ **Tracking & Analytics** für Optimierung  
✅ **Professionelle Nutzer-Experience** vom Futtercheck bis zum Partner-Angebot

**Du bist jetzt startklar!** 🚀

---

**Fragen? Probleme? Verbesserungsvorschläge?**

Diese Dokumentation ist ein lebendiges Dokument. Wenn du Fehler findest oder etwas unklar ist, aktualisiere diese Datei für zukünftige Referenz.

**Viel Erfolg mit deinem Reico-Recruiting-Funnel!** 🐾

---

*Erstellt für Cedric Nitsch | Stand: Juni 2026 | Version 1.0*
