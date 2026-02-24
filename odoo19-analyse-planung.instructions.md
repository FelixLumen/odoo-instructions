---
applyTo: "**"
---

# Odoo 19 – Analyse & Planung vor der Entwicklung

## ⚠️ DIESE DATEI HAT HÖCHSTE PRIORITÄT

Bevor Code geschrieben, ein Model erstellt oder eine View angelegt wird, MUSS eine strukturierte Analyse stattfinden. Diese Instruction gilt IMMER – bei jeder Anfrage, die eine Änderung oder Neuentwicklung betrifft.

> **📋 NACH DER ANALYSE**: Lade vor der Implementierung die relevanten spezifischen Instructions mit `read_file` aus `.github/instructions/`:
> - Models → `odoo19-orm-models.instructions.md`
> - Views → `odoo19-xml-views-qweb.instructions.md`
> - Security → `odoo19-security.instructions.md`
> - Code-Stil → `odoo19-coding-guidelines.instructions.md`

## Warum Analyse zuerst?

Odoo ist ein riesiges Framework mit hunderten Modulen, tausenden Models und zehntausenden Methoden. Die häufigsten (und teuersten) Fehler in Odoo-Projekten sind:

- Features neu bauen, die der Standard schon kann (nur Konfiguration nötig)
- Bestehende Methoden überschreiben, statt vorhandene Hooks zu nutzen
- Models anlegen, die es in ähnlicher Form schon gibt
- Geschäftslogik implementieren, die durch Odoo-Einstellungen aktivierbar ist
- Automations per Code bauen, statt Server Actions / Automated Actions zu nutzen

## Der Analyse-Workflow im Detail

### Phase 1: Anforderung verstehen und strukturieren

Wenn der Nutzer eine Anforderung beschreibt, zerlege sie zuerst in einzelne Aspekte:

```
ANFORDERUNG: "Wir brauchen ein Freigabesystem für Bestellungen über 5000€"

Zerlegung:
├── Geschäftsprozess: Genehmigung von Einkaufsbestellungen
├── Auslöser: Bestellsumme überschreitet Schwellwert
├── Beteiligte Rollen: Einkäufer, Abteilungsleiter, Geschäftsführung
├── Gewünschtes Verhalten: Bestellung wartet auf Freigabe
├── Betroffene Objekte: purchase.order
└── Benachrichtigungen: Freigeber soll informiert werden
```

**Stelle klärende Rückfragen**, bevor du analysierst:
- Welcher Geschäftsprozess genau?
- Welche Benutzerrollen sind beteiligt?
- Gibt es Ausnahmen oder Sonderfälle?
- Wie soll die Benutzeroberfläche aussehen?
- Gibt es bestehende Workarounds, die abgelöst werden sollen?

### Phase 2: Bestehenden Code im Workspace durchsuchen

Durchsuche den Workspace systematisch. Gehe dabei in dieser Reihenfolge vor:

#### 2a. Relevante Standard-Module identifizieren

```
Frage: Welche Odoo-Standard-Module sind für diese Anforderung relevant?

Suche nach:
→ Module im addons/-Verzeichnis mit passendem Namen
→ __manifest__.py-Dateien: 'summary' und 'description' lesen
→ Models in models/-Verzeichnissen der Kandidaten-Module
```

#### 2b. Bestehende Models und Felder untersuchen

```
Suche in den relevanten Modulen nach:
→ _name und _inherit Deklarationen
→ Felder die zur Anforderung passen
→ Selection-Felder mit Status-Workflows
→ Computed Fields die relevante Logik enthalten
```

#### 2c. Geschäftslogik und Methoden prüfen

```
Suche gezielt nach:
→ action_* Methoden (Workflow-Aktionen)
→ _compute_* Methoden (Berechnungslogik)
→ _check_* Methoden (Validierungen)
→ create/write/unlink Overrides
→ Vorhandene Hooks: _prepare_*, _get_default_*
→ Signals und Automations
```

#### 2d. Views und UI-Elemente checken

```
Suche in views/-Verzeichnissen nach:
→ Bestehende Form/List/Search Views für das Model
→ Bereits definierte Buttons und Statusbar
→ Gruppierungen und Filter
→ Invisible-Bedingungen (zeigen versteckte Features)
```

#### 2e. Konfigurationsoptionen prüfen

```
Viele Features sind in Odoo nur deaktiviert, nicht fehlend:
→ res.config.settings Einträge für das Modul
→ group_*-Felder in Settings (Feature-Toggles per Gruppe)
→ ir.config_parameter Werte
→ Bereits vorhandene Gruppen in security/
```

### Phase 3: Gap-Analyse erstellen

Erstelle eine tabellarische Übersicht:

```markdown
## Gap-Analyse: [Feature-Name]

| # | Anforderung | Standard-Abdeckung | Handlungsbedarf |
|---|------------|-------------------|-----------------|
| 1 | Freigabe ab 5000€ | purchase.order hat `approval` Feature (Settings → Purchases → Purchase Order Approval) | ✅ Konfiguration – Schwellwert setzen |
| 2 | Mehrstufige Freigabe | Standard unterstützt nur 1 Stufe | 🔧 Erweiterung nötig – _inherit purchase.order |
| 3 | E-Mail an Freigeber | mail.thread + Activities vorhanden | ✅ Standard – Automated Action konfigurieren |
| 4 | Dashboard für offene Freigaben | Kein spezifisches Dashboard | 🆕 Neuentwicklung – eigene Action + View |

Legende:
✅ Konfiguration = Nur Einstellungen/Daten ändern, kein Code
🔧 Erweiterung = Bestehende Funktionalität per _inherit erweitern
🆕 Neuentwicklung = Neues Model/Feature notwendig
⚠️ Prüfen = Weitere Analyse/Rückfrage nötig
```

### Phase 4: Umsetzungsplan formulieren

```markdown
## Umsetzungsplan: [Feature-Name]

### Konfigurationsänderungen (kein Code)
1. Purchase Order Approval aktivieren (Settings → Purchases)
2. Schwellwert auf 5000€ setzen
3. Automated Action für E-Mail-Benachrichtigung anlegen

### Modul: purchase_approval_extend
**Zweck:** Mehrstufige Freigabe für Einkaufsbestellungen

**Abhängigkeiten:** purchase, mail

**Änderungen an bestehenden Models:**
- purchase.order (_inherit)
  - Neues Feld: second_approval_required (Boolean, computed)
  - Neues Feld: second_approver_id (Many2one → res.users)
  - Erweiterung: button_approve() – zweite Stufe einfügen
  - Neue Methode: action_request_second_approval()

**Neue Models:**
- (keine)

**View-Änderungen:**
- purchase.order Form: Button "Zweite Freigabe anfordern" hinzufügen
- purchase.order Search: Filter für "Wartet auf 2. Freigabe"

**Security:**
- Neue Gruppe: group_purchase_second_approver
- Record Rule: 2. Freigeber sieht nur zugewiesene Bestellungen

**Geschätzter Aufwand:** [Klein/Mittel/Groß]

**Risiken:**
- Upgrade-Kompatibilität: button_approve() könnte sich in Odoo 20 ändern
- Performance: computed field second_approval_required bei vielen Bestellungen prüfen
```

### Phase 5: Freigabe einholen

**Antworte dem Nutzer mit der Analyse und dem Plan. Frage explizit:**

> "Hier ist meine Analyse und der Umsetzungsplan. Bevor ich mit der Implementierung beginne:
> - Stimmt die Einschätzung der Standard-Abdeckung?
> - Soll ich den Plan so umsetzen, oder gibt es Änderungen?
> - Soll ich mit einem bestimmten Teil anfangen?"

## Suchmuster für den Workspace

### Schnelle Model-Suche

```
Wo finde ich das Model 'sale.order'?
→ Suche nach: _name = 'sale.order' oder _name = "sale.order"
→ Suche nach: _inherit = 'sale.order' (zeigt alle Erweiterungen)
```

### Feature-Discovery

```
Gibt es schon eine Freigabe-Logik?
→ Suche nach: approval, approve, confirm, validate in *.py
→ Suche nach: button_approve, action_approve in *.py
→ Suche nach: state.*approved, state.*to_approve in *.py
```

### Settings-Discovery

```
Welche Konfigurationsoptionen gibt es für Einkauf?
→ Öffne: addons/purchase/models/res_config_settings.py
→ Suche nach: group_* Felder (Feature-Toggles)
→ Suche nach: module_* Felder (optionale Sub-Module)
→ Prüfe: addons/purchase/views/res_config_settings_views.xml
```

### Vererbungsketten nachvollziehen

```
Wer erweitert alles 'sale.order'?
→ Suche workspace-weit nach: _inherit = 'sale.order'
→ Ergebnis zeigt alle Module die das Model erweitern
→ Hilfreich um Seiteneffekte zu verstehen
```

## Standard-Module und ihre Kernfunktionen (Kurzreferenz)

Bevor du ein Feature baust, prüfe ob eines dieser Module es schon abdeckt:

| Anforderung | Prüfe zuerst | Schlüssel-Features |
|------------|-------------|-------------------|
| Genehmigungen | `approvals`, `purchase` (approval setting) | Konfigurierbare Genehmigungsworkflows |
| Automatische Aktionen | `base_automation` | Server Actions basierend auf Triggern |
| Geplante Aktionen | `base` (ir.cron) | Zeitbasierte Automatisierung |
| Benachrichtigungen | `mail`, `sms` | Chatter, Activities, E-Mail-Templates |
| Dokumente/Anhänge | `documents` | DMS mit Tags, Workflows, OCR |
| Berechnungen in Tabellen | `spreadsheet` | Eingebettete Kalkulationen |
| Benutzerdefinierte Felder | `base` (Studio-freie Methode: via _inherit) | Felder ohne Code |
| Berichterstellung | `base`, QWeb Reports | PDF-Reports, Dashboards |
| Zugriffssteuerung | `base` (Gruppen, Rules) | Oft reicht Konfiguration |
| Duplikatsprüfung | Constraints, `_sql_constraints` | Häufig existieren schon welche |
| Import/Export | `base_import` | Standardmäßig für alle Models verfügbar |
| API-Zugriff | `base` (External API) | JSON-2 API, XML-RPC bereits vorhanden |
| KI-Funktionen | `ai` (Odoo 19 neu) | AI Agents, AI Fields, AI Button |

## Checkliste vor dem Coden

Bevor du Code schreibst, beantworte jede Frage mit Ja:

- [ ] Anforderung ist klar und in Teilaspekte zerlegt?
- [ ] Relevante Standard-Module wurden identifiziert und untersucht?
- [ ] Models, Felder und Methoden im Workspace geprüft?
- [ ] Konfigurationsoptionen geprüft (Settings, Automated Actions)?
- [ ] Gap-Analyse ist erstellt (Standard vs. Custom)?
- [ ] Umsetzungsplan ist formuliert?
- [ ] Nutzer hat den Plan abgesegnet?
- [ ] Klar definiert: Was ist Konfiguration, was ist _inherit, was ist neu?

## Anti-Patterns (VERMEIDE DIESE!)

❌ "Hier ist der Code für dein neues Genehmigungsmodul" → ohne zu prüfen ob purchase.approval existiert

❌ Sofort ein neues Model anlegen → ohne zu prüfen ob ein bestehendes erweiterbar ist

❌ Eine Methode komplett überschreiben → ohne die Original-Methode zu lesen und super() zu nutzen

❌ Alle Felder in ein Custom-Modul packen → statt gezielt per _inherit zu erweitern

❌ Server Actions per Code bauen → ohne zu prüfen ob base_automation + UI-Konfiguration reicht

❌ Code liefern ohne Kontext → "Hier ist action_confirm()" ohne zu erklären welches Standard-Verhalten es erweitert
