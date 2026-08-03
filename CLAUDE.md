# CLAUDE.md — adw-skill

## Versionierung & Changelog (Release-Prozess, verbindlich)

Jeder Push nach `main` ist ein Release mit eigener Revisionsnummer (SemVer
0.x): **Änderung am Skill-Verhalten (SKILL.md, references/) → Minor-Bump,
reine Doku-/Fixes → Patch-Bump.** Es gibt keine Versionsdatei — Changelog
und Git-Tags sind die Versionsquelle.

Vor jedem Push:

1. **Changelog nachziehen** — `CHANGELOG.md` (EN) **und** `CHANGELOG.de.md`
   (DE) synchron halten (Keep-a-Changelog-Format), Compare-Link unten
   ergänzen.
2. **Tag setzen** — `git tag vX.Y.Z` auf dem Release-Commit.
3. **Push** — `git push && git push --tags`.

Reines Doku-Repo: kein Lint-/Test-/Codex-Gate nötig.

Historie: Die Versionen bis 0.1.2 wurden rückwirkend aus der Push-Historie
vergeben (`git reflog show origin/main`); ihre Tags zeigen auf die damals
gepushten Stände.
