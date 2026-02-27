---
applyTo: "**"
---

# Odoo 19 – Audiotranskripte in Anforderungen & Gap-Analyse überführen

## Zweck

Diese Instruction beschreibt, wie lange Audiotranskripte (Meetings, Workshops,
Interviews, Sprachnotizen) systematisch in eine belastbare
**Anforderungsliste** und anschließend in die Odoo-**Gap-Analyse** überführt
werden.

## Einsatzfall

Nutze diese Instruction, wenn der Nutzer:
- ein Roh-Transkript einfügt,
- einen sehr langen Fließtext als Gesprächsprotokoll liefert,
- aus gesprochenen Anforderungen eine strukturierte Umsetzungsbasis braucht.

## Grundregeln

1. **Keine Implementierung vor Analyse** – erst Anforderungen, dann Planung.
2. **Transkript ist Quelle, nicht Wahrheit** – Unklarheiten als Rückfrage markieren.
3. **Keine Halluzinationen** – nur Aussagen aus dem Transkript verwenden.
4. **Widersprüche explizit kennzeichnen** statt stillschweigend auflösen.
5. **Deutsch, präzise, umsetzungsnah** formulieren.

## Workflow

### Phase 1: Eingang normalisieren

- Entferne Füllwörter nur dann, wenn die fachliche Aussage unverändert bleibt.
- Erhalte fachliche Begriffe, Rollen, Zahlen, Fristen, Schwellenwerte.
- Segmentiere das Transkript in logische Blöcke (Themen/Abschnitte).
- Vergib Abschnitts-IDs für spätere Referenzen, z.B. `T01`, `T02`, ...

### Phase 2: Inhalte extrahieren

Extrahiere aus jedem Block:
- **Ziele/Business Outcome**
- **Akteure/Rollen**
- **Prozessschritte & Trigger**
- **Datenobjekte/Felder/Belege**
- **Regeln, Grenzwerte, Fristen**
- **Ausnahmen/Sonderfälle**
- **Integrationen (E-Mail, API, DMS, etc.)**
- **Nicht-funktionale Anforderungen** (Performance, Sicherheit, Audit, UX)

### Phase 3: Anforderungsliste erstellen

Erzeuge eine tabellarische Anforderungsliste:

```markdown
## Anforderungsliste aus Transkript

| ID | Priorität | Kategorie | Anforderung | Quelle | Akzeptanzkriterium | Offen/Frage |
|----|-----------|-----------|-------------|--------|--------------------|-------------|
| R-001 | Muss | Prozess | ... | T03 | ... | Nein |
```

Regeln:
- **Priorität:** Muss / Soll / Kann
- **Kategorie:** Prozess / Daten / UI / Reporting / Integration / Security / Sonstiges
- **Quelle:** Immer Abschnitts-ID (z.B. `T03`) angeben
- **Akzeptanzkriterium:** testbar und eindeutig formulieren
- Wenn unklar: `Offen/Frage = Ja` + konkrete Rückfrage

### Phase 4: Überführung in Odoo-Gap-Analyse

Nach der Anforderungsliste eine Gap-Analyse erzeugen:

```markdown
## Gap-Analyse: [Thema aus Transkript]

| # | Anforderung (ID) | Standard-Abdeckung | Handlungsbedarf |
|---|------------------|-------------------|-----------------|
| 1 | R-001 | ... | ✅ Konfiguration |
```

Nutze diese Klassifikation:
- ✅ **Konfiguration** = Standardfunktion vorhanden, nur Einstellungen/Daten
- 🔧 **Erweiterung** = vorhandenes Model/View per `_inherit` erweitern
- 🆕 **Neuentwicklung** = neues Feature/Model erforderlich
- ⚠️ **Prüfen** = Aussage unklar, Rückfrage nötig

### Phase 5: Qualitätssicherung der Analyse

Prüfe vor Ausgabe:
- Sind alle Muss-Anforderungen mit Akzeptanzkriterium versehen?
- Gibt es doppelte oder widersprüchliche Anforderungen?
- Sind Zahlen/Schwellenwerte/Frequenzen konsistent?
- Sind offene Punkte als Fragen explizit gesammelt?

### Phase 6: Abschluss mit Freigabefrage

Abschluss immer mit:

> "Hier ist die aus dem Transkript extrahierte Anforderungsliste inklusive Gap-Analyse.
> Bevor ich mit einer technischen Umsetzung beginne:
> - Sind die Anforderungen vollständig und korrekt priorisiert?
> - Soll ich offene Punkte zuerst klären?
> - Soll ich auf dieser Basis den Umsetzungsplan erstellen?"

## Ausgabeformat bei langen Transkripten

Wenn das Transkript sehr lang ist, antworte in dieser Reihenfolge:

1. **Kurz-Zusammenfassung** (max. 8 Bulletpoints)
2. **Anforderungsliste** (Tabelle)
3. **Offene Fragen** (nummeriert)
4. **Gap-Analyse** (Tabelle)
5. **Nächster Schritt/Freigabe**

## Anti-Patterns

❌ Direkt Code oder Modulstruktur liefern, ohne Anforderungsliste
❌ Aussagen ergänzen, die im Transkript nicht vorkommen
❌ Vage Anforderungen ohne Akzeptanzkriterium stehen lassen
❌ Widersprüche ignorieren
❌ Gap-Analyse ohne Referenz auf Anforderungen (R-IDs)
