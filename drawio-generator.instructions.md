---
applyTo: "**/*.drawio"
---

# draw.io Diagramme – Style-Guide & Generierungsregeln

## Überblick

Du bist ein Experte für die Erstellung von draw.io-Diagrammen im nativen
XML-Format (mxGraphModel). Du erstellst `.drawio`-Dateien, die direkt in
VS Code mit der draw.io-Extension oder in der draw.io-Web-App geöffnet
werden können.

Diese Instruction definiert den visuellen Standard für alle draw.io-Diagramme
im WoWiLift-Projekt. Alle generierten `.drawio`-Dateien MÜSSEN diesen
Konventionen folgen, damit ein einheitliches Erscheinungsbild gewährleistet ist.

> **Referenz-Datei:** `Documentation/reparaturprozess-flowchart.drawio`

## Dateistruktur

```xml
<mxfile host="65bd71144e">
    <diagram id="UNIQUE_ID" name="Diagrammtitel">
        <mxGraphModel grid="1" page="1" gridSize="10" guides="1"
            tooltips="1" connect="1" arrows="1" fold="1"
            pageScale="1" pageWidth="1600" pageHeight="2600"
            math="0" shadow="0">
            <root>
                <mxCell id="0"/>
                <mxCell id="1" parent="0"/>
                <!-- Inhalte hier -->
            </root>
        </mxGraphModel>
    </diagram>
</mxfile>
```

### Mehrere Seiten

Mehrere `<diagram>`-Elemente innerhalb von `<mxfile>` sind erlaubt
(z.B. Hauptflow + Technische Implementierung).

### Wichtige Grundregeln

- **IDs:** Jede `mxCell` braucht eine eindeutige `id`. `id="0"` und `id="1"` sind
  reserviert (Root-Container). Eigene Elemente starten ab `id="2"`.
- **Parent:** Alle sichtbaren Elemente haben `parent="1"`.
- **Vertices (Formen):** Verwende `vertex="1"` und ein `<mxGeometry>`-Element
  mit `x`, `y`, `width`, `height`.
- **Edges (Verbindungen):** Verwende `edge="1"` mit `source` und `target`
  Attributen, die auf existierende Vertex-IDs verweisen.
- **Styles:** Styles werden als Semikolon-getrennte Key-Value-Paare im
  `style`-Attribut angegeben.
- **Dateiendung:** Immer `.drawio` verwenden.
- **Validierung:** Alle `source`/`target`-Referenzen MÜSSEN auf existierende
  IDs zeigen. Keine ID darf doppelt vergeben werden.

---

## Farbpalette – Odoo App-Zuordnung

### Semantische Farben (Module / Geschäftsbereiche)

| Bereich | fillColor | strokeColor | Verwendung |
|---------|-----------|-------------|------------|
| **Helpdesk** | `#f8cecc` | `#b85450` | Tickets, Störmeldungen |
| **CRM** | `#dae8fc` | `#6c8ebf` | Leads, Chancen, Pipeline |
| **Purchase** | `#ffe6cc` | `#d6b656` | EK-Angebote, Lieferanten |
| **Sales** | `#d5e8d4` | `#82b366` | VK-Angebote, Aufträge |
| **Accounting** | `#e1d5e7` | `#9673a6` | Rechnungen, Buchhaltung |
| **Aufzug / Projekt** | `#B2EBF2` | `#00838F` | project.project, zentrales Datenobjekt |

### Funktionale Farben

| Zweck | fillColor | strokeColor | Verwendung |
|-------|-----------|-------------|------------|
| **Entscheidung (Raute)** | `#fff2cc` | `#d6b656` | Ja/Nein-Verzweigungen |
| **Neutral / Allgemein** | `#f5f5f5` | `#666666` | Legende, Rollen, neutrale Boxen |
| **Konfiguration (hervorgehoben)** | `#d5e8d4` | `#2D7600` | Standard-Konfiguration, `strokeWidth=2` |
| **Zu entwickeln (hervorgehoben)** | `#ffe6cc` | `#d6b656` | Neuentwicklung, `strokeWidth=2` |

### Status-Markierungen im Text

| Symbol | Bedeutung | HTML-Darstellung |
|--------|-----------|------------------|
| `★` | Noch zu entwickeln | `&lt;b style=&quot;color:#d6b656&quot;&gt;★&lt;/b&gt;` |
| `✅` | Bereits vorhanden | Direkt als Unicode-Zeichen |
| `📊` | Spätere Phase | Direkt als Unicode-Zeichen |

---

## Formen & Styles

### Prozessschritt (Standard-Box)

Leicht gerundetes Rechteck. **Immer `arcSize=4`** für subtile Rundung.

```
style="rounded=1;arcSize=4;whiteSpace=wrap;html=1;fillColor=#FARBE;strokeColor=#FARBE;fontSize=11;"
```

**Beispiele:**

```xml
<!-- Einfacher Prozessschritt -->
<mxCell id="2" value="&lt;b&gt;Titel&lt;/b&gt;&lt;br&gt;Beschreibung"
    style="rounded=1;arcSize=4;whiteSpace=wrap;html=1;fillColor=#f8cecc;strokeColor=#b85450;fontSize=11;"
    parent="1" vertex="1">
    <mxGeometry x="40" y="80" width="190" height="60" as="geometry"/>
</mxCell>

<!-- Detailreicher Prozessschritt -->
<mxCell id="10" value="&lt;b&gt;MODULNAME&lt;/b&gt;&lt;br&gt;Beschreibung&lt;br&gt;• Punkt 1&lt;br&gt;• Punkt 2"
    style="rounded=1;arcSize=4;whiteSpace=wrap;html=1;fillColor=#dae8fc;strokeColor=#6c8ebf;fontSize=11;align=left;spacingLeft=10;"
    parent="1" vertex="1">
    <mxGeometry x="210" y="420" width="370" height="190" as="geometry"/>
</mxCell>
```

### Entscheidung (Raute)

```
style="rhombus;whiteSpace=wrap;html=1;fillColor=#fff2cc;strokeColor=#d6b656;fontSize=11;"
```

**Typische Größe:** `width="130" height="90"`

```xml
<mxCell id="25" value="Prüfung&lt;br&gt;bestanden?"
    style="rhombus;whiteSpace=wrap;html=1;fillColor=#fff2cc;strokeColor=#d6b656;fontSize=11;"
    parent="1" vertex="1">
    <mxGeometry x="330" y="990" width="130" height="90" as="geometry"/>
</mxCell>
```

### Start / Ende (Ellipse)

```
style="ellipse;whiteSpace=wrap;html=1;fillColor=#FARBE;strokeColor=#FARBE;fontSize=11;"
```

**Typische Größe:** `width="130-150" height="55"`

```xml
<mxCell id="50" value="Vorgang&lt;br&gt;abgeschlossen"
    style="ellipse;whiteSpace=wrap;html=1;fillColor=#d5e8d4;strokeColor=#82b366;fontSize=11;"
    parent="1" vertex="1">
    <mxGeometry x="540" y="2240" width="130" height="55" as="geometry"/>
</mxCell>
```

### Hinweis / Notiz

```
style="shape=note;whiteSpace=wrap;html=1;size=14;fillColor=#FARBE;strokeColor=#FARBE;fontSize=10;align=left;spacingLeft=6;verticalAlign=top;spacingTop=4;"
```

Für optionale Hinweise zusätzlich `dashed=1;` hinzufügen.

```xml
<mxCell id="57" value="&lt;b&gt;Hinweistitel&lt;/b&gt;&lt;br&gt;Erklärungstext"
    style="shape=note;whiteSpace=wrap;html=1;size=14;fillColor=#fff2cc;strokeColor=#d6b656;fontSize=10;align=left;spacingLeft=6;verticalAlign=top;spacingTop=4;"
    parent="1" vertex="1">
    <mxGeometry x="700" y="700" width="280" height="150" as="geometry"/>
</mxCell>
```

### Infobox / Sidebar-Box (Legende, Rollen, Zusammenfassung)

```
style="rounded=1;arcSize=4;whiteSpace=wrap;html=1;fillColor=#f5f5f5;strokeColor=#666666;fontSize=10;align=left;verticalAlign=top;spacingLeft=8;spacingTop=4;"
```

Für hervorgehobene Boxen: `strokeWidth=2;` ergänzen.

### Weitere Formen (für spezielle Diagrammtypen)

| Typ | Style |
|-----|-------|
| **Rechteck (eckig)** | `rounded=0;whiteSpace=wrap;html=1;` |
| **Zylinder (Datenbank)** | `shape=cylinder3;whiteSpace=wrap;html=1;size=15;` |
| **Dokument** | `shape=document;whiteSpace=wrap;html=1;` |
| **Parallelogramm** | `shape=parallelogram;whiteSpace=wrap;html=1;` |
| **Hexagon** | `shape=hexagon;perimeter=hexagonPerimeter2;whiteSpace=wrap;html=1;` |
| **Cloud** | `ellipse;shape=cloud;whiteSpace=wrap;html=1;` |
| **Person/Actor** | `shape=mxgraph.basic.person;whiteSpace=wrap;html=1;` |
| **Swimlane** | `swimlane;startSize=23;` |

> Bei diesen Formen immer die Projektfarben aus der Farbpalette anwenden
> (`fillColor`, `strokeColor`) und `fontSize=11;` verwenden.

### Überschrift / Titel

```
style="text;html=1;fontSize=18;fontStyle=1;align=center;verticalAlign=middle;whiteSpace=wrap;"
```

```xml
<mxCell id="200" value="Diagramm-Titel"
    style="text;html=1;fontSize=18;fontStyle=1;align=center;verticalAlign=middle;whiteSpace=wrap;"
    parent="1" vertex="1">
    <mxGeometry x="200" y="10" width="700" height="40" as="geometry"/>
</mxCell>
```

---

## Verbindungen (Edges)

### Standard-Verbindung

```
style="edgeStyle=orthogonalEdgeStyle;rounded=1;"
```

### Verbindung mit Label

```
style="edgeStyle=orthogonalEdgeStyle;rounded=1;fontSize=10;"
```

Bei Labels auf Ja/Nein-Verzweigungen Schriftfarbe setzen:
- **Ja:** `fontColor=#82b366;` (grün)
- **Nein:** `fontColor=#b85450;` (rot)

```xml
<mxCell id="69" value="Ja"
    style="edgeStyle=orthogonalEdgeStyle;rounded=1;fontSize=10;fontColor=#82b366;"
    parent="1" source="25" target="30" edge="1">
    <mxGeometry relative="1" as="geometry"/>
</mxCell>
```

### Optionale / Hinweis-Verbindung (gestrichelt)

```
style="edgeStyle=orthogonalEdgeStyle;rounded=1;dashed=1;endArrow=open;endFill=0;"
```

### Nicht-gerichtete Verbindung (z.B. Zugehörigkeit)

```
style="dashed=1;endArrow=none;endFill=0;strokeColor=#6c8ebf;"
```

### Hervorgehobene Verbindung (z.B. neue Features)

```
style="edgeStyle=orthogonalEdgeStyle;rounded=1;fontSize=9;strokeWidth=2;strokeColor=#d6b656;"
```

### Bidirektionale Verbindung

```
style="edgeStyle=orthogonalEdgeStyle;rounded=1;dashed=1;strokeColor=#00838F;fontSize=9;startArrow=classic;startFill=1;"
```

---

## Legende

Jedes Diagramm MUSS eine Legende enthalten. Farben werden als **farbige Quadrate**
(HTML-Inline-Spans) dargestellt, NICHT als geschriebene Farbnamen.

### Farbfeld-Template

```
&lt;span style=&quot;display:inline-block;width:12px;height:12px;background:FILL_COLOR;border:1px solid STROKE_COLOR;vertical-align:middle;margin-right:5px;&quot;&gt;&lt;/span&gt; LABEL
```

### Vollständiges Legenden-Beispiel

```xml
<mxCell id="100"
    value="&lt;b&gt;Legende – Odoo Apps&lt;/b&gt;&lt;hr&gt;&lt;span style=&quot;display:inline-block;width:12px;height:12px;background:#f8cecc;border:1px solid #b85450;vertical-align:middle;margin-right:5px;&quot;&gt;&lt;/span&gt; HELPDESK&lt;br&gt;&lt;span style=&quot;display:inline-block;width:12px;height:12px;background:#dae8fc;border:1px solid #6c8ebf;vertical-align:middle;margin-right:5px;&quot;&gt;&lt;/span&gt; CRM&lt;br&gt;&lt;span style=&quot;display:inline-block;width:12px;height:12px;background:#ffe6cc;border:1px solid #d6b656;vertical-align:middle;margin-right:5px;&quot;&gt;&lt;/span&gt; PURCHASE&lt;br&gt;&lt;span style=&quot;display:inline-block;width:12px;height:12px;background:#d5e8d4;border:1px solid #82b366;vertical-align:middle;margin-right:5px;&quot;&gt;&lt;/span&gt; SALES&lt;br&gt;&lt;span style=&quot;display:inline-block;width:12px;height:12px;background:#e1d5e7;border:1px solid #9673a6;vertical-align:middle;margin-right:5px;&quot;&gt;&lt;/span&gt; ACCOUNTING&lt;br&gt;&lt;span style=&quot;display:inline-block;width:12px;height:12px;background:#B2EBF2;border:1px solid #00838F;vertical-align:middle;margin-right:5px;&quot;&gt;&lt;/span&gt; AUFZUG (project.project)&lt;hr&gt;&lt;span style=&quot;display:inline-block;width:12px;height:12px;background:#fff2cc;border:1px solid #d6b656;vertical-align:middle;margin-right:5px;&quot;&gt;&lt;/span&gt; Entscheidung (Raute)&lt;br&gt;Gestrichelt = Hinweis / Optional&lt;br&gt;Ellipse = Start / Ende&lt;hr&gt;&lt;b style=&quot;color:#d6b656&quot;&gt;★&lt;/b&gt; = Noch zu bauen&lt;br&gt;✅ = Bereits vorhanden&lt;br&gt;📊 = Spätere Phase"
    style="rounded=1;arcSize=4;whiteSpace=wrap;html=1;fillColor=#f5f5f5;strokeColor=#666666;fontSize=11;align=left;verticalAlign=top;spacingLeft=8;spacingTop=4;"
    parent="1" vertex="1">
    <mxGeometry x="1120" y="60" width="310" height="340" as="geometry"/>
</mxCell>
```

### Legende für Nicht-Odoo-Diagramme

Wenn das Diagramm keine Odoo-Module darstellt, nutze trotzdem die gleiche
Farbpalette und erstelle eine passende Legende mit Farbfeldern. Vergib Farben
nach semantischer Bedeutung (z.B. Abteilungen, Systeme, Phasen).

---

## Schriftgrößen

| Element | fontSize |
|---------|----------|
| **Diagramm-Titel** | `18` (+ `fontStyle=1` für Bold) |
| **Prozessschritt** | `11` |
| **Prozessschritt (hervorgehoben)** | `12` (+ `fontStyle=1` für Bold) |
| **Notiz / Hinweis** | `10` |
| **Infobox / Sidebar** | `10` oder `11` |
| **Edge-Label** | `9` oder `10` |

---

## Textformatierung in `value`

Text wird als HTML-encoded Content geschrieben:

| Gewünscht | Schreibweise |
|-----------|-------------|
| **Fett** | `&lt;b&gt;Text&lt;/b&gt;` |
| *Kursiv* | `&lt;i&gt;Text&lt;/i&gt;` |
| Zeilenumbruch | `&lt;br&gt;` |
| Horizontale Linie | `&lt;hr&gt;` |
| Aufzählung | `• Punkt 1&lt;br&gt;• Punkt 2` |
| Farbiger Text | `&lt;b style=&quot;color:#d6b656&quot;&gt;Text&lt;/b&gt;` |
| Trennlinie (ASCII) | `─────────────────` |

---

## Layout-Richtlinien

### Allgemein

- **Grid:** `gridSize="10"` – alle Positionen auf 10er-Raster ausrichten
- **Hauptflow:** Von oben nach unten, linksbündig
- **Sidebar-Elemente:** Rechts neben dem Hauptflow (x ≥ 1100)
- **Margin:** Mindestens 10px Abstand zwischen Elementen

### Typische Breiten

| Element | Breite |
|---------|--------|
| **Einfacher Prozessschritt** | 170–280px |
| **Detaillierter Prozessschritt** | 270–370px |
| **Entscheidungs-Raute** | 130px |
| **Notiz** | 220–280px |
| **Sidebar-Info** | 280–320px |
| **Ellipse (Start/Ende)** | 130–150px |

### Textausrichtung in Detail-Boxen

Bei Boxen mit Aufzählungslisten:
```
align=left;spacingLeft=10;
```

Bei Boxen mit Kopfzeile + Inhalt:
```
align=left;verticalAlign=top;spacingLeft=8;spacingTop=4;
```

---

## ID-Konventionen

| Elemente | ID-Schema | Beispiel |
|----------|-----------|---------|
| **Prozessschritte (Hauptflow)** | Aufsteigende Nummern | `2`, `3`, `10`, `15` |
| **Sidebar / Legende** | 100er-Nummern | `100`, `200`, `300` |
| **Erweiterungen / Zusätze** | 300er-Nummern | `320`, `321`, `322` |
| **Edges** | 60er-Nummern | `60`, `61`, `62` (oder `TE1`, `TE2`) |
| **Zweites Diagramm** | Prefix `T` | `T1`, `T10`, `T11` |

---

## Erstellungsprozedur

Wenn ein Diagramm angefragt wird, folge diesen Schritten:

1. **Typ bestimmen:** Kläre den Diagrammtyp
   (Flowchart, Architektur, ER-Diagramm, Sequenz, Netzwerk, etc.).
2. **Elemente identifizieren:** Bestimme alle Knoten und Verbindungen.
3. **Farben zuweisen:** Ordne Farben aus der Farbpalette nach
   semantischer Bedeutung zu (Odoo-Module, Abteilungen, Phasen).
4. **Layout berechnen:** Ordne Elemente mit ausreichend Abstand an
   (mindestens 40px zwischen Elementen).
   - Vertikale Layouts: Start bei `y=40`, `deltaY=100-120`
   - Horizontale Layouts: Start bei `x=40`, `deltaX=200`
5. **XML generieren:** Erstelle valides draw.io XML nach den Regeln
   dieser Instruction.
6. **Legende einfügen:** Erstelle Legende mit Farbfeld-Quadraten auf
   der rechten Seite des Diagramms.
7. **Datei speichern:** Speichere als `.drawio`-Datei im Verzeichnis
   `Documentation/`.
8. **Validieren:** Prüfe, dass alle `source`/`target`-Referenzen auf
   existierende IDs zeigen und keine ID doppelt vergeben ist.

---

## Diagrammtyp-spezifische Hinweise

### Flowchart / Prozessdiagramm

- Vertikaler Flow von oben nach unten
- Ellipsen für Start/Ende
- Rauten für Entscheidungen mit Ja/Nein-Beschriftung
- Abgerundete Rechtecke für Prozessschritte
- Notiz-Shapes für Hinweise und Sonderfälle
- Sidebar rechts für Legende, Rollen, Zusammenfassungen

### Architekturdiagramm

- Verwende Swimlanes (`swimlane;startSize=23;`) für Schichten
  (Frontend, Backend, Datenbank)
- Cloud-Shapes für externe Services
- Zylinder für Datenbanken
- Bidirektionale Pfeile für API-Kommunikation

### ER-Diagramm (Entity-Relationship)

- Rechtecke für Entitäten (mit Feldlisten)
- Rauten für Beziehungen
- Beschrifte Edges mit Kardinalitäten (`1`, `n`, `m`)
- Verwende `align=left;spacingLeft=10;` für Feldlisten

### Sequenzdiagramm

- Vertikale Lebenslinien mit `dashed=1;`
- Horizontale Pfeile zwischen Akteuren
- Aktivierungsboxen als schmale Rechtecke
  (`width=20; height=variabel`)

### Netzwerkdiagramm

- Verwende `shape=mxgraph.cisco.*` oder `shape=mxgraph.aws4.*`
  Styles für Netzwerk-/Cloud-Symbole
- Cloud-Shape für Internet/externe Netze
- Zylinder für Speichersysteme

### Modulübersicht / Technische Architektur

- Abgerundete Rechtecke für Module/Komponenten
- Farben nach Modul-Zuordnung (siehe Farbpalette)
- Gestrichelte Verbindungen für `_inherit`-Beziehungen
- `strokeWidth=2;` für Schlüsselmodule

---

## Vollständiges Flowchart-Beispiel

```xml
<mxfile host="65bd71144e">
  <diagram id="flow1" name="Beispiel-Flowchart">
    <mxGraphModel grid="1" page="1" gridSize="10" guides="1"
        tooltips="1" connect="1" arrows="1" fold="1"
        pageScale="1" pageWidth="1169" pageHeight="827"
        math="0" shadow="0">
      <root>
        <mxCell id="0"/>
        <mxCell id="1" parent="0"/>
        <!-- Start -->
        <mxCell id="2" value="Start"
            style="ellipse;whiteSpace=wrap;html=1;fillColor=#d5e8d4;strokeColor=#82b366;fontSize=11;"
            vertex="1" parent="1">
          <mxGeometry x="350" y="40" width="130" height="55" as="geometry"/>
        </mxCell>
        <!-- Prozess -->
        <mxCell id="3" value="&lt;b&gt;Daten verarbeiten&lt;/b&gt;"
            style="rounded=1;arcSize=4;whiteSpace=wrap;html=1;fillColor=#dae8fc;strokeColor=#6c8ebf;fontSize=11;"
            vertex="1" parent="1">
          <mxGeometry x="330" y="140" width="170" height="60" as="geometry"/>
        </mxCell>
        <!-- Entscheidung -->
        <mxCell id="4" value="Gültig?"
            style="rhombus;whiteSpace=wrap;html=1;fillColor=#fff2cc;strokeColor=#d6b656;fontSize=11;"
            vertex="1" parent="1">
          <mxGeometry x="350" y="240" width="130" height="90" as="geometry"/>
        </mxCell>
        <!-- Ende -->
        <mxCell id="5" value="Ende"
            style="ellipse;whiteSpace=wrap;html=1;fillColor=#f8cecc;strokeColor=#b85450;fontSize=11;"
            vertex="1" parent="1">
          <mxGeometry x="350" y="380" width="130" height="55" as="geometry"/>
        </mxCell>
        <!-- Verbindungen -->
        <mxCell id="6" style="edgeStyle=orthogonalEdgeStyle;rounded=1;"
            edge="1" source="2" target="3" parent="1">
          <mxGeometry relative="1" as="geometry"/>
        </mxCell>
        <mxCell id="7" style="edgeStyle=orthogonalEdgeStyle;rounded=1;"
            edge="1" source="3" target="4" parent="1">
          <mxGeometry relative="1" as="geometry"/>
        </mxCell>
        <mxCell id="8" value="Ja"
            style="edgeStyle=orthogonalEdgeStyle;rounded=1;fontSize=10;fontColor=#82b366;"
            edge="1" source="4" target="5" parent="1">
          <mxGeometry relative="1" as="geometry"/>
        </mxCell>
        <mxCell id="9" value="Nein"
            style="edgeStyle=orthogonalEdgeStyle;rounded=1;fontSize=10;fontColor=#b85450;"
            edge="1" source="4" target="3" parent="1">
          <mxGeometry relative="1" as="geometry">
            <Array as="points">
              <mxPoint x="560" y="285"/>
              <mxPoint x="560" y="170"/>
            </Array>
          </mxGeometry>
        </mxCell>
      </root>
    </mxGraphModel>
  </diagram>
</mxfile>
```

---

## Anti-Patterns (NICHT MACHEN)

❌ `rounded=1` ohne `arcSize=4` → Ecken zu stark gerundet

❌ Farbnamen in der Legende (`Rot = ...`) → immer Farbfeldquadrate verwenden

❌ `shadow=1` → Kein Schatten verwenden

❌ Eigene Farben erfinden → Immer aus der definierten Farbpalette wählen

❌ Schriftgröße kleiner als 9 → Nicht lesbar in PDF-Exports

❌ Boxen ohne `html=1;whiteSpace=wrap;` → Textdarstellung bricht

❌ Edges ohne `edgeStyle=orthogonalEdgeStyle;` → Unordentliche Linienführung

---

## Checkliste vor Abschluss

- [ ] Alle Boxen nutzen `rounded=1;arcSize=4;` (keine Ausnahme für Rechtecke)?
- [ ] Farbpalette eingehalten (keine fremden Farben)?
- [ ] Legende vorhanden mit Farbfeld-Quadraten?
- [ ] Schriftgrößen konsistent (11 für Prozess, 10 für Notizen)?
- [ ] Grid-Ausrichtung eingehalten (10er-Raster)?
- [ ] Alle Edges haben `edgeStyle=orthogonalEdgeStyle;rounded=1;`?
- [ ] Ja/Nein-Labels an Rauten mit korrekter Farbcodierung?
- [ ] Titel-Element vorhanden (fontSize=18)?
- [ ] Keine `shadow=1` Attribute?
- [ ] Alle `source`/`target`-Referenzen zeigen auf existierende IDs?
- [ ] Keine ID doppelt vergeben?
- [ ] Datei im `Documentation/`-Verzeichnis gespeichert?
