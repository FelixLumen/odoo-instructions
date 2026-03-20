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
- **Unbekannte Dateien:** Nicht automatisch stagen – zuerst den Nutzer fragen
- **Branch-Wechsel:** Immer sicherstellen, dass keine uncommitteten Änderungen vorliegen
