---
format: https://specscore.md/idea-specification
status: Draft
---

# Idea: Nested scenario suites — Given/When/Then as describe/context/it

**Status:** Draft
**Date:** 2026-07-12
**Owner:** alex
**Promotes To:** —
**Supersedes:** —
**Related Ideas:** [shared-verification-model](shared-verification-model.md)

## Problem Statement

Flat Given/When/Then (Gherkin-style peers) can't express **branching behavior from shared setup**. "Given a Google account: *when* the callback succeeds → these outcomes; *when* auth fails → those outcomes" forces two separate scenarios that each re-declare the Given, duplicating setup and scattering related behavior across files. Authors reach for nesting because it matches how they *think* about behavior — a decision tree rooted in context.

## Context

The founder's instinct: structure scenarios as a tree —

```
## Given an account mapped to a Google identity
### When he completes the OAuth callback
#### Then he is authenticated
#### Then the session is hardened
#### Then he reaches the dashboard
### When authentication fails
#### Then he is not authenticated
#### Then he gets an error message
#### Then he stays on the login page
```

This is exactly the `describe / context / it` model of RSpec/Jest/Mocha — widely preferred over flat Gherkin because it groups shared setup and branches naturally. It is not a novelty; it is a different, well-proven standard.

### The crucial semantic

A test *run* is a single linear execution — two mutually-exclusive `When`s cannot run in one pass. So **a Given with multiple Whens is a *suite*, not a scenario**: each root-to-leaf path (`Given → one When → its Thens`) is an independently-executed **case**. The `## Given` is a `describe`; each `### When`+`#### Then`s is an `it`.

## Recommended Direction

Teach the runner the nested tree:

1. **Parse** `## Given` → `### When` → `#### Then` as a tree (consistent heading levels).
2. **Execute** each `### When` branch as its own case: run the shared `## Given` setup, then that branch's steps/checks.
3. **Report and emit evidence per case** — the success case verifies `session-hardened`; the failure case verifies `error-shown`. Finer-grained evidence is a feature.
4. **Compose with the keystone** — `**Use:** [check]` works under a `#### Then` exactly as under a flat `## Then`; the two are orthogonal (nesting groups cases within a file; checks reuse verification across files).

## End State (vision)

The login example becomes readable *behavior*: one `## Given` per identity, success and failure branches side by side, shared post-auth checks reused via `**Use:**`, ACs referenced (not nested), and the AC-centric view generated. A BA reads the branching behavior; a dev runs it; an agent authors it — one file.

## Alternatives Considered

- **Flat files + a shared setup check** — two files that both `**Use:** [setup]`. Achieves shared setup with the keystone alone (zero new machinery) but loses the grouped, branch-visible readability. Viable fallback / stepping stone.
- **Keep flat Gherkin** — rejected: can't express branching-from-shared-setup without duplication.

## MVP Scope

Nested parse + per-branch execution + per-case reporting for a two-branch example (success/failure), on top of the existing runner. No new block kinds.

## Not Doing (and Why)

- Arbitrary depth beyond Given/When/Then (e.g. nested Whens) — start at three levels; deepen only with a real need.
- Blocking the keystone — the reusable-checks primitive ships first and is agnostic to nesting.

## Key Assumptions to Validate

1. Per-branch execution + per-case evidence is clean to model on the current runner (which today treats a file as one linear scenario).
2. The nested outline reads better *to BAs and devs both* than flat — the core "sellable" bet.
3. Shared `## Given` setup semantics (run once vs per-branch, isolation) are unambiguous.

## SpecScore Integration

Per-case `Verifies:` and evidence flow into Studio as they do per-scenario today, just finer-grained. AC-centric grouping stays a generated view (many-to-many between cases and ACs).

## Open Questions

- Is the `## Given` setup run once and shared, or re-run per branch for isolation? (Leaning: re-run per branch in a fresh workdir, for isolation — matches per-scenario sandboxing today.)
- Do we allow multiple `## Given`s (peer contexts) in one file, or one root Given per file?

---
*This document follows the https://specscore.md/idea-specification*
