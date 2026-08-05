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

## [0.2.2] — Unreleased

### Changed
- Config reference: the optional Gate field `tdd: true` documented — it
  marks the Gates that carry the deterministic RED proof in a lane's
  initial build (test-only agent pass, orchestrator runs exactly the marked
  Gates, at least one must be red before implementation starts).
  Recommendation: mark the test Gate; without a marked Gate the build stays
  single-stage. Schema example and `assets/config-template.yaml` carry the
  flag on the test Gates.
- Troubleshooting: new escalation row "tests green after the test-only pass
  — RED not confirmed" (cause: the tests do not cover the required
  behaviour or it already exists; remedy: sharpen issue/spec, start a new
  run — there is no retry loop).

## [0.2.1] — 2026-08-03

### Changed
- Config reference: three field-tested pitfalls documented — Poetry venvs
  in lane worktrees (wire absolute venv binaries into gates), `staging_job`
  vs. GitLab bridge/trigger jobs (omit `staging_job`, the pipeline status
  covers it), gitignored `.env` missing in worktrees (projects must fall
  back to CI defaults).
- Troubleshooting: limits updated to review-loop policy v2 (5 review
  rounds, descending severity floor, findings memory) and the full list of
  what ends up in `followups.md`. Completes references that SKILL.md 0.2.0
  already pointed to.

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

[0.2.2]: https://github.com/sostrowsk/agentic-developer-workflow-skill/compare/v0.2.1...HEAD
[0.2.1]: https://github.com/sostrowsk/agentic-developer-workflow-skill/compare/v0.2.0...v0.2.1
[0.2.0]: https://github.com/sostrowsk/agentic-developer-workflow-skill/compare/v0.1.2...v0.2.0
[0.1.2]: https://github.com/sostrowsk/agentic-developer-workflow-skill/compare/v0.1.1...v0.1.2
[0.1.1]: https://github.com/sostrowsk/agentic-developer-workflow-skill/compare/v0.1.0...v0.1.1
[0.1.0]: https://github.com/sostrowsk/agentic-developer-workflow-skill/releases/tag/v0.1.0
