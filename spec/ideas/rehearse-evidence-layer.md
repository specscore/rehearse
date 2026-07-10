---
format: https://specscore.md/idea-specification
status: Draft
---

# Idea: Rehearse v1 — the acceptance-evidence layer (standalone-capable, stack-native)

**Status:** Draft
**Date:** 2026-07-10
**Owner:** alex
**Promotes To:** —
**Supersedes:** —
**Related Ideas:** —

## Problem Statement

How might Rehearse become the deterministic proof layer for acceptance criteria — valuable alone, differentiated inside the SpecScore stack?

## Context

### Current stage (honest, 2026-07-10)

- **Code:** prototype. Tags v0.1.0–v0.2.0; 10 of 24 Go files are tests; markdown scenarios with executable Bash/Python/SQL/Starlark blocks run via a standalone binary. Dormant since 2026-06-12.
- **Distribution:** none — git tags only; no brew/scoop/winget, no install docs. Module path is `github.com/synchestra-io/rehearse` (orphaned by the Synchestra parking, 2026-07-09).
- **Adoption:** zero consumers across ~190 local repos (2026-07 ecosystem review).
- **Accidental validation:** the SpecScore Studio Phase-0 build (2026-07-10, specscore-cli `spec/features/cli/studio/index/_tests/`) produced **11 agent-authored executable markdown scenarios** — `Verifies:` AC identity in frontmatter + fenced Bash asserting exit codes/output against a real binary, all passing, authored without being told to "use Rehearse." The *shape* (scenario-as-file, colocated with the spec, AC-addressable, deterministic) emerged naturally; the Rehearse *runner* was not needed. This is the load-bearing observation: the value is the **binding**, not the runner.
- **Founder framing (verbatim, 2026-07-10):** imagined as "markdown scripting for deterministic testing," stuck as "a format for AI-agent verification," interested in Postman-like deterministic HTTP checks. Also: Rehearse "was imagined as a product that can stay on its own" — the standalone-vs-stack question must be answered deliberately, not drifted into.

### Prior art

**Hurl** (plain-text HTTP request/assert files) is the closest neighbor and is excellent — competing with it head-on is a lost war. **Postman/Newman**: right assertion vocabulary, wrong artifact model (JSON blobs, GUI-first, git-hostile). **Gauge/Cucumber**: markdown/gherkin binding died of glue-code maintenance — Rehearse's fenced-executable approach avoids glue and must keep doing so.

## Recommended Direction

**Reposition Rehearse as the acceptance-evidence layer: standalone-capable, stack-native** (the dalgo/inGitDB layering pattern). Two layers, deliberately:

1. **Core (standalone product, small and honest):** the scenario format + a thin runner. A scenario is a markdown file with fenced step blocks: `bash` (exists today, the escape hatch), and new **declarative check blocks** — `http` first (request + asserted status/headers/JSONPath, variables + captures for Postman-style chaining), later `sql` and `file` asserts. For HTTP, **embed Hurl syntax verbatim and delegate execution to the hurl binary when present** (bundle-or-detect); do NOT build an HTTP client — the "don't reinvent the driver" discipline. This layer runs in any repo with zero SpecScore knowledge. Standalone value: git-native, review-able, agent-authorable executable checks.
2. **Stack integrations (the differentiation and the strategic slot):** inside SpecScore-managed repos, scenarios bind to acceptance criteria (`Verifies: <feature>#ac:<slug>` frontmatter — the convention the 11 Studio scenarios already use), live in `spec/features/<slug>/_tests/`, run via **`specscore rehearse run`** (folded into specscore-cli — instantly rides the existing brew/scoop/winget channels and ends the module-path orphanhood), and emit **per-AC pass/fail results as `verified-behavior` facts** into SpecScore Studio's evidence ladder (the top confidence tier of the Studio design, `specstudio-skills/spec/research/studio-design-2026-07/03-entity-and-evidence-model.md`). Rehearse becomes the *producer of the stack's highest-grade evidence* — a slot nothing else fills, completing the loop: SpecScore specifies → `Verifies:` trailers trace → Rehearse proves → Studio serves the proof.

**Why coupling is intentional** (founder asked): (a) differentiation exists only via the AC binding — unbound Rehearse is a worse Hurl; (b) solo-founder GTM economics — the stack shares one funnel, a fifth standalone dev-tool brand repeats the under-marketed-tools pattern the ecosystem review flagged; (c) the need is asymmetric — the world has test runners, the stack lacks an evidence producer. The two-layer design keeps the standalone door open cheaply without splitting focus.

## End State (vision)

A team (or agent fleet) treats every acceptance criterion as something that can be **proven on demand**: `specscore rehearse run` executes the repo's scenarios in CI and locally; Studio's entity pages show per-AC green/amber evidence chips sourced from the latest runs; a failing scenario is a contradiction item, not a forgotten doc; and scenario authorship is predominantly done by implementation agents as part of landing each task (the Studio-build pattern, institutionalized). Standalone users run the same scenarios with the bare `rehearse` core and get value with no stack knowledge — some later discover the binding and convert into stack adopters.

## Roadmap to v1

- **v0.3 — the repositioning release:** runner folded into specscore-cli (`specscore rehearse run` + kept thin standalone binary built from the same package); `http` check block via Hurl embed; scenario discovery in `_tests/`; per-scenario/per-AC pass-fail report (human + JSON). Migrate the 11 Studio scenarios as the reference corpus. *Success gate: the Studio corpus runs green via the new runner in specscore-cli CI.*
- **v0.4 — evidence emission:** JSON results ingested by `studio index` as `verified-behavior` facts (adapter #5); `sql` + `file` assert blocks; captures/variables complete. *Gate: Studio `facts --class verified-behavior` returns real rows for cli/studio/index ACs.*
- **v0.5 — authoring ergonomics:** `specscore rehearse new <ac-id>` scaffolds a scenario from an AC's Given/When/Then; specstudio:implement instructs subagents to author scenarios via it (making last night's accidental pattern the paved road). *Gate: a full specify→implement pipeline run produces passing scenarios with zero hand-authoring.*
- **v1.0 — declared stable:** format spec published (rehearse.ink becomes truthful); ≥3 in-house repos with scenario suites in CI; standalone quickstart README. *Success criteria for v1 overall:* (1) every new Feature in specscore-cli/specstudio ships with runnable scenarios; (2) Studio evidence ladder's top tier is populated by Rehearse in the Sneat workspace; (3) one external (non-founder) repo adopts the standalone core.

## Alternatives Considered

- **Double down on standalone test-framework positioning:** loses to Hurl/pytest on their turf; no distribution; already empirically stalled.
- **Kill Rehearse, use bare bash scenarios forever:** the Studio corpus shows bash works — but without a runner there is no per-AC reporting, no evidence emission, no declarative HTTP, and every repo reinvents discovery/reporting conventions.
- **Adopt Hurl outright, no Rehearse:** loses the AC binding, the markdown colocated-with-spec artifact, and non-HTTP block types; Hurl becomes a dependency of the design anyway (as the embedded engine) — strictly dominated by the two-layer plan.

## MVP Scope

v0.3 as above, timeboxed to one focused week of agent-days: fold-in, Hurl-embedded http block, discovery + report, Studio corpus green. Nothing else.

## Not Doing (and Why)

- Own HTTP client implementation — Hurl embed instead; battle-tested engine for free.
- GUI / collection manager — git-native files are the artifact; Postman's model is the anti-pattern here.
- Browser/UI automation blocks — different beast (playwright exists); revisit only on real demand.
- Python/SQL/Starlark block *expansion* before v0.4 — bash + http cover the corpus; keep the surface small.
- Standalone marketing push before v1.0 — rehearse.ink stays modest until the format is stable (marketing-ahead-of-substance was a flagged credibility risk).

## Key Assumptions to Validate

| Tier | Assumption | How to validate |
|------|------------|-----------------|
| Must-be-true | Hurl syntax can be embedded + delegated cleanly (binary detect/bundle) | v0.3 spike, first week |
| Must-be-true | Per-AC scenario discovery/reporting fits the existing `_tests/` convention without breaking lint | run against the 11-scenario Studio corpus |
| Should-be-true | Implementation agents author scenarios unprompted when the paved road exists | measure in the next 2 specify→implement pipeline runs |
| Should-be-true | Studio ingestion of results as verified-behavior facts is a thin adapter | v0.4; reuse the INGR/JSON patterns from studio index |
| Might-be-true | Standalone core attracts external users organically | watch stars/installs post-v1; no push before that |

## SpecScore Integration

- **New Features this would create:** `cli/rehearse/run`, `cli/rehearse/new`, `studio` results adapter (in specscore-cli); format spec feature in this repo.
- **Existing Features affected:** `cli/studio/index` (adapter #5 for results ingestion); specstudio:implement (scenario-authoring instruction).
- **Dependencies:** Hurl (embedded/delegated); specscore-cli distribution channels.

## Open Questions

- Does the thin standalone `rehearse` binary survive long-term, or does the core live only as a Go package consumed by specscore-cli (standalone = `specscore rehearse` with no spec tree)? Decide at v0.4 from actual standalone demand.
- Module/repo home after fold-in: keep this repo as the format-spec home with the package extracted, or move wholesale into specscore-cli? (Repo-naming standard consultation needed.)

**Addendum (founder, 2026-07-10):** v0.3 scope grew two ways during specify: (1) SQL/DTQL/GraphQL blocks join http (GraphQL compiles onto the Hurl engine; DTQL runs via dalgo against SQLite stores incl. Studio fact stores); (2) the **context bag** is first-class in v0.3 — scenario-scoped variables with {{name}} interpolation across ALL block kinds and per-kind capture directives (Postman-style chaining generalized beyond HTTP: http→sql→bash flows). Spec: specscore-cli spec/features/cli/rehearse/run.

**v0.3 SHIPPED (2026-07-10T06:52Z):** specscore-cli main — `specscore rehearse run` with all five block kinds (bash, hurl-delegated HTTP, sql, dtql via real dalgo, graphql-on-hurl), first-class context bag (per-block-class consumption, four capture routes), human+JSON reports, 21-scenario self-hosting corpus green locally AND in CI (Rehearse corpus job, hurl 8.0.1). Six tasks, six commits with Verifies: trailers, coverage gate 100% throughout. Next: v0.4 evidence emission into studio index.

**v0.4 SHIPPED (2026-07-10T08:00Z):** specscore-cli main (ab90df6..8080107) — `rehearse run --report-out` persists a provenance envelope (runner_version, git_sha, git_dirty, started_at) to `.specscore/rehearse/latest.json`; fifth studio adapter `rehearse` ingests it as `verified-behavior` facts (verified-by + has-verification-status per scenario–AC, observed_at = run time, failures included); success gate green in CI: `studio facts --class verified-behavior` returns real rows for cli/studio/index ACs. Decided: no standalone binary (`specscore rehearse` with no spec tree IS standalone mode). Parked to v0.5: `file` assert block, `rehearse new` scaffolding. Next: v0.5 authoring ergonomics.
