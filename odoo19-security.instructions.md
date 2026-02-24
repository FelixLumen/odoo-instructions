---
applyTo: "**/security/**"
---

# Odoo 19 – Security (Zugriffskontrolle)

## Drei Ebenen der Sicherheit

1. **Access Rights** (ir.model.access) – CRUD-Rechte auf Model-Ebene
2. **Record Rules** (ir.rule) – Feinsteuerung auf Datensatz-Ebene via Domains
3. **Field Access** – Feld-Level-Zugriff über `groups`-Attribut

## 1. Benutzergruppen (security.xml)

Gruppen MÜSSEN vor der CSV-Datei im Manifest stehen!

```xml
<?xml version="1.0" encoding="UTF-8"?>
<odoo>
    <!-- Modul-Kategorie -->
    <record id="module_category_my_module" model="ir.module.category">
        <field name="name">Mein Modul</field>
        <field name="description">Kategorisierung für Mein Modul</field>
        <field name="sequence">100</field>
    </record>

    <!-- Benutzergruppe: Standard-Nutzer -->
    <record id="group_my_module_user" model="res.groups">
        <field name="name">Benutzer</field>
        <field name="category_id" ref="module_category_my_module"/>
        <field name="implied_ids" eval="[(4, ref('base.group_user'))]"/>
        <field name="comment">Kann eigene Einträge lesen und erstellen.</field>
    </record>

    <!-- Benutzergruppe: Manager (erbt von User) -->
    <record id="group_my_module_manager" model="res.groups">
        <field name="name">Manager</field>
        <field name="category_id" ref="module_category_my_module"/>
        <field name="implied_ids" eval="[(4, ref('group_my_module_user'))]"/>
        <field name="comment">Kann alle Einträge verwalten und löschen.</field>
    </record>
</odoo>
```

### Gruppen-Hierarchie

- `implied_ids`: Wenn Nutzer Gruppe A hat, bekommt er automatisch auch Gruppe B
- Gruppen sind **additiv** – Nutzer in mehreren Gruppen bekommt die Vereinigung aller Rechte
- `base.group_user` = Internal User (Standard-Mitarbeiter)
- `base.group_portal` = Portal-Nutzer
- `base.group_public` = Öffentlicher Zugang

## 2. Access Rights (ir.model.access.csv)

```csv
id,name,model_id:id,group_id:id,perm_read,perm_write,perm_create,perm_unlink
access_my_model_user,my.model.user,model_my_model,group_my_module_user,1,1,1,0
access_my_model_manager,my.model.manager,model_my_model,group_my_module_manager,1,1,1,1
access_my_model_line_user,my.model.line.user,model_my_model_line,group_my_module_user,1,1,1,0
access_my_model_tag_user,my.model.tag.user,model_my_model_tag,group_my_module_user,1,0,0,0
```

### CSV-Spalten

| Spalte | Beschreibung |
|--------|-------------|
| `id` | Eindeutige XML-ID (Modul-übergreifend einzigartig!) |
| `name` | Beschreibender Name |
| `model_id:id` | `model_<model_name>` wobei `.` durch `_` ersetzt wird |
| `group_id:id` | XML-ID der Gruppe (leer = alle Benutzer) |
| `perm_read` | 1 = Lesen erlaubt, 0 = nicht erlaubt |
| `perm_write` | 1 = Schreiben erlaubt |
| `perm_create` | 1 = Erstellen erlaubt |
| `perm_unlink` | 1 = Löschen erlaubt |

### Wichtige Regeln

- **model_id Format**: `model_my_model` für Model `my.model` (Punkte → Unterstriche)
- Für Models aus anderen Modulen: `<modul_name>.model_<model_name>`
- Rechte sind **additiv** zwischen Gruppen
- Leere `group_id` = globaler Zugriff (Vorsicht!)
- **Jedes Model braucht mindestens eine Access-Regel** – Odoo warnt sonst beim Laden

## 3. Record Rules (security.xml)

Record Rules schränken Zugriff auf bestimmte Datensätze ein:

```xml
<odoo noupdate="1">
    <!-- Benutzer sehen nur eigene Einträge -->
    <record id="rule_my_model_user" model="ir.rule">
        <field name="name">Mein Model: Nutzer sieht eigene</field>
        <field name="model_id" ref="model_my_model"/>
        <field name="domain_force">[('create_uid', '=', user.id)]</field>
        <field name="groups" eval="[(4, ref('group_my_module_user'))]"/>
        <field name="perm_read" eval="True"/>
        <field name="perm_write" eval="True"/>
        <field name="perm_create" eval="True"/>
        <field name="perm_unlink" eval="True"/>
    </field>

    <!-- Manager sehen alles (keine Domain = kein Filter) -->
    <record id="rule_my_model_manager" model="ir.rule">
        <field name="name">Mein Model: Manager sieht alles</field>
        <field name="model_id" ref="model_my_model"/>
        <field name="domain_force">[(1, '=', 1)]</field>
        <field name="groups" eval="[(4, ref('group_my_module_manager'))]"/>
    </record>

    <!-- Multi-Company Rule -->
    <record id="rule_my_model_company" model="ir.rule">
        <field name="name">Mein Model: Multi-Company</field>
        <field name="model_id" ref="model_my_model"/>
        <field name="domain_force">
            ['|', ('company_id', '=', False),
                  ('company_id', 'in', company_ids)]
        </field>
        <!-- Keine groups = globale Regel -->
    </record>
</odoo>
```

### Record Rules Logik

- **Ohne Gruppe (global)**: Werden UND-verknüpft – alle müssen erfüllt sein
- **Mit Gruppe**: Werden ODER-verknüpft innerhalb der Gruppe
- **Default-Allow**: Keine passende Regel = Zugriff erlaubt (wenn Access Rights OK)
- **Vorsicht mit globalen Regeln**: Mehrere sich widersprechende globale Regeln können ALLEN Zugriff sperren

### Verfügbare Variablen in domain_force

- `user` – Aktueller Benutzer (Singleton-Recordset)
- `company_id` – Aktuelle Firma des Nutzers (einzelne ID)
- `company_ids` – Alle Firmen des Nutzers (Liste von IDs)
- `time` – Python `time`-Modul

## 4. Feld-Level-Sicherheit

```python
# In Model-Definition
secret_field = fields.Char(groups='my_module.group_my_module_manager')
```

```xml
<!-- In View -->
<field name="internal_notes" groups="my_module.group_my_module_manager"/>
```

- Felder mit `groups` werden automatisch aus Views entfernt für nicht-berechtigte Nutzer
- Auch aus `fields_get()`-Antworten entfernt
- Zugriff via Code wirft `AccessError`

## 5. Sicherheitsfallstricke

```python
# GEFÄHRLICH – sudo() umgeht ALLE Sicherheit
record = self.sudo().browse(record_id)

# GEFÄHRLICH – Raw SQL umgeht ORM-Sicherheit
self.env.cr.execute("SELECT * FROM my_model WHERE id = %s", (record_id,))

# BESSER – Zugriffsrechte prüfen
record.check_access('write')  # Wirft AccessError wenn nicht erlaubt

# GEFÄHRLICH – Dynamischer Feldzugriff
getattr(record, field_name)  # Erlaubt Zugriff auf private Attribute!

# SICHER – Über __getitem__
record[field_name]  # Respektiert Field-Access-Regeln
```

### Wichtige Sicherheitsregeln

- Jede öffentliche Methode ist via RPC aufrufbar – Parameter können nicht vertraut werden
- Methoden mit `_`-Präfix sind nicht über RPC erreichbar
- `sudo()` und Raw SQL nur nach expliziter Zugriffsprüfung und mit minimalen User-Inputs
- ACL wird nur bei CRUD-Operationen geprüft, nicht bei Methodenaufrufen
- Admin/Superuser umgeht Record Rules – immer mit regulären Nutzern testen
