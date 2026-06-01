# 🐾 Cedric Nitsch – Reico Fachberater & Business Mentor

Recruiting-Funnel Landingpage für Reico & Partner Vertriebspartnerschaft. Optimiert für Lead-Generierung, Conversion und automatisiertes Recruiting.

---

## 📋 Seitenstruktur

| Datei | Beschreibung |
|---|---|
| `index.html` | Hauptseite mit Lead-Magnet, Brevo Opt-in-Formular, Testimonials, FAQ, CTAs |
| `futtercheck.html` | **Interaktiver Futtercheck** – 12-Fragen-Quiz mit Score & Brevo-Anbindung |
| `danke.html` | Danke-Seite nach Opt-in mit Terminbuchung & nächsten Schritten |
| `impressum.html` | Impressum (rechtliche Pflichtseite) |
| `datenschutz.html` | Datenschutzerklärung (DSGVO) |
| `images/` | Bilder (Hund, Katze, Pferd, Mensch, Profilbild) |
| `FUTTERCHECK.md` | Dokumentation des Futterchecks (Funktion, Konfiguration) |
| `style.css` | Legacy-Styles (nicht aktiv genutzt, Tailwind via CDN) |

---

## 🔧 Konfiguration

### 1. Formular-Backend (Brevo)

Das Opt-in-Formular auf `index.html` und der interaktive Futtercheck nutzen **Brevo** (ehemals Sendinblue).

- **index.html:** Brevo-Formular mit Double-Opt-in (Custom AJAX Handler, kein Brevo main.js)
- **futtercheck.html:** Brevo Contacts API (`/v3/contacts`) für Lead-Erstellung nach Quiz-Abschluss

**Brevo Listen-ID anpassen:**
In `futtercheck.html` findest du:
```javascript
const BREVO_LIST_ID = 2; // ⚠️ An deine Brevo-Liste anpassen!
```
Ändere die `2` auf die ID deiner Futtercheck-Liste in Brevo (unter Kontakte → Listen).

> Siehe auch: [FUTTERCHECK.md](FUTTERCHECK.md) für die vollständige Dokumentation des Futterchecks.

### 2. Meta-Pixel (Facebook/Instagram)

Das Meta Pixel ist bereits eingebaut und konfiguriert.

- **Pixel-ID:** `1267711028899835`
- **PageView:** Wird auf allen Seiten automatisch gefeuert
- **Lead-Event:** Wird auf der Danke-Seite (`danke.html`) automatisch gefeuert
- **Zusätzlich:** Lead-Event wird auch beim Formular-Submit auf `index.html` gefeuert

**Pixel-ID ändern:**
Suche in `index.html` und `danke.html` nach:
```javascript
fbq('init', '1267711028899835');
```
Ersetze `1267711028899835` mit deiner eigenen Pixel-ID.

### 3. Terminbuchung (Calendly / TidyCal)

Die Danke-Seite enthält vorbereitete Embed-Codes für Calendly und TidyCal.

**Calendly einrichten:**
1. Erstelle ein kostenloses Konto auf [https://calendly.com](https://calendly.com)
2. Erstelle einen Event-Typ "Info-Gespräch" (15 Min.)
3. In `danke.html`, kommentiere den Calendly-Embed ein und ersetze die URL:
   ```html
   <div class="calendly-inline-widget" 
        data-url="https://calendly.com/DEIN_USERNAME/info-gespraech" 
        style="min-width:320px;height:700px;">
   </div>
   <script src="https://assets.calendly.com/assets/external/widget.js" async></script>
   ```
4. Entferne den Platzhalter-Bereich

**TidyCal Alternative:**
1. Kaufe TidyCal auf [AppSumo](https://appsumo.com) (einmalig 29 €)
2. Kommentiere den TidyCal-Embed ein und ersetze den Pfad

---

## 🚀 Deployment (GitHub Pages)

1. Push die Änderungen auf den `main`-Branch
2. Gehe zu Repository → Settings → Pages
3. Source: `main` Branch, Ordner: `/ (root)`
4. Custom Domain: `cedricnitsch.de` (CNAME ist bereits konfiguriert)

---

## 📊 Änderungsübersicht (v2.0 – Recruiting-Funnel-Optimierung)

### Neu hinzugefügt:
- ✅ **Lead-Magnet Opt-in-Formular** (Futtercheck) mit Brevo-Integration
- ✅ **Interaktiver Futtercheck** (`futtercheck.html`) – 12-Fragen-Quiz mit Scoring & Brevo-API
- ✅ **Danke-Seite** (`danke.html`) mit Terminbuchungs-Integration
- ✅ **Testimonials-Sektion** mit 3 Beispiel-Testimonials (Avatar 1, 2 & 3)
- ✅ **Vertrauensleiste** mit Zahlen (30+ Jahre, 8.500+ Partner, 100 Mio. €)
- ✅ **Lead-Magnet CTA** an mehreren Stellen (Hero, nach FAQ, Footer)
- ✅ **Meta-Pixel Lead-Event** auf Formular-Submit und Danke-Seite

### Optimiert:
- ✅ **Hero-Bereich:** Tier-Fokus + emotionalere Headline ("Dein Hund verdient das Beste")
- ✅ **Haupt-CTA:** "Kostenlosen Futtercheck sichern" statt nur WhatsApp
- ✅ **Problem-Sektion:** Erweitert um Tier-Probleme (Avatar 1 & 3)
- ✅ **Lösungs-Sektion:** 6 klare Benefits statt 4
- ✅ **FAQ-Sektion:** Erweitert um Futtercheck-Frage und Verdienstfrage
- ✅ **Navigation:** Futtercheck als primärer CTA in der Navigation
- ✅ **SEO:** Vollständige Meta-Tags (OG, Description, Keywords, Alt-Texte)
- ✅ **Performance:** Lazy Loading für Bilder, lokale Bildpfade statt externe URLs
- ✅ **Mobile:** Verbesserte mobile Navigation und CTAs
- ✅ **Schritte-Sektion:** Von 5 auf 3 vereinfacht (Futtercheck → Gespräch → Start)

### Beibehalten:
- ✅ Design-System (Farben, Fonts, Tailwind-Konfiguration)
- ✅ WhatsApp als sekundärer Kontaktkanal
- ✅ Persönliche Story von Cedric
- ✅ Produktgalerie
- ✅ DSGVO-konforme Struktur (Impressum, Datenschutz)

---

## 🛠 Tech-Stack

- **HTML5** – Semantisches Markup
- **Tailwind CSS** (CDN) – Utility-first CSS Framework
- **Lucide Icons** – Moderne Icon-Library
- **Google Fonts** (Poppins) – Typografie
- **Brevo** (Sendinblue) – Formulare, Kontakte, E-Mail-Automation (DSGVO-konform)
- **Meta Pixel** – Facebook/Instagram Tracking
- **GitHub Pages** – Hosting

---

## 📝 Lizenz

Private Webseite. Alle Rechte vorbehalten.
