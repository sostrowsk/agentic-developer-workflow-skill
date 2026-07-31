---
name: adw-skill
description: Führt Entwicklungs-Issues vollautomatisch mit dem Agentic Developer Workflow (ADW) aus — 7 Phasen von Spec über Build und Reviews bis Push/CI, gegen ein beliebiges Git-Ziel-Repo (GitLab oder GitHub). Verwenden, wenn ein Issue/Feature/Bugfix per ADW umgesetzt werden soll ("lass ADW das Issue umsetzen", "adw run", "starte den Workflow für Issue #42"), ein ADW-Run fortgesetzt oder freigegeben werden soll (resume/approve), ein Run-Status oder eine Eskalation zu analysieren ist, oder ein Ziel-Repo ADW-fähig gemacht werden soll (.adw/config.yaml).
---

# ADW — Agentic Developer Workflow bedienen

ADW ist ein 7-Phasen-Orchestrator (Spec → Plan+Kontrakt → Build → Integration/E2E
→ Codex-Review → finaler Review → Push/CI). Kontrollfluss ist deterministischer
Code; Agenten (Claude-Code-CLI via Agent SDK, Codex-CLI) liefern nur Urteilsvermögen.
Abrechnung läuft über den Claude-Plan (stored-login-only), nie token-by-token.

Der Orchestrator liegt unter `$ADW_HOME` (Default:
`~/PycharmProjects/agentic-developer-workflow`). Alle Kommandos von dort ausführen:

```bash
cd "$ADW_HOME" && uv run adw <kommando>
```

## Workflow

### 1. Preflight (manuell)

Vor dem ersten Lauf gegen ein Ziel-Repo prüfen (es gibt KEIN Preflight-Script):

```bash
for t in uv git codex gh glab; do command -v $t >/dev/null && echo "OK: $t" || echo "FEHLT: $t"; done
ls ~/.claude/.credentials.json          # Claude stored-login
git -C <ziel-repo> remote -v            # Hosting (GitLab/GitHub) erkennen
ls <ziel-repo>/.adw/config.yaml         # Config vorhanden?
```

Alle `FEHLT:`-Punkte beheben, bevor ein Run startet.

### 2. Ziel-Repo ADW-fähig machen (einmalig)

Fehlt `.adw/config.yaml`: `examples/config.yaml` (im `$ADW_HOME`) nach
`<ziel-repo>/.adw/config.yaml` kopieren und an das Projekt anpassen
(base_branch, Gates je Lane, optional e2e/ci). Schema, Regeln und bekannte
Stolperfallen (Poetry-venv in Worktrees, Bridge-Deploy-Jobs):
[references/config.md](references/config.md). Config committen.
Danach als Baseline die Gates einmal von Hand im Haupt-Checkout ausführen —
ein von Anfang an rotes Gate lässt jeden Run scheitern.

### 3. Dry-Run vor dem ersten echten Lauf

Immer zuerst den Trockenlauf fahren — er verifiziert Config, Gates und den
kompletten Kontrollfluss mit Mocks (0 Tokens, kein Netz, kein Push):

```bash
uv run adw run --repo <ziel-repo> --issue "Demo" --dry-run --no-approval
```

Exit 0 = Setup steht. Exit 1 = zuerst `.adw/runs/<run_id>/escalation.md` lesen
(häufig: rote Gates in der Config).

### 4. Issue-Brief verfassen (Scope-Deckel ist Pflicht)

Der Issue-Text ist der EINZIGE Hebel vor der Spec-Phase — der Spec-Agent
erhält ihn wörtlich, und der Spec/Plan-Codex-Loop neigt dazu, Randfälle
aufzublasen, bis der Reviewer zufrieden ist. Ohne expliziten Deckel entsteht
verlässlich Gold-Plating (Erfahrung lexsource 2026-07: aus „Lock freigeben +
neu starten" wurden ~60 Tests, Ownership-Tokens und ein Marker-Subsystem).

Jeder Issue-Brief enthält darum verbindlich:

1. **Aufgabe(n)** — konkret, mit Dateipfaden/Zeilen, wenn bekannt.
2. **Scope-Deckel / Nicht-Ziele** — explizit benennen, was NICHT gebaut wird:
   keine neuen Persistenz-Zustände/Subsysteme ohne realen, nicht anderweitig
   begrenzten Schaden; bestehende Mechanismen (TTLs, Webhooks, Logs) als
   Backstop bevorzugen; Richtwert für die Testzahl nennen.
3. **Deferred-Ventil** — „weitergehende Härtungsideen gehören in einen
   ‚Deferred (bewusst nicht gebaut)'-Abschnitt der Spec, nicht in
   Akzeptanzkriterien." Das gibt dem Spec-Agenten einen legitimen Ort für
   gute Ideen außerhalb des Build-Scopes.
4. **Contract-Hinweis bei Single-Lane-Projekten** — „Contract nur für extern
   beobachtbare Flächen (Routen, Events, Template-Verhalten), keine internen
   Helper-Signaturen."

### 5. Echter Lauf

```bash
uv run adw run --repo <ziel-repo> --issue "Issue-Text ..."       # Text direkt
uv run adw run --repo <ziel-repo> --gitlab-issue <id>            # GitLab via glab
uv run adw run --repo <ziel-repo> --github-issue <nr>            # GitHub via gh
```

Genau EINE Issue-Quelle angeben. Optionen: `--parallel` (FE+BE-Lanes, verlangt
beide Lanes in der Config; verbraucht Plan-Kontingent schneller),
`--no-approval` (Plan-Freigabe überspringen), `--spec-approval` (zusätzlicher
Stopp NACH der Spec, VOR dem Plan — Exit 2, Freigabe per `adw approve`),
`--base-branch <name>` (Override, wird ab Run-Start gepinnt).

`--spec-approval` standardmäßig setzen, wenn das Issue nicht-trivial ist oder
Scope-Drift droht: Eine überdimensionierte Spec ist am Plan-Gate bereits in
Plan+Contract einbetoniert — am Spec-Gate kostet das Trimmen nur einen neuen
Run über eine einzelne Datei. Am Spec-Gate gilt dieselbe Freigabe-Pflicht wie
am Plan-Gate (siehe Schritt 6): erst `.adw/runs/<run_id>/spec-summary.md`
lesen — die Synthese-Zusammenfassung ist die Entscheidungsgrundlage —, dann
das Scope-Review gegen `spec.md` statt `plan.md` fahren und Zusammenfassung +
Review-Ergebnis + Empfehlung im Chat präsentieren.

### 6. Exit-Code auswerten und handeln

| Exit | Bedeutung | Aktion |
|---|---|---|
| 0 | `done` — Branch gepusht, CI + Staging grün | Ergebnis melden, MR/PR vorschlagen |
| 2 | `awaiting_approval` — Spec- oder Plan-Freigabe-Pause | Summary des Gates lesen (`spec-summary.md` bzw. `plan-summary.md`), Scope-Review (siehe unten), Ergebnis im Chat präsentieren; nach OK des Nutzers: `uv run adw approve <run_id> --repo <ziel-repo>` |
| 1 | Eskalation oder Fehler | Siehe [references/troubleshooting.md](references/troubleshooting.md) |

Die run_id steht in der CLI-Ausgabe; Übersicht: `uv run adw status --repo <ziel-repo>`.

**Freigabe-Pflicht an beiden Gates:** Der Run-Ordner ist gitignored, der Nutzer
sieht die Artefakte also nicht von selbst. Darum immer in dieser Reihenfolge:

1. **Summary lesen** — am Spec-Gate `.adw/runs/<run_id>/spec-summary.md`, am
   Plan-Gate `.adw/runs/<run_id>/plan-summary.md`. Die Fable-Synthese schreibt
   dort die Entscheidungsgrundlage: was und warum, Kernentscheidungen,
   Scope-Grenzen und Deferred-Punkte, Herkunft (welcher Entwurf lieferte was,
   wo widersprachen sie sich), offene Fragen.
2. **Scope-Review fahren** (Checkliste unten) — gegen `spec.md` am Spec-Gate,
   gegen `plan.md` + `contract.yaml` am Plan-Gate.
3. **Im Chat präsentieren** — Zusammenfassung des Artefakts (aus der Summary,
   nicht das Artefakt in voller Länge), Ergebnis des Scope-Reviews und eine
   klare Empfehlung (freigeben / Rückfrage / neuer Run mit geschärftem Brief).
   Nie nur „bitte freigeben" oder einen Dateipfad zum Selberlesen.

**Scope-Review vor jeder Freigabe (Pflicht):** Artefakte NICHT einfach
durchreichen, sondern gegen den Issue-Brief prüfen:

- **Mechanismen-Diff:** Welche Zustände, Helper, Subsysteme hat die Spec
  erfunden, die das Issue nicht verlangt? Jeden davon benennen und dem Nutzer
  als Scope-Ausweitung ausweisen (mit Einschätzung: berechtigt / Gold-Plating).
- **Testzahl-Verhältnis:** Testliste des Plans vs. Richtwert im Issue-Brief —
  bei deutlicher Überschreitung (> ~2×) Rückfrage statt Freigabe-Empfehlung.
- **Scope-Deckel-Abgleich:** Verstößt der Plan gegen ein explizites Nicht-Ziel
  des Issue-Briefs? Dann nicht freigeben, sondern neuen Run mit
  nachgeschärftem Brief empfehlen.
- **Echte Funde hervorheben:** Aufgeblähte Pläne enthalten oft 1–2 wertvolle
  Entdeckungen (z. B. „Stripe folgt keinem 301"). Diese explizit benennen und
  bei einem Neustart in den Issue-Brief übernehmen.

Ein pausierter, nicht freigegebener Run wird durch einen Neustart einfach
stehen gelassen (kein Cancel nötig); Artefakte von Hand zu trimmen und dann zu
approven ist ein unsupported Pfad (Artefakte wären nicht mehr Codex-reviewt).

### 7. Fortsetzen

- **Nach Crash oder Plan-Limit-Abbruch** („Agent-Lauf abgebrochen … adw resume"):
  `uv run adw resume <run_id> --repo <ziel-repo>` — setzt exakt am Checkpoint
  fort. Bei erschöpftem Claude-Plan-Fenster erst den Limit-Reset abwarten.
- **Nach Approval-Pause:** `approve`, nicht `resume` (resume pausiert erneut).
- **Eskalierte Runs** (`phase: escalated`) sind endgültig: Ursache aus dem
  Report klären, dann NEUEN Run starten.

## Wichtige Grenzen

- Spec-/Plan-Freigabe (Exit 2) nie eigenmächtig per `--no-approval`/`approve`
  umgehen — die Freigabe ist Sache des Nutzers, außer er hat sie explizit
  delegiert.
- Niemals manuell in `.adw/runs/<id>/trees/…`-Worktrees committen oder
  Branches wechseln — der Orchestrator erkennt Fremd-Commits und eskaliert.
- Erster echter Lauf gegen ein neues Ziel-Repo nur nach grünem Dry-Run.
- Ein echter Lauf verbraucht Claude-Plan-Kontingent (Fable 5 + Opus 4.8 +
  Sonnet 5) und Codex-Abo-Kontingent — vor großen `--parallel`-Läufen den
  Nutzer auf den Verbrauch hinweisen. Spec- und Plan-Phase laufen dabei mit
  je zwei unabhängigen Autoren (Opus 4.8 + Codex) plus einer Fable-Synthese
  statt mit einem Autor — die Authoring-Phasen kosten entsprechend mehr
  Kontingent (Claude und Codex) und Laufzeit als eine reine Ein-Autor-Phase.

## Eskalationen und Fehlerbilder

Für Exit 1, Eskalations-Reports, typische Fehlermeldungen und deren Behebung:
[references/troubleshooting.md](references/troubleshooting.md) lesen.
