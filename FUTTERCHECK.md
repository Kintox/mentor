# 🐾 Interaktiver Futtercheck – Dokumentation

Der Futtercheck ist ein **interaktives 12-Fragen-Quiz**, das als Lead-Magnet für den Recruiting-Funnel dient. Nutzer beantworten Fragen über ihr Haustier und erhalten einen personalisierten Ernährungs-Score.

---

## 📋 Funktionsweise

### Ablauf:
1. **Startseite** – CTA „Jetzt Futtercheck starten"
2. **12 Fragen** – Tierart, Name, Alter, Gewicht, Futter, Symptome, Allergien etc.
3. **E-Mail & DSGVO** – Nutzer gibt E-Mail + optional WhatsApp ein
4. **Auswertung** – Score (0–100) mit persönlicher Empfehlung
5. **CTA** – WhatsApp-Beratung oder Beratungsgespräch buchen

### Score-System:
| Score | Profil | Farbe | Bedeutung |
|-------|--------|-------|-----------|
| 75–100 | 🟢 Grün | Grün | Gute Basis – Feintuning möglich |
| 40–74 | 🟡 Gelb | Gelb | Verbesserungspotenzial |
| 0–39 | 🔴 Rot | Rot | Dringender Handlungsbedarf |

### Fragen:
1. Tierart (Hund/Katze)
2. Tiername
3. Alter
4. Gewicht/Größe
5. Aktivitätslevel
6. Aktuelles Futter
7. Symptome (Mehrfachauswahl)
8. Allergien
9. Leckerlis-Gewohnheiten
10. Zufriedenheit mit aktueller Ernährung
11. Vorname des Nutzers
12. E-Mail + WhatsApp (optional) + DSGVO

---

## 🔧 Konfiguration

### Brevo API-Key
Der API-Key ist bereits eingetragen in `futtercheck.html`:
```javascript
const BREVO_API_KEY = 'xkeysib-...';
```

### ⚠️ Brevo Listen-ID anpassen
Die Listen-ID bestimmt, in welche Brevo-Liste die Kontakte eingetragen werden:
```javascript
const BREVO_LIST_ID = 2; // ← Hier deine Listen-ID eintragen!
```

**So findest du die richtige Listen-ID:**
1. Gehe zu [app.brevo.com](https://app.brevo.com)
2. Navigiere zu **Kontakte → Listen**
3. Erstelle eine neue Liste (z.B. „Futtercheck Leads")
4. Die ID findest du in der URL oder in den Listen-Details
5. Trage die ID in `futtercheck.html` ein

### WhatsApp-Nummer
```javascript
const WHATSAPP_NUMBER = '4915678516818';
```

### Meta Pixel
Die Pixel-ID `1267711028899835` ist bereits konfiguriert. Folgende Events werden getrackt:
- **PageView** – Beim Laden der Seite
- **ViewContent** – Beim Start des Quiz
- **Lead** – Nach Abschluss des Quiz (mit Score als Value)

---

## 📊 Brevo-Kontaktdaten

Folgende Attribute werden bei jedem abgeschlossenen Futtercheck an Brevo gesendet:

| Attribut | Beschreibung | Beispiel |
|----------|-------------|----------|
| `email` | E-Mail-Adresse | max@example.de |
| `VORNAME` | Vorname des Nutzers | Max |
| `TIERART` | Hund oder Katze | Hund |
| `TIERNAME` | Name des Tieres | Bello |
| `FUTTERCHECK_SCORE` | Score (0–100) | 62 |
| `FUTTERCHECK_ERGEBNIS` | optimal / verbesserbar / kritisch | verbesserbar |
| `FUTTERCHECK_FUTTER` | Aktuelles Futter | trockenfutter_standard |
| `SMS` | WhatsApp-Nummer (optional) | 015678516818 |

> **Wichtig:** Stelle sicher, dass diese Attribute in Brevo als Kontakt-Attribute angelegt sind. Brevo erstellt sie bei der ersten Verwendung automatisch.

---

## 🧪 Testen

1. Öffne `futtercheck.html` im Browser
2. Durchlaufe alle 12 Fragen
3. Gib eine Test-E-Mail ein
4. Prüfe in Brevo unter **Kontakte**, ob der Kontakt angelegt wurde
5. Prüfe die Kontakt-Attribute (Score, Tierart, etc.)
6. Teste auch auf dem Handy (responsive Design)

### Browser-Konsole
Öffne die Browser-Konsole (F12 → Console) während des Tests. Du siehst:
- `✅ Kontakt erfolgreich in Brevo erstellt/aktualisiert` bei Erfolg
- `⚠️ Brevo-Antwort: ...` bei Fehlern (z.B. ungültige Listen-ID)

---

## 🔗 Integration auf der Hauptseite

Der Futtercheck ist in `index.html` verlinkt:
- **Navigation (Desktop + Mobile):** „Futtercheck" Link → `futtercheck.html`
- **Hero-CTA:** „Kostenlosen Futtercheck starten" → `futtercheck.html`
- **CTA-Button in der Nav:** 🐾 Kostenloser Futtercheck → `futtercheck.html`

---

## 📝 Lizenz

Private Webseite. Alle Rechte vorbehalten.
