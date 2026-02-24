---
applyTo: "**/data/knowledge_article.xml"
---

# Odoo 19 – Dokumentation & Wissensdatenbank aktualisieren

## Zweck

Diese Instruction steuert den Prozess zur Aktualisierung des Knowledge-Artikels
in `wowilift_custom/data/knowledge_article.xml`. Dieser XML-Data-Record wird bei
jedem Modul-Update (`odoo-bin -u wowilift_custom`) automatisch in die Odoo
Wissensdatenbank übertragen.

## Source of Truth

Die **XML-Datei im Repository** ist die einzige Quelle der Wahrheit:
`wowilift_custom/data/knowledge_article.xml`

Der alte Markdown-Artikel `Documentation/Wowilift_customizations.md` dient nur
noch als Referenz. Alle Änderungen werden direkt in der XML-Datei vorgenommen.

## Wichtig: noupdate="0"

Der Record verwendet `noupdate="0"`, d.h. er wird bei **jedem** Modul-Update
überschrieben. Manuelle Änderungen im Odoo-Editor gehen beim nächsten Update
verloren. Das ist gewollt – die XML-Datei ist die Source of Truth.

## HTML-Struktur im Artikel

### Grundgerüst

```xml
<field name="body" type="html">
    <div>
        <!-- Metadaten-Banner -->
        <div style="background-color: #d4edda; ...">
            <strong>📄 Dokumentationsstand: TT.MM.JJJJ</strong>
        </div>

        <!-- Abschnitte -->
        <h2>N. Abschnittstitel</h2>
        <h3>N.1 Unterabschnitt</h3>
        <!-- Inhalt -->

        <!-- Abnahme-Block -->
        <div style="background-color: #f8f5fa; border-left: 4px solid #875A7B; ...">
            <h3>Abnahme [Abschnittsname]</h3>
            <!-- Abnahme-Punkte -->
        </div>
    </div>
</field>
```

### Stil-Bausteine

#### Felder auflisten
```html
<ul>
    <li><strong>Feldname</strong> (Feldtyp): Beschreibung</li>
</ul>
```

#### Automation beschreiben
```html
<div style="background-color: #e8f4f8; border-left: 3px solid #17a2b8; padding: 10px 14px; margin: 10px 0;">
    <strong>Automation: automation_name</strong><br/>
    Trigger: Wann wird sie ausgelöst<br/>
    Beschreibung was passiert
</div>
```

#### Abnahme-Block (bestehender Abschnitt)
```html
<div style="background-color: #f8f5fa; border-left: 4px solid #875A7B; padding: 12px 16px; margin: 20px 0;">
    <h3>Abnahme [Abschnittsname]</h3>
    <p>Die im Abschnitt "N. Name" beschriebenen Funktionen wurden erfolgreich
    vorgeführt, gemeinsam geprüft und für in Ordnung befunden:</p>
    <ul>
        <li><strong>Punkt:</strong> abgenommen.</li>
    </ul>
    <p><strong>Kommentare:</strong></p>
    <p style="background-color: #fff3cd; padding: 8px; border: 1px solid #ffc107; border-radius: 4px;">
        Kommentartext oder <em>[Platzhalter für Kommentare]</em>
    </p>
</div>
```

#### Abnahme-Block (neuer/noch nicht abgenommener Abschnitt)
```html
<div style="background-color: #f8f5fa; border-left: 4px solid #875A7B; padding: 12px 16px; margin: 20px 0;">
    <h3>Abnahme [Neuer Abschnittsname]</h3>
    <p>Die im Abschnitt "N. Name" beschriebenen Funktionen wurden noch nicht abgenommen.</p>
    <p><strong>Kommentare:</strong></p>
    <p style="background-color: #fff3cd; padding: 8px; border: 1px solid #ffc107; border-radius: 4px;">
        <em>[Ausstehende Abnahme]</em>
    </p>
</div>
```

## Workflow: Dokumentation aktualisieren

### Schritt 1: Änderungen ermitteln

```bash
# Letzte Änderungen an Custom-Modulen
git log --since="YYYY-MM-DD" --oneline --no-merges -- wowilift_custom/ wowilift_pdf_parser/

# Geänderte Dateien
git diff --stat HEAD~N -- wowilift_custom/ wowilift_pdf_parser/
```

### Schritt 2: Änderungen kategorisieren

Ordne jede Änderung einem bestehenden Abschnitt zu:

| Abschnitt | Betroffene Dateien/Models |
|-----------|--------------------------|
| 1. Kernentitäten | `res_partner.py`, `vertrag.py`, `project_project.py` |
| 2. Kontaktmanagement | `res_partner.py`, `res_partner_views.xml` |
| 3. Aufzugsanlagen | `project_project.py`, `project_project_views.xml` |
| 4. Vertragsverwaltung | `vertrag.py`, `preisanpassung.py`, `rabatt_wartung.py` |
| 5. Verkaufsmanagement | `sale_order.py`, `sale_order_line.py` |
| 6. Rechnungsstellungen | `account_move.py`, Report-Templates |
| 7. Datenimport | `migrations/` |
| 8. Technische Besonderheiten | Infrastruktur, Fehlerbehandlung |
| NEU: Helpdesk/PDF-Parser | `wowilift_pdf_parser/` |
| NEU: Einkauf | `purchase_order.py`, RFQ-Reports |

### Schritt 3: HTML im XML aktualisieren

1. Lies `wowilift_custom/data/knowledge_article.xml`
2. Finde den passenden Abschnitt (HTML-Kommentare markieren die Abschnitte)
3. Füge neuen HTML-Content ein oder aktualisiere bestehenden
4. Aktualisiere das Datum im Metadaten-Banner

### Schritt 4: Screenshots empfehlen

```
📸 Screenshot-Liste für Abschnitt X:
1. [ ] Form-View von [Model] mit ausgefüllten Feldern
2. [ ] List-View mit Beispieldaten
3. [ ] Beispiel-Output (z.B. generierte Rechnung)
```

Screenshots werden vom Kunden manuell direkt in Odoo eingefügt
(in einem separaten Kind-Artikel, der nicht überschrieben wird).

## Inhaltliche Regeln

- **Sprache:** Deutsch, formell, aus Kundenperspektive
- **Kein Code** in der Dokumentation (kein Python, kein XML)
- **Keine internen Feldnamen** ohne Erklärung (z.B. `fv_kontakttyp` → "Kontakttyp")
- **Keine Git-Referenzen** (keine Commit-Hashes oder Branch-Namen)
- **Abgenommene Punkte** nicht ändern, nur neue Punkte hinzufügen
- **Kommentare** des Kunden beibehalten (auch negative)
- **Nummerierung** fortlaufend: neuer Abschnitt = nächste Nummer

## XML-spezifische Hinweise

- `&amp;` statt `&` in HTML-Attributen und Text
- Alle Tags müssen geschlossen werden (`<br/>` nicht `<br>`)
- Keine `<script>` oder `<style>` Tags – nur Inline-Styles
- `type="html"` im `<field>`-Tag nicht vergessen
- HTML muss wohlgeformt sein (valid XHTML)

## Anti-Patterns

❌ Technische Implementierungsdetails einfügen
❌ Abgenommene Abschnitte inhaltlich ändern
❌ Markdown-Syntax im HTML verwenden
❌ Odoo-Artikel manuell editieren statt die XML-Datei
❌ `noupdate="1"` setzen (verhindert Updates)
