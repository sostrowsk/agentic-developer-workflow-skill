# Changelog

All notable changes to the adw-skill are documented in this file.
The format follows [Keep a Changelog](https://keepachangelog.com/en/1.1.0/);
versioning follows [SemVer](https://semver.org/) (0.x: minor = skill-behavior
changes, patch = docs/fixes).

**Release process:** every push to `main` is a release — it gets an entry
here and a git tag `vX.Y.Z` (`git push && git push --tags`). There is no
separate version file; the changelog and the tags are the version source.
Versions up to 0.1.2 were assigned retroactively from the push history.

Deutsche Fassung: [CHANGELOG.de.md](CHANGELOG.de.md)

## [0.2.0] — 2026-08-03

### Changed
- Approval gates made binding in SKILL.md: at the spec gate
  `spec-summary.md`, at the plan gate `plan-summary.md` must be read and
  presented in chat — summary + scope-review result + clear
  recommendation, never just "please approve" (matches ADW orchestrator
  0.3.0 dual authoring: two draft authors [Opus 4.8 + Codex] plus Fable
  best-of synthesis per authoring phase; consumption note added to the
  limits section).
- SKILL.md workflow hardening carried along: preflight checklist without a
  script, issue-brief template with scope ceiling / deferred valve /
  contract note, `--spec-approval` recommendation, scope-review checklist
  before every approval.
- This changelog and the release process.

## [0.1.2] — 2026-07-15

### Added
- MIT license.

## [0.1.1] — 2026-07-15

### Changed
- ADW references in the README point to the GitHub repo.

## [0.1.0] — 2026-07-15

Initial release.

### Added
- adw-skill: Claude skill for the Agentic Developer Workflow — SKILL.md
  operating guide (preflight, target-repo setup, dry run, run/resume/
  approve, exit-code handling), config reference and troubleshooting under
  `references/`, packaged `adw-skill.skill`.

[0.2.0]: https://github.com/sostrowsk/agentic-developer-workflow-skill/compare/v0.1.2...v0.2.0
[0.1.2]: https://github.com/sostrowsk/agentic-developer-workflow-skill/compare/v0.1.1...v0.1.2
[0.1.1]: https://github.com/sostrowsk/agentic-developer-workflow-skill/compare/v0.1.0...v0.1.1
[0.1.0]: https://github.com/sostrowsk/agentic-developer-workflow-skill/releases/tag/v0.1.0
