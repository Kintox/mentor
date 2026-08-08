# DOI-Bestätigungs-Mail – Dokumentation

## 📧 Betreff & Preheader

### **Betreff-Vorschläge:**
1. **„{{ contact.VORNAME }}, bitte bestätige deine E-Mail-Adresse"** *(persönlich, direkt)*
2. **„Nur noch 1 Klick bis zu deinem Futtercheck-Ergebnis"** *(nutzenorientiert)*
3. **„Fast geschafft! Bestätige jetzt deine E-Mail"** *(motivierend, kurz)*
4. **„{{ contact.VORNAME }}, dein Futtercheck für {{ contact.TIERNAME }} wartet"** *(sehr persönlich)*

**Empfehlung:** Option 3 oder 4 – kurz, klar, mit Personalisierung.

---

### **Preheader:**
```
Nur noch ein Klick, dann erhältst du dein persönliches Futtercheck-Ergebnis!
```
*(Bereits in der HTML-Vorlage integriert)*

---

## 🔧 Brevo DOI-Platzhalter: `{{ doiURL }}`

### **Was ist der `{{ doiURL }}` Platzhalter?**
- Brevo generiert automatisch einen **eindeutigen Bestätigungs-Link** für jeden Kontakt
- Dieser Link ist **nur einmal nutzbar** und hat eine begrenzte Gültigkeit (Standard: 30 Tage)
- Der Link führt zu einer Brevo-Seite, die die E-Mail-Adresse verifiziert

### **Wo wird der Platzhalter eingesetzt?**
Im HTML-Template an **zwei Stellen**:

1. **Im CTA-Button (href):**
   ```html
   <a href="{{ doiURL }}" target="_blank">
     Jetzt E-Mail-Adresse bestätigen →
   </a>
   ```

2. **Als Fallback-Link (Text):**
   ```html
   {{ doiURL }}
   ```
   *(Falls der Button nicht funktioniert, kann der Nutzer den Link kopieren)*

---

## ⚙️ Brevo DOI-Konfiguration

### **1. DOI-Template in Brevo einrichten**

**Schritt-für-Schritt:**
1. **Brevo > Campaigns > Email Templates > Create Template**
2. Upload der `doi_bestaetigung.html` oder manueller Import
3. **Template-Name:** `DOI-Bestätigung Futtercheck`
4. **Kategorie:** Transactional

---

### **2. DOI in der Kontaktliste aktivieren**

**Brevo > Contacts > Listen auswählen > Einstellungen**

Option: **„Double Opt-In aktivieren"**
- DOI-Template auswählen: `DOI-Bestätigung Futtercheck`
- **Weiterleitungs-URL nach Bestätigung festlegen** (siehe unten)

---

### **3. Weiterleitungs-URL nach Bestätigung**

Nach dem Klick auf den `{{ doiURL }}` wird der Kontakt:
1. **Automatisch bestätigt** (Status: `subscribed`)
2. **Weitergeleitet** zur konfigurierten URL

**Mögliche Weiterleitungs-URLs:**

| Option | URL | Zweck |
|--------|-----|-------|
| **Thank-You-Page** | `https://cedricnitsch.de/bestaetigung-danke` | Bedanken + erste Inhalte zeigen |
| **Direkt zum Futtercheck** | `https://cedricnitsch.de/futtercheck-ergebnis` | Sofortiger Zugriff auf Ergebnis |
| **Landing Page mit Bonus** | `https://cedricnitsch.de/bonus-gratis-guide` | Lead Magnet als Belohnung |

**Empfehlung:**  
→ **Thank-You-Page mit CTA zum Futtercheck** (Beste UX + Trust-Building)

---

### **4. Automation-Trigger nach DOI-Bestätigung**

**Brevo > Automation > Erstellen**

**Trigger:** `Kontakt bestätigt E-Mail (DOI)`  
**Filter:** Liste = `Futtercheck Leads`

**Aktionen:**
1. **Warte 2 Minuten** *(Brevo-Verarbeitung abwarten)*
2. **Sende Mail 1:** Futtercheck-Ergebnis mit Link
3. **Warte 2 Tage**
4. **Sende Mail 2:** Tipps zur Futterumstellung
5. **Warte 3 Tage**
6. **Sende Mail 3:** Erfolgsgeschichten + Calendly-CTA
7. Usw.

---

## 🛠️ Technische Integration im Futtercheck

### **Was passiert im Backend?**

**Vorher (nicht DSGVO-konform):**
```javascript
// Kontakt wird SOFORT aktiviert
await brevo.contacts.createContact({
  email: userEmail,
  attributes: { VORNAME, TIERNAME },
  listIds: [12345],
  updateEnabled: true
});
// ❌ Problem: Nutzer hat noch nicht zugestimmt!
```

**Nachher (DSGVO-konform mit DOI):**
```javascript
// Kontakt wird mit DOI-Status angelegt
await brevo.contacts.createContact({
  email: userEmail,
  attributes: { VORNAME, TIERNAME },
  listIds: [12345],
  updateEnabled: false, // ← Wichtig!
  emailBlacklisted: false
});
// ✅ DOI-Mail wird automatisch versendet
// ✅ Automation startet erst nach Bestätigung
```

**Wichtig:**
- `updateEnabled: false` → verhindert, dass bestehende Kontakte überschrieben werden
- Brevo versendet die DOI-Mail **automatisch**, wenn DOI für die Liste aktiviert ist
- **Kein manueller Versand der DOI-Mail nötig!**

---

## 📋 Checkliste: DOI-Setup abschließen

- [ ] `doi_bestaetigung.html` in Brevo als Template hochladen
- [ ] DOI in der Kontaktliste aktivieren
- [ ] DOI-Template zuweisen
- [ ] Weiterleitungs-URL konfigurieren (z.B. Thank-You-Page)
- [ ] Automation mit Trigger „Kontakt bestätigt E-Mail" erstellen
- [ ] Test durchführen: Neue E-Mail im Futtercheck eintragen
- [ ] Prüfen: DOI-Mail erhalten? Link funktioniert? Weiterleitung korrekt?
- [ ] Prüfen: Automation startet nach Bestätigung?

---

## 🔍 Troubleshooting

### **Problem: DOI-Mail wird nicht versendet**
**Lösung:**
- Prüfen: Ist DOI in der Liste aktiviert?
- Prüfen: Ist das DOI-Template korrekt zugewiesen?
- Prüfen: Ist `updateEnabled: false` im API-Call gesetzt?

### **Problem: Automation startet trotz DOI sofort**
**Lösung:**
- Trigger prüfen: Muss `Kontakt bestätigt E-Mail (DOI)` sein, NICHT `Kontakt hinzugefügt`
- Filter prüfen: Nur für die richtige Liste

### **Problem: Link im DOI-Mail funktioniert nicht**
**Lösung:**
- `{{ doiURL }}` darf NICHT in Anführungszeichen oder mit Leerzeichen stehen
- Muss exakt so im href stehen: `href="{{ doiURL }}"`

---

## 📞 Support

Bei Fragen zur Implementierung:
- **Brevo Support:** https://help.brevo.com/hc/de/articles/360000946299
- **DOI Best Practices:** https://help.brevo.com/hc/de/articles/360018508719

---

**Status:** ✅ Template erstellt, bereit für Brevo-Upload  
**Erstellt am:** 01.06.2026  
**Projekt:** Reico Recruiting-Funnel | Cedric Nitsch
