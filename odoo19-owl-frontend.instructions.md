---
applyTo: "**/static/src/**/*.js,**/static/src/**/*.xml"
---

# Odoo 19 – OWL Frontend-Framework

## Überblick

OWL (Odoo Web Library) ist Odoos eigenes UI-Framework (~20kb gzipped). Es kombiniert Konzepte aus React (Hooks, reaktives System) und Vue (Template-Syntax) mit XML-QWeb-Templates.

## Grundstruktur einer OWL-Komponente

Eine Komponente besteht typischerweise aus 3 Dateien am gleichen Ort:

```
static/src/components/my_component/
├── my_component.js      # Logik
├── my_component.xml     # Template
└── my_component.scss    # Styling (optional)
```

### JavaScript-Datei

```javascript
/** @odoo-module **/

import { Component, useState, useRef, onMounted } from "@odoo/owl";
import { useService } from "@web/core/utils/hooks";
import { registry } from "@web/core/registry";

export class MyComponent extends Component {
    static template = 'my_module.MyComponent';
    static props = {
        title: { type: String, optional: true },
        records: { type: Array },
    };

    setup() {
        // Hooks nur in setup() aufrufen!
        this.state = useState({
            counter: 0,
            isOpen: false,
        });

        // Services nutzen
        this.orm = useService("orm");
        this.notification = useService("notification");
        this.action = useService("action");

        // Refs für DOM-Zugriff
        this.inputRef = useRef("myInput");

        // Lifecycle
        onMounted(() => {
            console.log("Komponente eingehängt");
        });
    }

    increment() {
        this.state.counter++;
    }

    async loadData() {
        const records = await this.orm.searchRead(
            "res.partner",
            [["is_company", "=", true]],
            ["name", "email"]
        );
        // Verarbeitung...
    }

    showNotification() {
        this.notification.add("Aktion erfolgreich!", {
            type: "success",  // success, warning, danger, info
            sticky: false,
        });
    }

    openRecord(recordId) {
        this.action.doAction({
            type: "ir.actions.act_window",
            res_model: "res.partner",
            res_id: recordId,
            views: [[false, "form"]],
        });
    }
}
```

### XML-Template

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<templates xml:space="preserve">
    <t t-name="my_module.MyComponent">
        <div class="o_my_component">
            <h3 t-out="props.title"/>
            <div class="o_counter">
                <span t-out="state.counter"/>
                <button class="btn btn-primary" t-on-click="increment">
                    Erhöhen
                </button>
            </div>
            <input t-ref="myInput" type="text"
                   class="form-control"
                   t-on-keyup="onKeyup"/>
            <t t-foreach="props.records" t-as="rec" t-key="rec.id">
                <div class="o_record_item" t-on-click="() => this.openRecord(rec.id)">
                    <t t-out="rec.name"/>
                </div>
            </t>
        </div>
    </t>
</templates>
```

### Template-Namenskonvention

```
addon_name.ComponentName
```

## Registries

Registries sind Key-Value-Speicher für erweiterbare Funktionalität:

```javascript
import { registry } from "@web/core/registry";

// Services registrieren
registry.category("services").add("my_service", myServiceDefinition);

// Systray-Items hinzufügen
registry.category("systray").add("my_module.MySystrayItem", {
    Component: MySystrayComponent,
});

// Actions registrieren
registry.category("actions").add("my_module.my_action", MyActionComponent);

// Feldwidgets registrieren
registry.category("fields").add("my_widget", {
    component: MyFieldWidget,
    supportedTypes: ["char", "text"],
});
```

## Wichtige Hooks

```javascript
import { useState, useRef, useEffect, onMounted, onWillStart,
         onWillUpdateProps, onPatched, onWillUnmount } from "@odoo/owl";
import { useService, useBus } from "@web/core/utils/hooks";

setup() {
    // Reaktiver State
    this.state = useState({ value: 0 });

    // DOM-Referenzen
    this.myRef = useRef("refName");

    // Odoo Services
    this.rpc = useService("rpc");
    this.orm = useService("orm");
    this.action = useService("action");
    this.dialog = useService("dialog");
    this.notification = useService("notification");
    this.user = useService("user");

    // Event-Bus lauschen
    useBus(this.env.bus, "WEB_CLIENT_READY", this.onReady);

    // Lifecycle-Hooks
    onWillStart(async () => {
        // Async-Daten laden VOR dem ersten Render
        this.data = await this.orm.searchRead("my.model", [], ["name"]);
    });

    onMounted(() => {
        // DOM ist verfügbar
    });

    onPatched(() => {
        // Nach jedem Re-Render
    });

    onWillUnmount(() => {
        // Cleanup
    });

    // Effekte (reagiert auf Änderungen)
    useEffect(
        () => { /* Effekt ausführen */ },
        () => [this.state.value]  // Dependencies
    );
}
```

## Kommunikation zwischen Komponenten

```javascript
// Parent → Child: über Props
// In Parent-Template:
// <ChildComponent title="'Hallo'" onUpdate.bind="handleUpdate"/>

// Child → Parent: über Callback-Props
class ChildComponent extends Component {
    static props = {
        onUpdate: { type: Function },
    };

    notifyParent() {
        this.props.onUpdate(this.state.value);
    }
}

// Globaler Event-Bus
this.env.bus.trigger("MY_EVENT", { data: "payload" });
useBus(this.env.bus, "MY_EVENT", (ev) => { /* handler */ });
```

## OWL auf Portal/Website verwenden

```javascript
// 1. Komponente erstellen und in public_components Registry registrieren
import { registry } from "@web/core/registry";

registry.category("public_components").add("my_module.MyPublicComponent", {
    Component: MyPublicComponent,
});

// 2. In web.assets_frontend Bundle aufnehmen (__manifest__.py)
// 'assets': { 'web.assets_frontend': ['my_module/static/src/portal_component/**/*'] }

// 3. In Template verwenden
// <owl-component name="my_module.MyPublicComponent" />
```

## Assets-Bundles im Manifest

```python
'assets': {
    'web.assets_backend': [
        'my_module/static/src/**/*',          # Alle Backend-Assets
    ],
    'web.assets_frontend': [
        'my_module/static/src/portal/**/*',   # Portal/Website
    ],
    'web.assets_qweb': [
        # Wird seit Odoo 15 automatisch gehandhabt
    ],
}
```

## Best Practices

- **useState** IMMER verwenden für reaktive Daten – ohne useState kein automatisches Re-Render
- **Props validieren** mit `static props = {...}` für jede Komponente
- **Hooks nur in setup()** aufrufen – nie in anderen Methoden
- **Services statt direktem Zugriff** – `useService("orm")` statt manueller RPC-Aufrufe
- **CSS-Klassen mit o_ Präfix** – z.B. `o_my_component`, `o_my_button`
- **Templates in separaten XML-Dateien** – nicht inline mit `xml\`...\`` (außer für Prototyping)
- **Owl Devtools** Browser-Extension zum Debuggen installieren
