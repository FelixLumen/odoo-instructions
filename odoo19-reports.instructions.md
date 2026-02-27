---
applyTo: "**/report/**"
---

# Odoo 19 – Reports & PDF-Erzeugung

## Überblick

Reports in Odoo basieren auf HTML/QWeb-Templates und werden via **wkhtmltopdf** in PDF konvertiert. Bootstrap-Klassen und FontAwesome sind verfügbar.

## Report-Action definieren

```xml
<record id="action_report_my_model" model="ir.actions.report">
    <field name="name">Mein Bericht</field>
    <field name="model">my.model</field>
    <field name="report_type">qweb-pdf</field>
    <field name="report_name">my_module.report_my_model_document</field>
    <field name="report_file">my_module.report_my_model_document</field>
    <field name="print_report_name">'Bericht_%s' % object.name</field>
    <field name="binding_model_id" ref="model_my_model"/>
    <field name="binding_type">report</field>
    <!-- Optionales Papierformat -->
    <field name="paperformat_id" ref="my_module.paperformat_my_report"/>
</record>
```

### Report-Typen

- `qweb-pdf` – PDF-Ausgabe (Standard)
- `qweb-html` – HTML-Ausgabe im Browser

## Report-Template

```xml
<?xml version="1.0" encoding="UTF-8"?>
<odoo>
    <template id="report_my_model_document">
        <t t-call="web.html_container">
            <t t-foreach="docs" t-as="doc">
                <t t-call="web.external_layout">
                    <div class="page">
                        <!-- Kopfbereich -->
                        <div class="row mb-4">
                            <div class="col-6">
                                <h2>Bericht</h2>
                                <p>
                                    <strong>Referenz:</strong>
                                    <span t-field="doc.name"/>
                                </p>
                                <p>
                                    <strong>Datum:</strong>
                                    <span t-field="doc.date"
                                          t-options='{"format": "dd.MM.yyyy"}'/>
                                </p>
                            </div>
                            <div class="col-6 text-end">
                                <strong>Kunde:</strong><br/>
                                <span t-field="doc.partner_id.name"/><br/>
                                <span t-field="doc.partner_id.street"/><br/>
                                <span t-field="doc.partner_id.zip"/>
                                <span t-field="doc.partner_id.city"/>
                            </div>
                        </div>

                        <!-- Positionstabelle -->
                        <table class="table table-sm o_main_table mt-4">
                            <thead>
                                <tr>
                                    <th>Pos.</th>
                                    <th>Bezeichnung</th>
                                    <th class="text-end">Menge</th>
                                    <th class="text-end">Einzelpreis</th>
                                    <th class="text-end">Gesamt</th>
                                </tr>
                            </thead>
                            <tbody>
                                <t t-foreach="doc.line_ids" t-as="line">
                                    <tr>
                                        <td><t t-out="line_index + 1"/></td>
                                        <td><span t-field="line.name"/></td>
                                        <td class="text-end">
                                            <span t-field="line.quantity"/>
                                        </td>
                                        <td class="text-end">
                                            <span t-field="line.price_unit"
                                                  t-options='{"widget": "monetary",
                                                             "display_currency": doc.currency_id}'/>
                                        </td>
                                        <td class="text-end">
                                            <span t-field="line.subtotal"
                                                  t-options='{"widget": "monetary",
                                                             "display_currency": doc.currency_id}'/>
                                        </td>
                                    </tr>
                                </t>
                            </tbody>
                        </table>

                        <!-- Summenblock -->
                        <div class="row justify-content-end mt-3">
                            <div class="col-4">
                                <table class="table table-sm">
                                    <tr class="border-black">
                                        <td><strong>Gesamtbetrag</strong></td>
                                        <td class="text-end">
                                            <strong>
                                                <span t-field="doc.total"
                                                      t-options='{"widget": "monetary",
                                                                 "display_currency": doc.currency_id}'/>
                                            </strong>
                                        </td>
                                    </tr>
                                </table>
                            </div>
                        </div>

                        <!-- Fußbereich -->
                        <t t-if="doc.notes">
                            <p class="mt-4">
                                <strong>Bemerkungen:</strong><br/>
                                <span t-field="doc.notes"/>
                            </p>
                        </t>
                    </div>
                </t>
            </t>
        </t>
    </template>
</odoo>
```

## Wichtige Template-Elemente

### Layouts

- `web.html_container` – Äußerer Container (notwendig)
- `web.external_layout` – Standard-Header/Footer mit Firmenlogo
- `web.internal_layout` – Nur interner Report-Rahmen

### Feld-Ausgabe

```xml
<!-- t-field: Formatierte Ausgabe (nutzt Widgets) -->
<span t-field="doc.amount"/>

<!-- t-field mit Optionen -->
<span t-field="doc.date" t-options='{"format": "dd.MM.yyyy"}'/>
<span t-field="doc.amount" t-options='{"widget": "monetary",
                                       "display_currency": doc.currency_id}'/>

<!-- t-out: Rohausgabe (HTML-escaped) -->
<span t-out="doc.name"/>
```

### Verfügbare Variablen

- `docs` – Recordset der zu druckenden Dokumente
- `doc` – Aktueller Datensatz (in Schleife)
- `company` – Aktuelle Firma
- `user` – Aktueller Benutzer
- `time` – Python time-Modul
- `formatLang()` – Zahlenformatierung nach Locale
- `format_date()` – Datumsformatierung

### Barcodes

```xml
<!-- QR-Code -->
<img t-att-src="'/report/barcode/QR/%s' % doc.name"
     style="width:100px;height:100px;"/>

<!-- Barcode mit Parametern -->
<img t-att-src="'/report/barcode/?barcode_type=%s&amp;value=%s&amp;width=%s&amp;height=%s'
     % ('Code128', doc.reference, 600, 100)"/>
```

## Papierformat

```xml
<record id="paperformat_my_report" model="report.paperformat">
    <field name="name">Mein Papierformat</field>
    <field name="default" eval="False"/>
    <field name="format">A4</field>
    <field name="orientation">Portrait</field>
    <field name="margin_top">40</field>
    <field name="margin_bottom">28</field>
    <field name="margin_left">7</field>
    <field name="margin_right">7</field>
    <field name="header_line" eval="False"/>
    <field name="header_spacing">35</field>
    <field name="dpi">90</field>
</record>
```

## Custom Report (eigene Daten)

```python
# report/my_report.py
from odoo import api, models

class MyCustomReport(models.AbstractModel):
    _name = 'report.my_module.report_my_model_document'
    _description = 'Custom Report für mein Model'

    @api.model
    def _get_report_values(self, docids, data=None):
        docs = self.env['my.model'].browse(docids)
        return {
            'doc_ids': docids,
            'doc_model': 'my.model',
            'docs': docs,
            'data': data,
            # Eigene berechnete Daten hinzufügen
            'summary': self._compute_summary(docs),
            'chart_data': self._get_chart_data(docs),
        }

    def _compute_summary(self, docs):
        return {
            'total': sum(docs.mapped('total')),
            'count': len(docs),
        }
```

## Übersetzbare Reports

```xml
<!-- Haupttemplate -->
<template id="report_my_model_document">
    <t t-call="web.html_container">
        <t t-foreach="docs" t-as="doc">
            <t t-set="lang" t-value="doc.partner_id.lang"/>
            <t t-call="my_module.report_my_model_document_content"
               t-lang="lang"/>
        </t>
    </t>
</template>

<!-- Übersetzbares Sub-Template -->
<template id="report_my_model_document_content">
    <t t-set="doc" t-value="doc.with_context(lang=lang)"/>
    <t t-call="web.external_layout">
        <div class="page">
            <!-- Inhalte hier werden in der Sprache des Partners gerendert -->
        </div>
    </t>
</template>
```

## Custom CSS für Reports

```xml
<template id="report_assets_common_custom" inherit_id="web.report_assets_common">
    <xpath expr="." position="inside">
        <link href="/my_module/static/src/scss/report.scss"
              rel="stylesheet" type="text/scss"/>
    </xpath>
</template>
```

## Best Practices

- **`t-field` statt `t-out`** für formatierte Feldausgabe verwenden
- **Bootstrap-Klassen** nutzen (col, row, table, text-end etc.)
- **Seitenumbrüche**: `style="page-break-before: always;"` oder `page-break-inside: avoid;`
- **Bilder**: Immer Breite/Höhe angeben (wkhtmltopdf-Kompatibilität)
- **Externe Ressourcen vermeiden**: Alle Assets lokal im Modul
- **Custom Fonts**: Im `web.report_assets_common` Bundle registrieren
- **Testen**: Reports sowohl als PDF als auch HTML prüfen
