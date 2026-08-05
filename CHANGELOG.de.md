# Changelog

Alle nennenswerten Änderungen am adw-skill werden in dieser Datei
dokumentiert. Format nach [Keep a Changelog](https://keepachangelog.com/de/1.1.0/);
Versionierung nach [SemVer](https://semver.org/lang/de/) (0.x: Minor =
Änderungen am Skill-Verhalten, Patch = Doku/Fixes).

**Release-Prozess:** Jeder Push nach `main` ist ein Release — er bekommt
einen Eintrag hier und ein Git-Tag `vX.Y.Z` (`git push && git push --tags`).
Es gibt keine separate Versionsdatei; Changelog und Tags sind die
Versionsquelle. Die Versionen bis 0.1.2 wurden rückwirkend aus der
Push-Historie vergeben.

English edition: [CHANGELOG.md](CHANGELOG.md)

## [0.2.2] — Unreleased

### Changed
- Config-Referenz: das optionale Gate-Feld `tdd: true` dokumentiert — es
  markiert die Gates, die im Initial-Build einer Lane den deterministischen
  RED-Beweis liefern (reiner Test-Lauf des Agents, der Orchestrator führt
  genau die markierten Gates aus, mindestens eines muss rot sein, bevor
  implementiert wird). Empfehlung: das Test-Gate markieren; ohne markiertes
  Gate bleibt der Build einstufig. Schema-Beispiel und
  `assets/config-template.yaml` tragen das Flag an den Test-Gates.
- Troubleshooting: neue Eskalations-Zeile „Tests nach reinem Test-Lauf grün
  — RED nicht bestätigt" (Ursache: Die Tests decken das geforderte
  Verhalten nicht ab oder es existiert bereits; Abhilfe: Issue/Spec
  schärfen, neuen Run starten — es gibt keinen Retry-Loop).

## [0.2.1] — 2026-08-03

### Changed
- Config-Referenz: drei praxiserprobte Stolperfallen dokumentiert —
  Poetry-venvs in Lane-Worktrees (venv-Binaries absolut in die Gates
  verdrahten), `staging_job` vs. GitLab-Bridge-/Trigger-Jobs (`staging_job`
  weglassen, der Pipeline-Status deckt es ab), gitignorierte `.env` fehlt im
  Worktree (Projekt muss auf CI-Defaults zurückfallen).
- Troubleshooting: Limits auf Review-Loop-Policy v2 nachgezogen (5
  Review-Runden, absteigende Severity-Schwelle, Findings-Gedächtnis) und die
  vollständige Liste dessen, was in `followups.md` landet. Vervollständigt
  Verweise, auf die SKILL.md 0.2.0 bereits zeigte.

## [0.2.0] — 2026-08-03

### Changed
- Freigabe-Gates in SKILL.md verbindlich gemacht: Am Spec-Gate muss
  `spec-summary.md`, am Plan-Gate `plan-summary.md` gelesen und im Chat
  präsentiert werden — Zusammenfassung + Scope-Review-Ergebnis + klare
  Empfehlung, nie nur „bitte freigeben" (passend zum ADW-Orchestrator
  0.3.0 Dual-Authoring: zwei Entwurfs-Autoren [Opus 4.8 + Codex] plus
  Fable-Best-of-Synthese je Authoring-Phase; Verbrauchshinweis im
  Grenzen-Abschnitt ergänzt).
- SKILL.md-Workflow-Härtung mitgezogen: Preflight-Checkliste ohne Script,
  Issue-Brief-Vorlage mit Scope-Deckel / Deferred-Ventil / Contract-Hinweis,
  `--spec-approval`-Empfehlung, Scope-Review-Checkliste vor jeder Freigabe.
- Dieses Changelog und der Release-Prozess.

## [0.1.2] — 2026-07-15

### Added
- MIT-Lizenz.

## [0.1.1] — 2026-07-15

### Changed
- ADW-Referenzen im README zeigen auf das GitHub-Repo.

## [0.1.0] — 2026-07-15

Erstes Release.

### Added
- adw-skill: Claude-Skill für den Agentic Developer Workflow —
  SKILL.md-Bedienungsanleitung (Preflight, Ziel-Repo-Setup, Dry-Run,
  run/resume/approve, Exit-Code-Handling), Config-Referenz und
  Troubleshooting unter `references/`, paketiertes `adw-skill.skill`.

[0.2.2]: https://github.com/sostrowsk/agentic-developer-workflow-skill/compare/v0.2.1...HEAD
[0.2.1]: https://github.com/sostrowsk/agentic-developer-workflow-skill/compare/v0.2.0...v0.2.1
[0.2.0]: https://github.com/sostrowsk/agentic-developer-workflow-skill/compare/v0.1.2...v0.2.0
[0.1.2]: https://github.com/sostrowsk/agentic-developer-workflow-skill/compare/v0.1.1...v0.1.2
[0.1.1]: https://github.com/sostrowsk/agentic-developer-workflow-skill/compare/v0.1.0...v0.1.1
[0.1.0]: https://github.com/sostrowsk/agentic-developer-workflow-skill/releases/tag/v0.1.0
