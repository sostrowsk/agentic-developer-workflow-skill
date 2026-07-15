# adw-skill

Claude-Skill für den [Agentic Developer Workflow (ADW)](https://gitlab.com/ostrowsk/agentic-developer-workflow) —
bringt einer Claude-Instanz bei, Entwicklungs-Issues vollautomatisch durch die
sieben ADW-Phasen zu fahren (Spec → Plan → Build → Integration/E2E → Reviews → Push/CI),
gegen GitLab- oder GitHub-Projekte.

## Installation

```bash
# Als Kopie …
cp -r adw-skill ~/.claude/skills/
# … oder als Symlink (Änderungen wirken sofort)
ln -sn "$(pwd)/adw-skill" ~/.claude/skills/adw-skill
```

Danach triggert der Skill auf Formulierungen wie „lass ADW Issue #42 umsetzen"
oder direkt per `/adw-skill`. Voraussetzung: der ADW-Orchestrator unter
`$ADW_HOME` (Default `~/PycharmProjects/agentic-developer-workflow`).

## Inhalt

- `adw-skill/SKILL.md` — Trigger + Kern-Workflow (Preflight → Config → Dry-Run → Run → approve/resume)
- `adw-skill/scripts/preflight.sh` — Startklar-Check (Werkzeuge, Login, Ziel-Repo-Config)
- `adw-skill/references/` — Config-Schema und Troubleshooting/Eskalations-Referenz
- `adw-skill/assets/config-template.yaml` — Vorlage für `.adw/config.yaml` im Ziel-Repo

Paketieren (erzeugt `adw-skill.skill`, gitignored): `package_skill.py adw-skill .` aus dem skill-creator.
