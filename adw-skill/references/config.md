# .adw/config.yaml — Referenz

Die komplette projektspezifische Konfiguration lebt im ZIEL-Repo unter
`.adw/config.yaml`. Vorlage: `assets/config-template.yaml`. ADW validiert
fail-fast: fehlende Datei, unbekannte Keys, doppelte Keys, Lane ohne Gates
oder Gate ohne Timeout brechen den Run sofort mit klarer Meldung ab.

## Schema

```yaml
base_branch: staging           # Pflicht: Fork-/Diff-Basis der Lanes

lanes:                         # Pflicht: mindestens eine Lane (backend|frontend)
  backend:
    gates:                     # Pflicht je Lane: >= 1 Gate
      - {name: black,  cmd: "black --check .",      timeout: 120}
      - {name: isort,  cmd: "isort --check-only .", timeout: 120}
      # tdd: true (optional, Default false) = RED-Beweis-Gate
      - {name: pytest, cmd: "pytest -x -q",         timeout: 1800, tdd: true}
  frontend:                    # optional; --parallel verlangt BEIDE Lanes
    gates:
      - {name: eslint, cmd: "npm run lint",         timeout: 300}

e2e:                           # optional; läuft nur mit --parallel
  cmd: "npx playwright test"
  timeout: 1800

ci:                            # optional; Defaults siehe unten
  provider: gitlab             # gitlab | github; ohne Angabe Auto-Erkennung
  poll_interval: 60            # Sekunden zwischen CI-Polls (Default 60)
  timeout: 2700                # CI-Warte-Budget in Sekunden (Default 2700)
  staging_job: deploy-staging  # Job, der zusätzlich grün sein muss (optional)
```

## Regeln und Stolperfallen

- **Gates = Qualitätsgrenze.** ADW committet Lane-Ergebnisse nur nach grünen
  Gates. Was kein Gate prüft, prüft in Phase 3 niemand — mindestens
  Linter + Testsuite konfigurieren. Reihenfolge = Ausführungsreihenfolge,
  das erste rote Gate stoppt (fail fast).
- **Timeouts sind Pflicht** (Sekunden, > 0) — jedes Gate-Kommando läuft mit
  hartem Subprocess-Timeout.
- **`tdd: true` markiert die RED-Beweis-Gates** (optional, Default false).
  Ist in einer Lane mindestens ein Gate markiert, läuft ihr Initial-Build
  zweistufig: Der Build-Agent schreibt zuerst NUR die Tests, danach führt der
  Orchestrator genau die markierten Gates aus. Mindestens eines muss rot sein
  — erst dieser Beweis gibt die Implementierung frei (dieselbe Agent-Session,
  der RED-Check verbraucht keine Gate-Iteration). Sind alle grün, eskaliert
  der Run. **Empfehlung: genau das Test-Gate markieren** — Linter/Formatter
  beweisen an reinen Tests nichts. Ohne markiertes Gate bleibt der Build
  einstufig wie bisher; Fix-Läufe (Review/E2E) durchlaufen die RED-Stufe nie.
  Die RED-Tests dürfen danach nicht mehr verschwinden: Gates gelten nur als
  grün, solange die Dateien des Beweises noch existieren.
- **Ports:** Gates bekommen `BACKEND_PORT`/`FRONTEND_PORT` als Env-Variablen
  (deterministisch je Run/Lane) — für Dev-Server in Gate-/E2E-Kommandos nutzen.
- **`ci.provider`:** Ohne Angabe erkennt ADW das Hosting am Hostnamen der
  origin-URL (`github*` → GitHub, `gitlab*` → GitLab, auch Self-Hosted mit
  sprechendem Hostnamen). Bei anderem Hostnamen (z. B. `code.firma.de`) ist
  der Key Pflicht, sonst eskaliert Phase 7.
- **`staging_job`:** Name des CI-Jobs, der zusätzlich zur grünen Pipeline
  erfolgreich sein muss (GitLab: Pipeline-Job; GitHub: Job-Name in einem
  Workflow-Run). Weglassen, wenn es keinen Staging-Deploy gibt.
- **Env der Gates ist eine Whitelist** (PATH, HOME, LANG, VIRTUAL_ENV, …) —
  Secrets/API-Keys aus der Umgebung erreichen Gate-Kommandos nicht. Braucht
  ein Gate Umgebungswerte, gehören sie in Projekt-Dateien (z. B. `.env`, die
  das Kommando selbst lädt), nicht in die Shell-Umgebung.
- Die Config ist für alle ADW-Agenten unveränderlich — Änderungen macht nur
  der Mensch, per normalem Commit. `base_branch`-Änderungen wirken erst auf
  NEUE Runs (laufende Runs pinnen ihre Basis beim Start).
- **Poetry-Projekte: venv-Binaries direkt verdrahten.** Lanes laufen in
  git-Worktrees; dort legt `poetry run` ein NEUES, leeres venv an (Poetry
  keyed venvs am Projektpfad), und alle Gates scheitern an fehlenden
  Dependencies. Stattdessen den venv-Pfad ermitteln
  (`poetry env info -p` im Haupt-Checkout) und die Binaries absolut in die
  Gate-Kommandos schreiben, z. B.
  `/home/<user>/.cache/pypoetry/virtualenvs/<env>/bin/python manage.py test`.
- **`staging_job` sieht keine Bridge-Jobs.** ADW prüft den Job über den
  GitLab-`/jobs`-Endpoint; Trigger-/Bridge-Jobs (Downstream-Deploys, z. B.
  docker-host-Trigger) erscheinen dort nicht — ein als Bridge deployter
  Staging-Job führt zu „Staging-Job existiert nicht in der Pipeline". In dem
  Fall `staging_job` weglassen; der Pipeline-Gesamtstatus deckt den
  Bridge-Status mit ab.
- **Gitignorierte `.env` fehlt im Worktree.** Gates laufen ohne die lokale
  `.env` — das Projekt muss (wie in CI) auf saubere Defaults zurückfallen
  (z. B. SQLite). Tut es das nicht, gehört ein committetes Test-Env-File ins
  Repo, nicht die `.env` in den Worktree.
