---
applyTo: "**/*.py,**/*.js,**/*.xml,**/*.scss"
---

# Odoo 19 – Coding Guidelines & Konventionen

## Python-Konventionen

### Import-Reihenfolge

```python
# 1. Standard-Bibliotheken
import base64
import logging
from datetime import datetime, timedelta

# 2. Odoo-Imports
from odoo import api, fields, models, _
from odoo.exceptions import UserError, ValidationError
from odoo.tools import float_compare, float_is_zero

# 3. Modul-Imports
from odoo.addons.my_module.utils import helper_function
```

### Modell-Attribut-Reihenfolge

```python
class MyModel(models.Model):
    # 1. Private Attribute
    _name = 'my.model'
    _description = 'Mein Model'
    _inherit = ['mail.thread', 'mail.activity.mixin']
    _order = 'sequence, name'
    _rec_name = 'name'
    _sql_constraints = [
        ('name_uniq', 'unique(name, company_id)', 'Name muss pro Firma eindeutig sein!'),
    ]

    # 2. Default-Methode
    def _default_company(self):
        return self.env.company

    # 3. Feld-Deklarationen (Reihenfolge: Basic → Relational → Computed)
    name = fields.Char(required=True, index=True)
    active = fields.Boolean(default=True)
    sequence = fields.Integer(default=10)
    state = fields.Selection([...], default='draft')
    company_id = fields.Many2one('res.company', default=_default_company)
    partner_id = fields.Many2one('res.partner')
    line_ids = fields.One2many('my.model.line', 'parent_id')
    tag_ids = fields.Many2many('my.model.tag')
    total = fields.Float(compute='_compute_total', store=True)

    # 4. Compute-Methoden
    @api.depends('line_ids.amount')
    def _compute_total(self):
        for record in self:
            record.total = sum(record.line_ids.mapped('amount'))

    # 5. Constraint-Methoden
    @api.constrains('amount')
    def _check_amount(self):
        for record in self:
            if record.amount < 0:
                raise ValidationError(_("Betrag darf nicht negativ sein."))

    # 6. Onchange-Methoden
    @api.onchange('partner_id')
    def _onchange_partner(self):
        if self.partner_id:
            self.currency_id = self.partner_id.currency_id

    # 7. CRUD-Methoden (create, write, unlink, copy)
    @api.model_create_multi
    def create(self, vals_list):
        return super().create(vals_list)

    # 8. Action-Methoden
    def action_confirm(self):
        self.ensure_one()
        self.write({'state': 'confirmed'})

    # 9. Business-Methoden
    def _prepare_invoice_values(self):
        self.ensure_one()
        return {...}
```

### Namenskonventionen

| Element | Konvention | Beispiel |
|---------|-----------|---------|
| Model-Name | `modul.model` (lowercase, Punkte) | `sale.order.line` |
| Python-Klasse | CamelCase | `SaleOrderLine` |
| Felder | snake_case | `partner_id`, `total_amount` |
| Many2one | `_id` Suffix | `partner_id` |
| One2many/M2M | `_ids` Suffix | `line_ids`, `tag_ids` |
| Compute | `_compute_<feld>` | `_compute_total` |
| Onchange | `_onchange_<feld>` | `_onchange_partner` |
| Constraint | `_check_<was>` | `_check_amount` |
| Action | `action_<verb>` | `action_confirm` |
| Private | `_`-Präfix | `_prepare_values` |
| Wizard | `modul.model.wizard` | `sale.advance.payment.inv` |

### Wichtige Python-Regeln

- **`_()` für übersetzbare Strings**: `raise UserError(_("Fehler: %s", name))`
- **`%` statt `.format()`** bevorzugen (besser für Übersetzungen): `_("Wert: %s") % value`
- **Kein `\n` in Strings** – Odoo erwartet separate Paragraph-Elemente
- **`ensure_one()`** in Action-Methoden die auf einem Record arbeiten
- **`with_context()` statt Context direkt ändern**: `records.with_context(key=val)`
- **`filtered()`, `mapped()`, `sorted()`** statt Schleifen
- **Logging**: `_logger = logging.getLogger(__name__)`

## XML-Konventionen

### Datei-Organisation

```
views/
├── my_model_views.xml       # Form, List, Search eines Models
├── my_model_templates.xml   # Website/Portal-Templates (getrennt!)
├── menuitems.xml            # Menüs separat
```

### XML-ID-Konventionen

| Typ | Muster | Beispiel |
|-----|--------|---------|
| View | `view_<model>_<type>` | `view_sale_order_form` |
| Action | `action_<model>` | `action_sale_order` |
| Menü | `menu_<bereich>` | `menu_sale_root` |
| Sequenz | `seq_<model>` | `seq_sale_order` |
| Gruppe | `group_<modul>_<rolle>` | `group_sale_manager` |
| Record Rule | `rule_<model>_<wer>` | `rule_sale_order_user` |

### XML-Daten-Regeln

- `<data>` nur mit `noupdate="1"` verwenden, sonst direkt unter `<odoo>`
- Records nach Model gruppieren
- Abhängigkeitsreihenfolge beachten (Security vor Views)

## JavaScript-Konventionen

```javascript
/** @odoo-module **/

// Immer @odoo-module Header für das Modulsystem

// Klassen: CamelCase
export class MyComponent extends Component { }

// Funktionen/Methoden: camelCase
onButtonClick() { }

// Konstanten: UPPER_SNAKE_CASE
const MAX_RETRIES = 3;

// CSS-Klassen: o_ Präfix mit snake_case
// o_my_component, o_my_button

// Template-Namen: addon_name.ComponentName
static template = 'my_module.MyComponent';
```

## SCSS-Konventionen

```scss
// Odoo-Variablen nutzen statt harte Werte
.o_my_component {
    height: $o-statusbar-height;
    background: $o-view-background-color;

    .o_my_inner {
        // Verschachtelung statt flache Selektoren
        padding: $o-horizontal-padding;
    }
}

// Klassen-Prefix: o_ (Odoo-Konvention)
// Odoo-SCSS-Variablen statt harte Pixelwerte
// Keine !important außer in absoluten Ausnahmefällen
```

## Commit-Nachricht-Format

```
[TAG] module_name: Kurzbeschreibung

Ausführliche Beschreibung (optional)

closes #task-id
```

**Tags:**
- `[IMP]` – Verbesserung/Enhancement
- `[FIX]` – Bugfix
- `[ADD]` – Neue Funktionalität
- `[REM]` – Entfernung von Code
- `[REF]` – Refactoring (keine funktionale Änderung)
- `[MOV]` – Code verschoben
- `[REV]` – Revert
- `[I18N]` – Übersetzungen

## Performance-Richtlinien

- **ORM statt SQL** – Außer bei nachweislichem Performance-Problem
- **read_group()** für Aggregationen statt Python-Schleifen
- **store=True** für oft durchsuchte Computed Fields
- **Batch-Operationen**: `create(vals_list)` statt Schleifen
- **Kein browse() in Schleifen** – Vorher IDs sammeln, einmal browsen
- **Prefetching respektieren**: Felder über Recordset-Iteration zugreifen
- **SQL-Constraints** statt Python-Constraints wo möglich (performanter)
- **Index** auf häufig gefilterte/sortierte Felder setzen

## Häufige Anti-Patterns

```python
# ❌ SCHLECHT: browse in Schleife
for id in ids:
    record = self.browse(id)

# ✅ GUT: einmal browsen
records = self.browse(ids)
for record in records:
    ...

# ❌ SCHLECHT: Felder einzeln lesen
for record in records:
    name = record.read(['name'])[0]['name']

# ✅ GUT: direkt zugreifen (Prefetching!)
for record in records:
    name = record.name

# ❌ SCHLECHT: Summe in Python
total = sum(r.amount for r in records)

# ✅ GUT: Aggregation in DB
result = self.read_group(domain, ['amount:sum'], [])

# ❌ SCHLECHT: SQL-Injection-Risiko
self.env.cr.execute("SELECT * FROM my_table WHERE name = '%s'" % name)

# ✅ GUT: Parametrisiert
self.env.cr.execute("SELECT * FROM my_table WHERE name = %s", (name,))
```
