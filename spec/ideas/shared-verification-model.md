---
format: https://specscore.md/idea-specification
status: Draft
---

# Idea: Shared verification model — thin ACs, reusable checks, generated summaries (SpecScore ⇄ Rehearse)

**Status:** Draft
**Date:** 2026-07-12
**Owner:** alex
**Promotes To:** —
**Supersedes:** —
**Related Ideas:** [rehearse-evidence-layer](rehearse-evidence-layer.md)

## Problem Statement

Acceptance criteria and scenarios exist in **two divergent forms** across the stack, drifting apart with no contract between them:

- **SpecScore:** ACs are *inline* `### AC:` sections in a feature README — thin "what must be true," bundling requirements; proof delegated to a separate scenario. Reference syntax `feature#ac:slug`.
- **Rehearse (standalone repo):** ACs are *files* `_acs/{slug}.ac.md` carrying Description + Inputs + a Verification block — a heavier, reusable, parameterized unit. Reference syntax `feature/ac-slug`.

Plus two runners (specscore-cli `internal/rehearse` vs the standalone `rehearse` repo, fold-in unresolved). The result: no single source of truth for an AC, two reference syntaxes, and a format that will duplicate as it grows. We need one model that gives **single source of truth, readability, and reusability of ACs** — usable across both a standalone Rehearse and the SpecScore ecosystem.

## Context

### Current stage (honest, 2026-07-12)

- SpecScore's inline `### AC:` is a Stable Doc-Kind, woven into lint (coverage P-001), plans' `Verifies:`, and Studio.
- Rehearse's `_acs/*.ac.md` is its own model; its README markets **reusable, parameterized ACs** ("write once, reference from many flows") as a differentiator.
- Reference model already decided family-wide: **Decision 0010 — references are URLs** (RFC 3986, slug in the case-sensitive path, empty authority = current repo, lint-validated resolution).
- Parent idea [rehearse-evidence-layer](rehearse-evidence-layer.md) established the positioning: **standalone-capable, stack-native**, two layers (a standalone core + an AC-binding differentiation layer), and the founder's line that *"unbound Rehearse is a worse Hurl"* — differentiation lives in the AC binding.

### The reconciliation (with the parent idea)

This is **not** a pivot to "standalone-first." It is the concrete architecture of the parent idea's AC-binding layer. "Standalone product with pluggable AC sources, SpecScore as default" == the parent's "standalone-capable, stack-native." The design below is what makes standalone Rehearse *more than* "a worse Hurl": a composable AC + reusable-check model is real differentiation even unbound.

## Recommended Direction

Un-conflate, thin, and share. Four moves:

1. **Un-conflate Rehearse's AC into two concepts** (borrowing SpecScore's "AC = intent / scenario = proof" separation — its genuinely good idea):
   - **AC** = *thin intent*: id (slug), one-sentence statement, feature URL-ref, status, `applies-to`. No verification inside.
   - **Check** = *reusable, parameterized verification unit* (Inputs + a fenced verification block). This is Rehearse's real differentiator, now **honestly named** instead of smuggled inside "AC."
2. **Scenarios remain the proof** — method-specific flow steps that `Verifies:` a set of ACs and compose shared checks via a new **`### use <check> with <params>`** include primitive. *This primitive is the keystone* — it is what makes "write verification once, reuse across flows" real once verification leaves the AC.
3. **One linkage system** — adopt Decision 0010 wholesale: slug is the id, references are URLs, lint guarantees resolution. Ends the `#ac:` vs `/`-path divergence.
4. **Pluggable AC sources, SpecScore default** — the runner needs only a thin contract `{id, statement, feature-ref, status}`. SpecScore's inline `### AC:` and Rehearse's `_acs/` files both expose it; SpecScore is the default in the ecosystem, swappable elsewhere. **SpecScore cedes the AC + scenario *format* to Rehearse** and keeps owning the "what to build" layer (features, requirements, plans, decisions). Clean dependency: Rehearse independent; SpecScore consumes it.

### Generated summaries are a first-class output, not a nicety

The feature README keeps its `## Acceptance Criteria` section, but it is a **lint-generated, committed summary** projected from the source-of-truth AC files — embedding each AC's **statement** (not just a link), plus which methods/REQs it covers.

This is a materialized read-model over a single source of truth, and it earns its place on **economics**: a human — or an AI agent — reading the feature spec gets the full AC picture **inline, in one read, with zero per-AC tool calls**, while the `_acs/` files remain the authoritative detail retrievable only when genuinely needed. Generated + committed = GitHub-readable *and* drift-free (the `index-entries --fix` pattern already exists). This directly answers "won't file-per-AC hurt readability and cost agents N lookups?" — no, because the summary is denormalized on purpose.

## End State (vision)

The login worked example (email + Google + Facebook): three tiers of ACs — email-only auth ACs, OAuth-family ACs shared by Google+Facebook, and universal post-auth ACs shared by all — where each AC and each shared check is defined **once** (4 shared post-auth ACs referenced 3× each, not 12 copies). Method scenarios differ only in the auth steps; the post-auth tail is a byte-identical `### use` of the same checks. Humans read a rich generated summary in the feature spec; agents author and maintain the underlying files fluently; SpecScore.Studio renders backlinks and coverage automatically. (Full sketch lives in the design conversation that seeded this idea.)

## Roadmap (feature decomposition, once validated)

Promotes to several Features across both repos:

- **Rehearse — reusable check primitive** (`### use <check> with <params>`). *Keystone; build first.*
- **Rehearse — thin AC format** + un-conflation from checks.
- **Rehearse — URL+slug linkage** (adopt 0007/0010 + ref-resolution lint).
- **SpecScore — AC-as-plugin**: cede AC/scenario format; feature `## Acceptance Criteria` becomes a generated summary; coverage lint via URL refs.
- **SpecScore — generated-summary lint rule** (`lint --fix` builds the AC summary table from `_acs/` frontmatter).
- **Studio — hassle-free authoring/rendering** (compute backlinks + coverage; author ACs/checks/scenarios).
- **Migration** — inline `### AC:` → thin AC files across repos. *Cheapest now (small corpus); do last.*

## Alternatives Considered

- **Unify on Rehearse's current rich AC format** — rejected: imposes verification-heavy ACs on SpecScore and loses the intent/proof separation.
- **Federate two formats behind a thin resolver** (each keeps its native format) — viable fallback if un-conflation proves too disruptive; lighter but perpetual dual-format maintenance.
- **Status quo** — rejected: uncontrolled drift, ambiguous canonical source.

## MVP Scope

Prove it end-to-end in **one repo**, before any migration: the `### use` check primitive + thin AC + the generated summary, exercised by the login example (email + one OAuth provider). If that reads well and runs green, the model is validated.

## Not Doing (and Why)

- **The ecosystem-wide migration** — not until the model is validated on the MVP.
- **The domain/brand decision** (rehearse.md vs .ink) — parallel track; downstream of "standalone-capable vs standalone-first," which this design does not need resolved to proceed.
- **Full Studio UX** — MVP is file + CLI; Studio polish follows.

## Key Assumptions to Validate

1. **The check-include primitive is expressive enough** to capture real shared verification (the login post-auth tail) without contortion.
2. **It is genuinely usable.** Founder's own doubt, recorded honestly: file-per-AC + checks may be hard to maintain *by hand*. The bet is that **AI agents + SpecScore.Studio make it hassle-free**; if that bet fails, the model is too heavy for humans and should fall back to federation.
3. **Generated summaries preserve readability** and give agents cheap inline context (the economics claim above).
4. **The reusable check is worth keeping** as a distinct concept — i.e., standalone Rehearse is *not* "a worse Hurl" once it has composable ACs + checks.

## SpecScore Integration

SpecScore cedes AC/scenario *format* ownership to Rehearse (default, swappable plugin); keeps features/requirements/plans/decisions. Feature READMEs carry a generated AC summary; coverage lint follows URL refs. Studio renders it.

## Open Questions

- Does SpecScore keep *any* native AC affordance for zero-dependency authors, or is Rehearse's thin AC always required?
- Un-conflation vs federation — commit to un-conflation, or keep federation as the escape hatch if the maintenance bet fails?
- Root decision to record when validated: *"SpecScore cedes the AC + scenario format to Rehearse; Rehearse is standalone-capable with pluggable AC sources, SpecScore default."* (An ADR, once the MVP validates the direction.)

---
*This document follows the https://specscore.md/idea-specification*
