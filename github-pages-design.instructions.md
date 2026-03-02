---
applyTo: "docs/**/*.html"
---

# GitHub Pages – Design-Richtlinien & Aufbau

## Überblick

Diese Instruction definiert den visuellen Standard und die technische Architektur
für alle GitHub-Pages-Seiten im Projekt. Das Design basiert auf der FLUVIA.ONE
Corporate Identity und muss bei jeder Erstellung oder Änderung von Seiten im
`docs/`-Verzeichnis eingehalten werden.

> **Referenz-Datei:** `docs/index.html`
> **Live-URL:** Wird über GitHub Pages aus dem `docs/`-Ordner auf dem Branch `dev_stage` bereitgestellt.

## Farbpalette (PFLICHT)

Alle Farben sind als CSS Custom Properties definiert und MÜSSEN verwendet werden.
Keine Inline-Hex-Werte außerhalb dieser Palette.

```css
:root {
    --primary: #3774a5;        /* Hauptfarbe – Buttons, Links, Akzente */
    --primary-dark: #356d96;   /* Hover-Zustand, Gradient-Endpunkt */
    --primary-light: #6fa3ce;  /* Sekundäre Akzente, Icons */
    --primary-bg: #e7f1fa;     /* Heller Hintergrund für Icons, Badges */
    --accent: #7090A2;         /* Dekorative Elemente */
    --wave-bg: #BDCCD4;        /* Wellenformen, dekorative SVGs */
    --border: #d4dfe6;         /* Kartenränder, Trennlinien */
    --bg: #f4f8fb;             /* Seitenhintergrund */
    --text: #2c3e50;           /* Haupttext */
    --text-light: #5a7184;     /* Sekundärtext, Beschreibungen */
    --white: #ffffff;          /* Kartenhintergrund */
    --card-shadow: 0 2px 12px rgba(55, 116, 165, 0.08);
    --card-shadow-hover: 0 8px 28px rgba(55, 116, 165, 0.16);
}
```

### Kategorie-Farben für Badges

| Kategorie  | CSS-Klasse      | Hintergrund  | Textfarbe  |
|-----------|-----------------|-------------|-----------|
| Prozess   | `.cat-prozess`  | `--primary-bg` (#e7f1fa) | `--primary` (#3774a5) |
| Daten     | `.cat-daten`    | #e0f4f4     | #1a8a8a   |
| Finanzen  | `.cat-finanzen` | #ede7f3     | #6b4d8a   |

Neue Kategorien folgen dem gleichen Schema: heller Hintergrund + dunkle Textfarbe.

## Typografie

- **Schriftart:** `Inter` via Google Fonts (Fallback: system-ui Stack)
- **Einbindung:** Immer mit `preconnect` für Performance

```html
<link rel="preconnect" href="https://fonts.googleapis.com">
<link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
<link href="https://fonts.googleapis.com/css2?family=Inter:wght@300;400;500;600;700&display=swap" rel="stylesheet">
```

### Schriftgrößen

| Element             | Größe     | Gewicht | Farbe          |
|--------------------|----------|---------|----------------|
| Hero h1            | 2rem     | 700     | white          |
| Hero h2            | 1.1rem   | 400     | white (0.9 opacity) |
| Hero p.lead        | 0.95rem  | 400     | white (0.8 opacity) |
| Section Title h2   | 1.35rem  | 700     | `--text`       |
| Card h3            | 1rem     | 600     | `--text`       |
| Card p             | 0.82rem  | 400     | `--text-light` |
| Footer             | 0.82rem  | 400     | `--text-light` |
| Buttons            | 0.82rem  | 500     | abhängig       |
| Badge/Kategorie    | 0.7rem   | 600     | abhängig       |

## Seitenstruktur

Jede GitHub-Pages-Seite folgt diesem Aufbau:

```
┌──────────────────────────────────────────┐
│  HERO (Gradient + Welle)                 │
│  ┌────────────────────────────┐          │
│  │  Logo (Fluviaone.png)     │          │
│  │  H1 Haupttitel            │          │
│  │  H2 Untertitel            │          │
│  │  P  Beschreibungstext     │          │
│  └────────────────────────────┘          │
│  ~~~~ Wellen-SVG (Übergang) ~~~~        │
├──────────────────────────────────────────┤
│  MAIN (max-width: 1200px)               │
│  ┌──────┐ ┌──────┐ ┌──────┐             │
│  │Intro │ │Intro │ │Intro │ ← 3 Karten │
│  │Card  │ │Card  │ │Card  │             │
│  └──────┘ └──────┘ └──────┘             │
│                                          │
│  ── Section Title ──                     │
│  ┌──────────┐ ┌──────────┐              │
│  │ Content  │ │ Content  │ ← Grid      │
│  │ Card     │ │ Card     │              │
│  └──────────┘ └──────────┘              │
├──────────────────────────────────────────┤
│  FOOTER                                  │
│  ─── Gradient-Linie ───                  │
│  Text · FLUVIA.ONE                       │
└──────────────────────────────────────────┘
```

## Komponenten im Detail

### 1. Hero-Header

Der Hero nutzt einen Gradient-Hintergrund mit dekorativen Radialverläufen
und einer SVG-Welle als Übergang zum Content-Bereich.

```html
<div class="hero">
    <div class="hero-content">
        <img src="Fluviaone.png" alt="FLUVIA.ONE Logo"
             style="height: 56px; margin-bottom: 20px;
                    filter: brightness(0) invert(1); opacity: 0.95;">
        <h1>Seitentitel</h1>
        <h2>Untertitel</h2>
        <p class="lead">Beschreibung</p>
    </div>
    <div class="wave-bottom">
        <svg viewBox="0 0 1440 48" preserveAspectRatio="none"
             xmlns="http://www.w3.org/2000/svg">
            <path d="M0,24 C240,48 480,0 720,24 C960,48 1200,0 1440,24
                     L1440,48 L0,48 Z" fill="#f4f8fb"/>
        </svg>
    </div>
</div>
```

**Wichtig:**
- Logo wird mit `filter: brightness(0) invert(1)` weiß dargestellt
- `fill` der SVG-Welle MUSS `--bg` (#f4f8fb) entsprechen
- Hero-Hintergrund: `linear-gradient(135deg, --primary, --primary-dark)`
- Dekorative Radialverläufe mit `--primary-light` (25% Opacity) und `--wave-bg` (15% Opacity)

### 2. Intro-Karten

Drei Karten oberhalb des Hauptinhalts, die die wichtigsten Features erklären.

```css
.intro-card {
    background: var(--white);
    border-radius: 15px;           /* Immer 15px! */
    padding: 28px 24px;
    box-shadow: var(--card-shadow);
    text-align: center;
    transition: transform 0.2s, box-shadow 0.2s;
}
```

**Icon-Container:**
- 56×56px, `border-radius: 14px`
- Hintergrund: `--primary-bg`
- SVG-Icon: 28×28px, Farben nur `--primary` und `--primary-light`

**Hover-Effekt:**
- `transform: translateY(-3px)`
- `box-shadow: var(--card-shadow-hover)`

### 3. Section Titles

Überschriften mit blauer Unterstreichung:

```css
.section-title::after {
    content: '';
    width: 48px;
    height: 3px;
    background: var(--primary);
    border-radius: 2px;
}
```

### 4. Content-Karten

```css
.diagram-card {
    background: var(--white);
    border: 1px solid var(--border);
    border-radius: 15px;           /* Immer 15px! */
    box-shadow: var(--card-shadow);
    transition: transform 0.2s, box-shadow 0.2s;
}
```

**Kategorie-Badge:**
- `border-radius: 20px` (Pill-Shape)
- Uppercase, `letter-spacing: 0.6px`

**Card-Actions:**
- Hintergrund: `--bg`
- Obere Border: `1px solid var(--border)`

### 5. Buttons

```css
/* Primary: Blau-weiß */
.btn-primary {
    background: var(--primary);
    color: white;
    border-color: var(--primary);
    border-radius: 8px;
}

/* Secondary: Weiß mit Hover → Primary-Background */
.btn-secondary {
    background: var(--white);
    color: var(--text);
    border-radius: 8px;
}
.btn-secondary:hover {
    background: var(--primary-bg);
    color: var(--primary);
    border-color: var(--primary-light);
}
```

### 6. Modal / Overlay

```css
.modal-overlay {
    background: rgba(44, 62, 80, 0.6);  /* --text basiert */
    backdrop-filter: blur(4px);
}
.modal {
    border-radius: 15px;
    box-shadow: 0 12px 48px rgba(55, 116, 165, 0.25);
}
```

### 7. Footer

```html
<footer>
    <p>Seitentitel · Bereitgestellt von
       <span class="footer-brand">FLUVIA.ONE</span></p>
</footer>
```

- Dekorative Gradient-Linie oben (80px breit, `--primary-light` → `--primary`)
- `FLUVIA.ONE` als `.footer-brand` in `--primary`

## SVG-Icon-Richtlinien

Inline-SVG-Icons folgen der Fluvia-Farbpalette:

- **Primärfarbe:** `fill="#3774a5"` oder `stroke="#356d96"`
- **Sekundärfarbe:** `fill="#6fa3ce"`
- **Weiße Details:** `fill="white"` mit `opacity="0.5"` oder `opacity="0.6"`
- **Viewbox:** Immer angeben, z.B. `viewBox="0 0 28 28"`
- **Hintergrund-Rechteck:** `fill="#e7f1fa"` mit `rx="26"` (bei großen Icons) oder `rx="14"` (bei kleinen)

## Responsive Design

| Breakpoint    | Anpassungen |
|--------------|-------------|
| > 600px      | Standard-Layout (Grid, Hero-Padding 56px) |
| ≤ 600px      | Hero-Padding 36px, h1 1.5rem, Grid 1-spaltig, Modal fullscreen |

## Datei- und Ordnerstruktur

```
docs/
├── index.html              ← Hauptseite
├── Fluviaone.png           ← Logo (wird weiß invertiert im Hero)
└── diagrams/               ← EINZIGER Arbeitsort für draw.io Dateien
    ├── freigegeben.drawio   (im diagrams-Array → auf der Seite sichtbar)
    ├── entwurf.drawio       (NICHT im Array → unsichtbar, aber versioniert)
    └── ...
```

### Workflow für draw.io-Diagramme

1. **Alle** `.drawio`-Dateien werden direkt in `docs/diagrams/` erstellt und bearbeitet
2. **Freigabe** erfolgt durch Eintrag im `diagrams`-Array in `docs/index.html`
3. **Nicht freigegebene** Dateien liegen im Ordner, sind aber auf der Seite unsichtbar
4. Es gibt **keine Kopien** in anderen Ordnern (z.B. `Documentation/`)

> **Wichtig:** `Documentation/` wird NICHT mehr für draw.io-Dateien verwendet.
> Alle Diagramme leben ausschließlich in `docs/diagrams/`.

### Neue Seiten hinzufügen

1. HTML-Datei in `docs/` erstellen
2. CSS-Variablen aus der Referenz-Datei übernehmen
3. Hero → Main → Footer Struktur einhalten
4. Logo einbinden: `<img src="Fluviaone.png" ...>`
5. Keine externen CSS-Frameworks (kein Bootstrap etc.)

## draw.io Viewer Integration

Für interaktive draw.io-Diagramme wird `embed.diagrams.net` per iframe
mit postMessage-Kommunikation verwendet (NICHT `viewer-static.min.js`).

### Warum iframe + postMessage?

- `GraphViewer.createViewerForElement()` zeigt ein **statisches Bild**, das erst
  nach Klick interaktiv wird → schlechte UX
- `viewer.diagrams.net/?#U{url}` funktioniert nicht mit **privaten Repos**
  (der Server kann die Datei nicht abrufen)
- **Lösung:** XML per `fetch()` von der eigenen Domain laden, dann per
  `postMessage` an den iframe senden

### Ablauf

```
1. fetch(fileUrl)  →  XML von GitHub Pages laden (gleiche Domain)
2. iframe erstellen: src="https://embed.diagrams.net/?spin=1&proto=json&configure=1"
3. Event 'configure' empfangen  →  Konfiguration senden
4. Event 'init' empfangen       →  XML per action:'load' senden
5. Diagramm wird sofort interaktiv angezeigt
```

### Code-Template für Viewer

```javascript
async function openViewer(fileUrl, xml) {
    const iframe = document.createElement('iframe');
    iframe.style.cssText = 'width:100%;height:100%;border:none;display:block';
    iframe.src = 'https://embed.diagrams.net/?spin=1&proto=json&configure=1';

    const handler = (evt) => {
        if (evt.origin !== 'https://embed.diagrams.net') return;
        try {
            const msg = JSON.parse(evt.data);
            if (msg.event === 'configure') {
                iframe.contentWindow.postMessage(JSON.stringify({
                    action: 'configure',
                    config: { defaultFonts: [], css: '' }
                }), '*');
            } else if (msg.event === 'init') {
                iframe.contentWindow.postMessage(JSON.stringify({
                    action: 'load', xml: xml, autosave: 0
                }), '*');
            }
        } catch (e) { /* ignore non-JSON */ }
    };
    window.addEventListener('message', handler);
    iframe._messageHandler = handler; // für Cleanup

    container.innerHTML = '';
    container.appendChild(iframe);
}

function closeViewer() {
    const iframe = container.querySelector('iframe');
    if (iframe?._messageHandler) {
        window.removeEventListener('message', iframe._messageHandler);
    }
    container.innerHTML = '';
}
```

## Diagramm-Definitionen

Neue Diagramme werden als JavaScript-Objekte im `diagrams`-Array registriert:

```javascript
{
    file: 'dateiname.drawio',     // Dateiname in docs/diagrams/
    title: 'Anzeigename',         // Wird in Card + Modal angezeigt
    description: 'Beschreibung.', // 1-2 Sätze
    category: 'Prozess',          // Prozess | Daten | Finanzen
    catClass: 'cat-prozess'       // CSS-Klasse für Badge-Farbe
}
```

### Neues Diagramm hinzufügen (Checkliste)

1. [ ] `.drawio`-Datei nach `docs/diagrams/` kopieren
2. [ ] Eintrag im `diagrams`-Array in `index.html` ergänzen
3. [ ] Kategorie korrekt setzen (inkl. `catClass`)
4. [ ] Beschreibung in 1-2 Sätzen formulieren
5. [ ] Testen: Öffnen, Zoom, Download, diagrams.net-Link

## GitHub Pages Konfiguration

- **Source:** Branch `dev_stage`, Ordner `/docs`
- **Visibility:** Erfordert GitHub Pro bei privaten Repos
- **Build:** Automatisch nach jedem Push (kein manuelles Rebuild nötig)
- **Deploy-Zeit:** ca. 1-2 Minuten nach Push

## Anti-Patterns (VERMEIDE!)

❌ Bootstrap, Tailwind oder andere CSS-Frameworks einbinden
❌ Farben als Hex-Werte direkt verwenden statt CSS-Variablen
❌ `border-radius` anders als 15px für Karten / 8px für Buttons / 20px für Badges
❌ `viewer-static.min.js` für draw.io (zeigt nur statisches Bild)
❌ Dateien außerhalb von `docs/` referenzieren (GitHub Pages kann sie nicht ausliefern)
❌ Externe Abhängigkeiten außer Google Fonts und embed.diagrams.net
❌ Dark Mode – aktuell nicht vorgesehen (nur Light)
