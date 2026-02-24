---
applyTo: "**/models/**/*.py"
---

# Odoo 19 – ORM, Models & Felder

## Model-Typen

```python
from odoo import models, fields, api

# Persistentes Model (Datenbanktabelle)
class MyModel(models.Model):
    _name = 'my.module.model'
    _description = 'Beschreibung des Models'
    _order = 'sequence, name'
    _rec_name = 'name'

# Transientes Model (temporär, für Wizards)
class MyWizard(models.TransientModel):
    _name = 'my.module.wizard'
    _description = 'Wizard Beschreibung'

# Abstraktes Model (kein Datenbanktabelle, nur Mixin)
class MyMixin(models.AbstractModel):
    _name = 'my.module.mixin'
    _description = 'Mixin Beschreibung'
```

## Vererbungsmuster

```python
# 1. Erweiterung eines bestehenden Models (häufigster Fall)
class SaleOrderExtension(models.Model):
    _inherit = 'sale.order'  # Kein _name → erweitert das existierende Model

    custom_field = fields.Char(string='Eigenes Feld')

# 2. Neues Model basierend auf bestehendem (Kopie)
class CustomSaleOrder(models.Model):
    _name = 'custom.sale.order'
    _inherit = 'sale.order'  # Mit _name → neues Model, kopiert Felder

# 3. Delegation (Komposition)
class DetailedProduct(models.Model):
    _name = 'detailed.product'
    _inherits = {'product.template': 'product_tmpl_id'}

    product_tmpl_id = fields.Many2one('product.template', required=True, ondelete='cascade')
    detail_field = fields.Char()

# 4. Mehrfachvererbung (Mixins)
class MyModel(models.Model):
    _name = 'my.model'
    _inherit = ['mail.thread', 'mail.activity.mixin']  # Chatter-Integration
```

### Odoo 19 Inheritance-Neuerungen

- **_inherit wird strikt validiert**: Keine widersprüchlichen Felddefinitionen mehr erlaubt
- **Automatische Dependency-Erkennung**: Framework erkennt abhängige Models aus _inherit und _depends
- **Dynamische Priorisierung**: Bei Multi-Modul-Customizations wird das zuletzt geerbte Model priorisiert

## Feldtypen

```python
# Einfache Felder
name = fields.Char(string='Name', required=True, index=True, translate=True)
description = fields.Text(string='Beschreibung')
active = fields.Boolean(default=True)
sequence = fields.Integer(default=10)
amount = fields.Float(digits=(12, 2))
price = fields.Monetary(currency_field='currency_id')
date = fields.Date(default=fields.Date.today)
datetime_field = fields.Datetime(default=fields.Datetime.now)
state = fields.Selection([
    ('draft', 'Entwurf'),
    ('confirmed', 'Bestätigt'),
    ('done', 'Erledigt'),
], default='draft', string='Status', tracking=True)
html_content = fields.Html(sanitize=True)
image = fields.Image(max_width=1024, max_height=1024)
binary_file = fields.Binary(attachment=True)
color = fields.Integer(string='Farbe')

# Relationale Felder
partner_id = fields.Many2one('res.partner', string='Kunde', ondelete='restrict')
line_ids = fields.One2many('my.model.line', 'parent_id', string='Positionen')
tag_ids = fields.Many2many('my.model.tag', string='Tags')
currency_id = fields.Many2one('res.currency', string='Währung',
    default=lambda self: self.env.company.currency_id)

# Computed Fields
total = fields.Float(compute='_compute_total', store=True)

@api.depends('line_ids.subtotal')
def _compute_total(self):
    for record in self:
        record.total = sum(record.line_ids.mapped('subtotal'))

# Computed mit Inverse (editierbar)
display_name = fields.Char(compute='_compute_display', inverse='_inverse_display', store=True)

# Related Field (Shortcut für computed)
partner_email = fields.Char(related='partner_id.email', store=True, readonly=False)
```

## API-Dekoratoren

```python
@api.depends('field1', 'field2')          # Für compute-Methoden: Neuberechnung bei Änderung
def _compute_something(self):
    for record in self:
        record.result = record.field1 + record.field2

@api.constrains('field1', 'field2')       # Validierung bei create/write
def _check_something(self):
    for record in self:
        if record.field1 < 0:
            raise ValidationError("Wert darf nicht negativ sein!")

@api.onchange('partner_id')               # UI-Reaktion auf Feldänderung (nur in Forms)
def _onchange_partner(self):
    if self.partner_id:
        self.payment_term_id = self.partner_id.property_payment_term_id

@api.model                                # Methode auf Model-Ebene (nicht auf Recordset)
def get_default_values(self):
    return {}

@api.model_create_multi                   # Optimierter Batch-Create
def create(self, vals_list):
    records = super().create(vals_list)
    # Custom-Logik nach Erstellung
    return records
```

## CRUD-Methoden überschreiben

```python
@api.model_create_multi
def create(self, vals_list):
    for vals in vals_list:
        if not vals.get('reference'):
            vals['reference'] = self.env['ir.sequence'].next_by_code('my.model')
    return super().create(vals_list)

def write(self, vals):
    if 'state' in vals and vals['state'] == 'done':
        self._check_completion()
    return super().write(vals)

def unlink(self):
    if any(record.state == 'done' for record in self):
        raise UserError("Erledigte Datensätze können nicht gelöscht werden!")
    return super().unlink()

def copy(self, default=None):
    default = dict(default or {})
    default['name'] = f"{self.name} (Kopie)"
    return super().copy(default)
```

## Suche & Recordsets

```python
# Suche
records = self.env['res.partner'].search([
    ('is_company', '=', True),
    ('country_id.code', '=', 'DE'),
], limit=10, order='name asc')

# Recordset-Operationen
names = records.mapped('name')                    # Liste aller Namen
companies = records.filtered(lambda r: r.is_company)  # Filtern
sorted_records = records.sorted(key='name')       # Sortieren

# Domänen-Operatoren: =, !=, <, >, <=, >=, like, ilike, in, not in,
#                     child_of, parent_of, =like, =ilike
```

## Performance-Tipps

- **read_group** statt Schleifen für Aggregationen nutzen
- **search_fetch()** und **fetch()** für Cache-Population bei schlechtem Prefetching
- **store=True** bei oft gefilterten Computed Fields (Suche erfolgt über DB)
- **Prefetching**: Odoo liest automatisch alle Felder eines Recordsets auf einmal – vermeide browse() in Schleifen
- **SQL nur bei nachweislichem Bedarf**: `self.env.cr.execute()` mit Parametern (nie String-Formatierung!)
- **Lazy Loading**: Odoo 19 lädt Felder erst bei Zugriff → weniger Speicherverbrauch

## Namenskonventionen

- Model-Name: `mein_modul.model_name` (Punkte, lowercase)
- Python-Klasse: `MeinModulModelName` (CamelCase)
- Felder: `snake_case`
- Many2one: Suffix `_id` (z.B. `partner_id`)
- One2many / Many2many: Suffix `_ids` (z.B. `line_ids`, `tag_ids`)
- Compute-Methoden: `_compute_<feldname>`
- Onchange-Methoden: `_onchange_<feldname>`
- Action-Methoden: `action_<name>` (z.B. `action_confirm`)
- Constraint-Methoden: `_check_<was_geprüft_wird>`
