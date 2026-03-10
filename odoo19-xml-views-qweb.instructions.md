---
applyTo: "**/*.xml"
---

# Odoo 19 – XML Views & QWeb Templates

## View-Grundstruktur

Alle Views werden als XML-Records definiert und im Manifest unter `data` registriert.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<odoo>
    <!-- View-Definition -->
    <record id="view_my_model_form" model="ir.ui.view">
        <field name="name">my.model.form</field>
        <field name="model">my.model</field>
        <field name="arch" type="xml">
            <!-- View-Inhalt hier -->
        </field>
    </record>
</odoo>
```

## Form View

```xml
<form string="Mein Formular">
    <header>
        <button name="action_confirm" type="object" string="Bestätigen"
                class="oe_highlight" invisible="state != 'draft'"/>
        <button name="action_cancel" type="object" string="Abbrechen"
                invisible="state == 'done'"/>
        <field name="state" widget="statusbar"
               statusbar_visible="draft,confirmed,done"/>
    </header>
    <sheet>
        <div class="oe_title">
            <label for="name"/>
            <h1>
                <field name="name" placeholder="Name eingeben..."/>
            </h1>
        </div>
        <group>
            <group string="Allgemein">
                <field name="partner_id"/>
                <field name="date"/>
                <field name="state" invisible="1"/>
            </group>
            <group string="Finanzen">
                <field name="amount"/>
                <field name="currency_id"/>
            </group>
        </group>
        <notebook>
            <page string="Positionen" name="lines">
                <field name="line_ids">
                    <list editable="bottom">
                        <field name="product_id"/>
                        <field name="quantity"/>
                        <field name="price_unit"/>
                        <field name="subtotal"/>
                    </list>
                </field>
            </page>
            <page string="Notizen" name="notes">
                <field name="notes" placeholder="Interne Notizen..."/>
            </page>
        </notebook>
    </sheet>
    <chatter/>
</form>
```

### Form-View Regeln

- `<header>` für Status-Buttons und Statusbar
- `<sheet>` für den Hauptinhalt
- `<group>` für 2-Spalten-Layout (verschachtelte `<group>` = nebeneinander)
- `<notebook>/<page>` für Tabs
- `<chatter/>` am Ende für Nachrichten-Feed (erfordert `mail.thread` Mixin)
- **invisible** statt `attrs` (seit Odoo 17+): `invisible="state != 'draft'"`

## List/Tree View

```xml
<record id="view_my_model_list" model="ir.ui.view">
    <field name="name">my.model.list</field>
    <field name="model">my.model</field>
    <field name="arch" type="xml">
        <list string="Meine Liste" decoration-danger="state == 'cancel'"
              decoration-success="state == 'done'"
              default_order="date desc" multi_edit="1">
            <field name="name"/>
            <field name="partner_id"/>
            <field name="date"/>
            <field name="amount" sum="Gesamt"/>
            <field name="state" widget="badge"
                   decoration-info="state == 'draft'"
                   decoration-success="state == 'done'"/>
        </list>
    </field>
</record>
```

## Search View

```xml
<record id="view_my_model_search" model="ir.ui.view">
    <field name="name">my.model.search</field>
    <field name="model">my.model</field>
    <field name="arch" type="xml">
        <search string="Suche">
            <field name="name" string="Name"
                   filter_domain="['|', ('name', 'ilike', self), ('reference', 'ilike', self)]"/>
            <field name="partner_id"/>
            <separator/>
            <filter name="filter_draft" string="Entwurf"
                    domain="[('state', '=', 'draft')]"/>
            <filter name="filter_confirmed" string="Bestätigt"
                    domain="[('state', '=', 'confirmed')]"/>
            <separator/>
            <filter name="filter_this_month" string="Dieser Monat"
                    domain="[('date', '>=', (context_today() - relativedelta(day=1)).strftime('%Y-%m-%d'))]"/>
            <group expand="0" string="Gruppieren nach">
                <filter name="group_partner" string="Kunde"
                        context="{'group_by': 'partner_id'}"/>
                <filter name="group_state" string="Status"
                        context="{'group_by': 'state'}"/>
                <filter name="group_month" string="Monat"
                        context="{'group_by': 'date:month'}"/>
            </group>
        </search>
    </field>
</record>
```

## Kanban View

```xml
<record id="view_my_model_kanban" model="ir.ui.view">
    <field name="name">my.model.kanban</field>
    <field name="model">my.model</field>
    <field name="arch" type="xml">
        <kanban default_group_by="state" class="o_kanban_small_column">
            <field name="color"/>
            <templates>
                <t t-name="card">
                    <field name="name" class="fw-bold fs-5"/>
                    <field name="partner_id"/>
                    <div class="d-flex justify-content-between">
                        <field name="date"/>
                        <field name="amount" widget="monetary"/>
                    </div>
                    <field name="tag_ids" widget="many2many_tags"
                           options="{'color_field': 'color'}"/>
                </t>
            </templates>
        </kanban>
    </field>
</record>
```

### Kanban-Besonderheiten

- QWeb-Templates innerhalb von `<templates>`, NICHT Owl-Templates (kein `t-on-click` etc.)
- `record`-Objekt verfügbar mit `.value` (formatiert) und `.raw_value` (Rohwert)
- `widget.deletable` und `widget.editable` zum Prüfen der Rechte

## Actions & Menüs

```xml
<!-- Window Action -->
<record id="action_my_model" model="ir.actions.act_window">
    <field name="name">Meine Einträge</field>
    <field name="res_model">my.model</field>
    <field name="view_mode">list,form,kanban</field>
    <field name="search_view_id" ref="view_my_model_search"/>
    <field name="context">{'default_state': 'draft'}</field>
    <field name="domain">[('active', '=', True)]</field>
    <field name="help" type="html">
        <p class="o_view_nocontent_smiling_face">
            Erstellen Sie Ihren ersten Eintrag
        </p>
    </field>
</record>

<!-- Menü-Hierarchie -->
<menuitem id="menu_my_module_root" name="Mein Modul" sequence="10"
          web_icon="my_module,static/description/icon.png"/>
<menuitem id="menu_my_module_main" name="Einträge"
          parent="menu_my_module_root" sequence="10"
          action="action_my_model"/>
```

## View-Vererbung (Erweiterung bestehender Views)

```xml
<record id="view_partner_form_inherit" model="ir.ui.view">
    <field name="name">res.partner.form.inherit.my_module</field>
    <field name="model">res.partner</field>
    <field name="inherit_id" ref="base.view_partner_form"/>
    <field name="arch" type="xml">
        <!-- Vor einem Element einfügen -->
        <xpath expr="//field[@name='website']" position="before">
            <field name="my_custom_field"/>
        </xpath>

        <!-- Nach einem Element einfügen -->
        <xpath expr="//page[@name='sales_purchases']" position="after">
            <page string="Mein Tab" name="my_tab">
                <field name="my_field"/>
            </page>
        </xpath>

        <!-- Element ersetzen -->
        <xpath expr="//field[@name='phone']" position="replace">
            <field name="phone" widget="phone"/>
        </xpath>

        <!-- Attribute ändern -->
        <xpath expr="//field[@name='email']" position="attributes">
            <attribute name="required">1</attribute>
        </xpath>

        <!-- Kurzform (wenn Element eindeutig) -->
        <field name="website" position="after">
            <field name="my_field"/>
        </field>
    </field>
</record>
```

### XPath-Regeln

- **Niemals Core-XML-IDs überschreiben** – nur erben
- Position-Werte: `before`, `after`, `inside`, `replace`, `attributes`
- `position="inside"` fügt am Ende des Elements ein (Standard)
- Kurzsyntax nur wenn das Element eindeutig ist

## QWeb-Template-Direktiven

```xml
<!-- Ausgabe (automatisch HTML-escaped) -->
<t t-out="variable"/>
<span t-out="record.name"/>

<!-- Bedingungen -->
<t t-if="state == 'draft'">Entwurf</t>
<t t-elif="state == 'done'">Erledigt</t>
<t t-else="">Sonstiges</t>

<!-- Schleifen -->
<t t-foreach="records" t-as="rec">
    <span t-out="rec.name"/>
    <!-- rec_index, rec_size, rec_first, rec_last verfügbar -->
</t>

<!-- Attribute setzen -->
<div t-att-class="'highlight' if important else ''"/>
<a t-att-href="'/my/path/' + str(record.id)"/>

<!-- Variablen -->
<t t-set="total" t-value="sum(lines.mapped('amount'))"/>

<!-- Sub-Templates aufrufen -->
<t t-call="module_name.template_name"/>

<!-- Felder in Reports (formatiert) -->
<span t-field="record.amount"/>
<span t-field="record.date" t-options='{"format": "dd.MM.yyyy"}'/>
```

## XML-Daten & noupdate

```xml
<!-- Aktualisierbare Daten (werden bei Modul-Update überschrieben) -->
<odoo>
    <record id="my_data_record" model="my.model">
        <field name="name">Standardwert</field>
    </record>
</odoo>

<!-- Nicht-aktualisierbare Daten (z.B. Sequenzen, Cron-Jobs) -->
<odoo noupdate="1">
    <record id="seq_my_model" model="ir.sequence">
        <field name="name">Mein Model Sequenz</field>
        <field name="code">my.model</field>
        <field name="prefix">MM/%(year)s/</field>
        <field name="padding">4</field>
    </record>
</odoo>
```

## Best Practices für XML

- Records nach Model gruppieren
- `<data>` nur für `noupdate=1` verwenden, sonst direkt unter `<odoo>`
- Aussagekräftige XML-IDs: `view_<model>_<view_type>`, `action_<model>`, `menu_<bereich>`
- Keine hart-kodierten Strings in Views – Odoo übersetzt automatisch
- `invisible` bevorzugen statt Elemente per XPath zu entfernen
