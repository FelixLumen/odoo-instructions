---
applyTo: "**/tests/**/*.py"
---

# Odoo 19 – Testing & Qualitätssicherung

## Testarten in Odoo

1. **Python Unit Tests** – Geschäftslogik testen
2. **JavaScript Unit Tests** – Frontend-Code isoliert testen
3. **Tour-Tests (Integration)** – End-to-End-Tests mit simuliertem Browser

## Teststruktur

```
my_module/
├── tests/
│   ├── __init__.py           # Importiert alle test_-Module
│   ├── test_my_model.py      # Model-Tests
│   ├── test_access.py        # Sicherheitstests
│   └── test_workflow.py      # Workflow-Tests
```

```python
# tests/__init__.py
from . import test_my_model
from . import test_access
from . import test_workflow
```

## Python Unit Tests

### TransactionCase (Standard)

Jede Testmethode läuft in einer eigenen Transaktion, die am Ende zurückgerollt wird:

```python
from odoo.tests import TransactionCase, tagged
from odoo.exceptions import ValidationError, UserError

@tagged('post_install', '-at_install')  # Nach Installation ausführen
class TestMyModel(TransactionCase):

    @classmethod
    def setUpClass(cls):
        """Testdaten einmalig für die gesamte Klasse erstellen."""
        super().setUpClass()
        cls.partner = cls.env['res.partner'].create({
            'name': 'Testkunde',
            'email': 'test@example.com',
        })
        cls.record = cls.env['my.model'].create({
            'name': 'Testeintrag',
            'partner_id': cls.partner.id,
        })

    def test_create_record(self):
        """Testen der Record-Erstellung."""
        record = self.env['my.model'].create({
            'name': 'Neuer Eintrag',
            'partner_id': self.partner.id,
        })
        self.assertTrue(record.exists())
        self.assertEqual(record.state, 'draft')

    def test_compute_total(self):
        """Testen von berechneten Feldern."""
        self.record.line_ids = [
            (0, 0, {'product_id': 1, 'quantity': 2, 'price_unit': 10.0}),
            (0, 0, {'product_id': 2, 'quantity': 1, 'price_unit': 25.0}),
        ]
        self.assertAlmostEqual(self.record.total, 45.0, places=2)

    def test_action_confirm(self):
        """Testen einer Statusänderung."""
        self.record.action_confirm()
        self.assertEqual(self.record.state, 'confirmed')

    def test_constraint_negative_amount(self):
        """Testen einer Validierung."""
        with self.assertRaises(ValidationError):
            self.record.write({'amount': -100})

    def test_unlink_done_record(self):
        """Testen, dass erledigte Records nicht gelöscht werden können."""
        self.record.action_confirm()
        self.record.action_done()
        with self.assertRaises(UserError):
            self.record.unlink()
```

### Sicherheits-Tests

```python
@tagged('post_install', '-at_install')
class TestMyModelAccess(TransactionCase):

    @classmethod
    def setUpClass(cls):
        super().setUpClass()
        # Testbenutzer mit spezifischen Gruppen erstellen
        cls.user_basic = cls.env['res.users'].create({
            'name': 'Basis-Nutzer',
            'login': 'basic_user',
            'groups_id': [(6, 0, [
                cls.env.ref('my_module.group_my_module_user').id,
            ])],
        })
        cls.user_manager = cls.env['res.users'].create({
            'name': 'Manager',
            'login': 'manager_user',
            'groups_id': [(6, 0, [
                cls.env.ref('my_module.group_my_module_manager').id,
            ])],
        })

    def test_user_cannot_delete(self):
        """Basis-Nutzer darf nicht löschen."""
        record = self.env['my.model'].with_user(self.user_basic).create({
            'name': 'Test',
        })
        with self.assertRaises(Exception):
            record.unlink()

    def test_user_sees_only_own_records(self):
        """Basis-Nutzer sieht nur eigene Einträge."""
        # Record als Admin erstellen
        admin_record = self.env['my.model'].create({'name': 'Admin-Eintrag'})

        # Als Basis-Nutzer suchen
        visible = self.env['my.model'].with_user(self.user_basic).search([])
        self.assertNotIn(admin_record, visible)

    def test_manager_sees_all(self):
        """Manager sieht alle Einträge."""
        visible = self.env['my.model'].with_user(self.user_manager).search([])
        self.assertTrue(len(visible) > 0)
```

### HttpCase (für Controller-Tests)

```python
from odoo.tests import HttpCase

@tagged('post_install', '-at_install')
class TestMyController(HttpCase):

    def test_homepage_loads(self):
        """Testen, ob die Seite lädt."""
        response = self.url_open('/my/page')
        self.assertEqual(response.status_code, 200)

    def test_json_endpoint(self):
        """Testen eines JSON-Endpoints."""
        self.authenticate('admin', 'admin')
        response = self.url_open('/my/api/data', data='{}',
                                  headers={'Content-Type': 'application/json'})
        self.assertEqual(response.status_code, 200)
```

## Performance-Tests

```python
def test_create_performance(self):
    """Testen der Abfrageanzahl."""
    with self.assertQueryCount(admin=3):
        # Maximal 3 DB-Queries für diese Operation
        self.env['my.model'].create({'name': 'Performance-Test'})
```

## Tour-Tests (Integration/E2E)

```javascript
// static/tests/my_tour.js
import { registry } from "@web/core/registry";

registry.category("web_tour.tours").add("my_module_tour", {
    test: true,
    url: "/odoo/my-model",
    steps: () => [
        {
            trigger: ".o_list_button_add",
            content: "Klicke auf 'Neu'",
            run: "click",
        },
        {
            trigger: ".o_field_widget[name='name'] input",
            content: "Name eingeben",
            run: "edit Tour-Test-Eintrag",
        },
        {
            trigger: ".o_form_button_save",
            content: "Speichern",
            run: "click",
        },
        {
            trigger: ".o_field_widget[name='name']:contains('Tour-Test-Eintrag')",
            content: "Prüfen ob gespeichert",
        },
    ],
});
```

```python
# Tour in Python ausführen
@tagged('post_install', '-at_install')
class TestMyTour(HttpCase):
    def test_tour(self):
        self.start_tour("/odoo/my-model", "my_module_tour", login="admin")
```

## Tests ausführen

```bash
# Alle Tests eines Moduls
odoo-bin -d testdb -i my_module --test-enable --stop-after-init

# Spezifische Tests mit Tags
odoo-bin -d testdb --test-tags /my_module

# Nur Post-Install-Tests
odoo-bin -d testdb --test-tags post_install

# CLI-Parameter für Debugging
odoo-bin --test-enable --log-level=test --log-sql  # SQL-Queries loggen
```

## Test-Tags

```python
@tagged('post_install', '-at_install')  # Standard: nach Installation
@tagged('at_install')                    # Während Installation
@tagged('-standard', 'nice')             # Nicht in Standard-Suite, eigener Tag
@tagged('post_install', '-at_install', 'my_feature')  # Eigener Tag
```

## Best Practices

- **setUpClass** statt setUp für Testdaten → schneller (einmalige Erstellung)
- **Demo-Daten in Tests vermeiden** – eigene Testdaten erstellen
- **Jeden Testfall isolieren** – keine Abhängigkeiten zwischen Tests
- **Beschreibende Methodennamen**: `test_<was_getestet_wird>`
- **Edge Cases testen**: Leere Werte, Negativwerte, Grenzwerte
- **Sicherheit separat testen**: Eigene Testklasse für Access Rights/Record Rules
- **assertQueryCount** für performancekritische Operationen
- **BaseCommon** Testklasse (ab Odoo 16+) für optimierten Kontext (Tracking deaktiviert etc.)
