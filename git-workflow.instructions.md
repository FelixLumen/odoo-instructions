---
applyTo: "**"
---

# Git & Deployment – Workspace-Konventionen

## Workspace-Umgebung

- **Workspace-Pfad:** `\\wsl.localhost\Ubuntu-24.04\home\felix\wowilift_sh`
- **Terminal:** PowerShell 5.1 auf Windows, CWD ist automatisch das Workspace-Root
- **Repository:** `https://github.com/FelixLumen/WoWiLift_sh.git`
- **Hauptbranch:** `main`

## Wichtig: CWD-Verhalten im Terminal

Das Terminal startet im Workspace-Verzeichnis (`wowilift_sh`). Git-Befehle funktionieren direkt **ohne `cd`**. Ein `cd` auf WSL-Pfade (`/home/felix/...`) schlägt in PowerShell fehl.

```powershell
# ❌ FALSCH – PowerShell kann keine Unix-Pfade auflösen
cd /home/felix/wowilift_sh

# ❌ FALSCH – Doppelte Backslash-UNC-Pfade funktionieren nicht zuverlässig
cd "\\\\wsl.localhost\\Ubuntu-24.04\\home\\felix\\wowilift_sh"

# ✅ RICHTIG – CWD ist bereits korrekt, Git direkt aufrufen
git status
git add .
git commit -m "..."
```

## Commit-Workflow

### 1. Status prüfen

```powershell
git status
```

### 2. Änderungen prüfen (optional)

```powershell
git diff
git diff --staged
```

### 3. Dateien stagen

```powershell
# Einzelne Dateien
git add wowilift_vertrag/models/mail_activity_plan.py

# Ganzes Modul
git add wowilift_vertrag/

# Alles (mit Vorsicht)
git add -A
```

### 4. Committen

```powershell
git commit -m "feat(modulname): Kurzbeschreibung der Änderung"
```

### 4.5 ⚠️ PFLICHT: Modulversionen prüfen vor dem Push

Vor jedem Push muss sichergestellt werden, dass alle Module mit geänderten
Dateien auch eine angehobene `version` in ihrer `__manifest__.py` haben.
Ohne Versionserhöhung erkennt der Server keine Änderungen und führt kein
`-u <modul>` beim nächsten Start durch.

**Schritt 1 – Geänderte Module ermitteln:**

```powershell
# Alle Toplevel-Verzeichnisse mit Änderungen seit dem letzten Push
git diff --name-only origin/main HEAD | ForEach-Object { ($_ -split '/')[0] } | Sort-Object -Unique
```

Alternativ, wenn der Commit noch nicht existiert:

```powershell
# Staged + unstaged Änderungen
git diff --name-only HEAD | ForEach-Object { ($_ -split '/')[0] } | Sort-Object -Unique
```

**Schritt 2 – Version in `__manifest__.py` prüfen:**

Für jedes geänderte Modul (das ein Odoo-Modul ist, d.h. `__manifest__.py`
besitzt):

```powershell
# Beispiel für ein Modul
wsl -d Ubuntu-24.04 -e bash -lc "grep version /home/felix/wowilift_sh/<modulname>/__manifest__.py"
```

**Schritt 3 – Version anheben falls nötig:**

Das Format ist `19.0.X.Y.Z`. Die letzte oder vorletzte Stelle inkrementieren:

```python
# Vorher:  'version': '19.0.1.2.3',
# Nachher: 'version': '19.0.1.2.4',
```

Nach der Änderung in `__manifest__.py` nochmals stagen und committen:

```powershell
git add <modulname>/__manifest__.py
git commit --amend --no-edit
# ODER als eigenen Commit:
git commit -m "chore(<modulname>): Version auf 19.0.X.Y.Z anheben"
```

> **Ausnahmen:** Reine Dokumentations-Commits (`knowledge_article*.xml`
> in `wowilift_custom/data/`) und README-Änderungen erfordern KEINE
> Versionserhöhung. Bei allen anderen Änderungen (Python, XML-Views,
> Security-CSV, JS) ist die Versionserhöhung **Pflicht**.

### 5. Pushen

```powershell
git push origin main
```

## Commit-Message-Konvention

Format: `<typ>(<modul>): <beschreibung>`

| Typ | Verwendung |
|-----|-----------|
| `feat` | Neues Feature / neue Funktion |
| `fix` | Bugfix |
| `refactor` | Code-Umbau ohne Verhaltensänderung |
| `style` | Formatierung, CSS, keine Logik-Änderung |
| `docs` | Dokumentation |
| `security` | Zugriffsrechte, Record Rules |
| `data` | Stammdaten, Sequenzen, Konfiguration |

Beispiele:
```
feat(vertrag): Verwalterwechsel-Aktivitätsplan automatisch starten
fix(partner): Verwalter-Sync bei Objekten ohne Besitzer
security(vertrag): Zugriffsrechte für Aktivitätspläne erweitern
refactor(invoice): PDF-Generierung in eigene Methode auslagern
```

## Sicherheitsregeln

- **Niemals `--force` pushen** ohne explizite Nutzer-Bestätigung
- **Niemals `git reset --hard`** ohne explizite Nutzer-Bestätigung
- **Vor dem Push:** Immer `git status` und `git diff` prüfen
- **Vor dem Push:** Immer Modulversionen prüfen (Schritt 4.5) – **ohne Versionserhöhung werden Änderungen auf dem Server nicht aktiviert!**
- **Unbekannte Dateien:** Nicht automatisch stagen – zuerst den Nutzer fragen
- **Branch-Wechsel:** Immer sicherstellen, dass keine uncommitteten Änderungen vorliegen
