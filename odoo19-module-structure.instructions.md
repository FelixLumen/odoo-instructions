---
applyTo: "**/__manifest__.py,**/__init__.py"
---

# Odoo 19 – Modulstruktur & Manifest

## __manifest__.py

Das Manifest ist die zentrale Metadaten-Datei jedes Moduls. Odoo 19 erkennt und installiert Module anhand dieser Datei.

```python
{
    'name': 'Mein Modul',
    'version': '19.0.1.0.0',        # Format: odoo_version.major.minor.patch
    'category': 'Sales',             # Odoo-App-Kategorie
    'summary': 'Kurzbeschreibung in einem Satz',
    'description': """
        Ausführliche Beschreibung des Moduls.
        Markdown wird unterstützt.
    """,
    'author': 'Firma / Entwickler',
    'website': 'https://example.com',
    'license': 'LGPL-3',            # LGPL-3 für Community, OEEL-1 für Enterprise
    'depends': ['base', 'sale'],     # Abhängigkeiten (immer minimal halten)
    'data': [
        'security/security.xml',      # Gruppen ZUERST (vor CSV!)
        'security/ir.model.access.csv',
        'views/my_model_views.xml',
        'data/my_data.xml',
        'report/my_report_template.xml',
    ],
    'demo': [
        'demo/demo_data.xml',
    ],
    'assets': {
        'web.assets_backend': [
            'my_module/static/src/**/*',
        ],
    },
    'installable': True,
    'application': True,             # True = eigenes App-Icon im Menü
    'auto_install': False,
    'sequence': 10,                  # Ladereihenfolge (-10 = früher)
}
```

### Wichtige Regeln

- **Version**: Immer mit `19.0.` beginnen, dann semantische Versionierung
- **depends**: Nur direkte Abhängigkeiten auflisten. Odoo 19 kann _inherit und _depends automatisch auflösen, aber explizite Deklaration bleibt Best Practice
- **data-Reihenfolge**: `security.xml` VOR `ir.model.access.csv` (Gruppen müssen existieren, bevor sie referenziert werden)
- **assets**: Neues Format ab Odoo 15+ – Glob-Patterns statt einzelner Dateien

## __init__.py

```python
# Root __init__.py
from . import models
from . import controllers
from . import wizard

# models/__init__.py
from . import my_model
from . import my_other_model
```

### Datei-Namenskonvention

Für jedes Modell-Set einen Hauptnamen wählen:

| Bereich | Dateiname |
|---------|-----------|
| Models | `models/<main_model>.py` |
| Views | `views/<main_model>_views.xml` |
| Demo | `demo/<main_model>_demo.xml` |
| Data | `data/<main_model>_data.xml` |
| Tests | `tests/test_<main_model>.py` |

Backend-Views und Frontend-Templates in separaten Dateien halten.

## Scaffold-Befehl

```bash
odoo-bin scaffold <module_name> <addons_path>
```

Erzeugt automatisch die Grundstruktur. Danach anpassen.

## Odoo 19 Neuerungen bei der Modul-Architektur

- **Strikte _inherit-Validierung**: Odoo 19 erkennt überlappende Felddefinitionen und löst Konflikte dynamischer auf
- **Inkrementelle Registry-Updates**: Statt komplettem Rebuild wird das Registry inkrementell aktualisiert (~40% schneller)
- **Lazy Field Loading**: Felder und berechnete Werte werden erst bei Zugriff geladen
- **Verbessertes Connection Pooling**: Weniger Idle-Sessions in der Datenbank
- **Python 3.11 Structural Typing**: Explizitere Modul-Deklarationen möglich
