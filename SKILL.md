---
name: frontend-architecture
description: Patterns for structuring a component-based frontend app — component tiering, feature-folder organization, data/service boundaries, state-management layering, page composition & data loading, and forms/schema validation. Use when adding or placing a new shared component, structuring a new feature folder, wiring a data or service call, choosing where state should live, composing a page or route, or wiring a form to validation.
license: Complete terms in LICENSE.txt
---

# Frontend architecture

Stack-agnostic placement rules for a component-based frontend: where a component, a data call, a piece of state, or a form validator belongs. Six topics, cross-referenced; each is a full reference file under `references/`. The current topic registry — name, trigger phrase, gist, reference link, and scan parameters — lives at [references/index.md](references/index.md).

## Project-initialization

Before placing new work under any of these six topics, check for `docs/frontend-architecture/` at the current package root. If missing, initialization runs silently once, scanning all six topics into per-topic docs and an index — informing every topic below with this project's actual stack and conventions, without ever overriding this skill's own patterns. Trigger detail, scan mechanics, staleness, and the dependency gate on `grilling`/`domain-modeling`: [init.md](references/init.md).

These six topics place *new* work — a new component, a new call, a new form, a new page. Default mode is strictly additive: it never proposes or performs a refactor of your existing code, even where project-initialization scanning (see [init.md](references/init.md)) finds it diverging from these patterns. Divergence gets recorded and flagged for you to weigh, not acted on. A non-default **Audit mode** (below) can re-surface that divergence on demand across your existing code without waiting for new placement work — it's still read-only, only reporting, never fixing. A non-default **Refactor mode** (below) relaxes this boundary one step further: it composes Audit mode's scan with Plan mode's charting into a decisions-only map, then makes a one-time offer to execute the resolved plan once the map's frontier is exhausted. A future migration mode may relax the fix boundary further still — not designed here.

## Audit mode

On an explicit, unambiguous request to assess existing code against these patterns — e.g. "audit this project," "check compliance" — not on an ordinary placement request, the agent scans all six topics (or a named subset) against a given path (default: package root) and reports where the code diverges. Read-only: it never proposes or applies a fix, only flags it for you to weigh, same as the divergence project-initialization records in passing. Findings refresh the same per-topic `Conflicts`/`Exceptions` sections project-initialization already maintains, via the same diff-proposal flow — audit is a way to re-run that scan on demand, not a separate report. Requires `docs/frontend-architecture/` to exist, silently running project-initialization first if it doesn't. Trigger phrasing, scope axes, scan mechanics, confidence hedging, and output shape: [audit.md](references/audit.md).

## Plan mode

On an explicit, unambiguous request to plan an effort that also names frontend-architecture or one of its topics/patterns — e.g. `/frontend-architecture plan this feature` — not on a request that's merely frontend-domain-shaped without naming the skill, the agent wraps `wayfinder`'s map-charting step: right after the destination is named, it gathers this skill's current guidance from [references/index.md](references/index.md) — every topic's gist, plus a fuller distillation and reference link for whichever topic(s) are judged relevant to the destination — and injects it into the resulting map's `Notes`, so every ticket on that map gets grilled with this guidance already in view. Doesn't require `docs/frontend-architecture/` to exist — it enriches with it when present, and surfaces a one-time heads-up when it's absent, rather than nudging you through project-initialization mid-charting. Requires the `wayfinder`, `grilling`, and `domain-modeling` skills; full bail-out if any are missing.

> Plan mode only engages when frontend-architecture is explicitly named alongside a planning request. An ordinary `/wayfinder` invocation with no mention of frontend-architecture proceeds as wayfinder's own standalone map-charting — this skill never intercepts it.

Trigger phrasing, the enrichment pass, Notes shape, the no-init heads-up, scope, and the dependency gate: [plan.md](references/plan.md).

## Refactor mode

On an explicit request that both asks to plan a refactor and names frontend-architecture or one of its topics — e.g. `` `/frontend-architecture refactor this` `` — not on a request that's merely refactor-shaped without naming the skill, the agent composes Audit mode's diagnostic scan with Plan mode's `wayfinder`-charting rather than reinventing either: it runs Audit's full scan against a given path (default: package root), hands the resulting Conflicts/Exceptions as seed context into `wayfinder`'s destination-naming and Notes, then lets a normal breadth-first grilling pass decide the real ticket set — grouping, prioritizing, or dropping findings, never a mechanical one-ticket-per-finding dump. The resulting map stays decisions-only, same as any `wayfinder` map, but relaxes the fix boundary one step further than audit mode ever does: once the map's frontier is exhausted, refactor mode makes a one-time offer to execute the resolved plan with `/implement` (preferred) or `/tdd` (fallback). Requires the `wayfinder`, `grilling`, and `domain-modeling` skills plus a configured issue tracker (`docs/agents/issue-tracker.md`); full bail-out if any are missing.

> Refactor mode only engages when frontend-architecture is explicitly named alongside a request to plan a refactor. A bare `/wayfinder` invocation or a bare "refactor this" request never silently triggers it.

Trigger phrasing, the dependency gate, scope, the diagnose-phase findings handoff, map seed shape, the execution offer, output shape, and drift handling: [refactor.md](references/refactor.md).

## Placing a new shared component

Placement is decided by reuse/coupling scope and composition depth — never by statefulness. Four tiers, `ui` -> `atom` -> `molecules` -> `organisms`, importing downward only; features and pages sit above them. Full placement criterion, the two classes of UI state that justify it, and the app-shell exception: [component-tiering.md](references/component-tiering.md).

## Structuring a new feature folder

Global code holds only domain-agnostic infrastructure; anything that knows a feature's shape lives in that feature's own folder, layered in one fixed order (contract -> DI seam -> service -> schema -> store -> hooks -> lib -> root surface). Cross-feature state graduates to the nearest common composition root, never to global code or a cross-import. Full boundary, layering skeleton, and thin-pages rule: [feature-folders.md](references/feature-folders.md).

## Wiring a data or service call

A service is a plain, framework-agnostic function that validates its response and throws a typed error on failure. Components and hooks never call it directly — every call is mediated by a query-caching layer, and an organism receives data via an injected contract rather than importing a service. Full contract, query-boundary, and organism-wiring rules: [data-service-boundary.md](references/data-service-boundary.md).

## Choosing where state should live

Three tiers — server state (query cache only), local UI state (the default), and shared client state (promoted only once something outside the owner needs it) — placed at the nearest common ancestor of what shares it. State that must survive a refresh or be bookmarkable syncs to the URL instead. Full tiers, promotion rule, and URL-composition rules: [state-management.md](references/state-management.md).

## Composing a page or route

A page/route may fetch directly — "thin" means no business logic, not no data — with auth centralized in middleware and loading/error states handled by the framework's native boundaries. Client interactivity pushes to the leaves; a query-mediation layer is opt-in per page, only once a client component needs to independently refetch or mutate. Full fetch, boundary, and staleness-policy rules: [page-composition.md](references/page-composition.md).

## Wiring a form to validation

A feature's schema folder holds two unrelated kinds of schema — data-boundary DTOs and form-input schemas — reusable but not required to share. A hook owns the form instance and wires the schema through a resolver; presentational field components stay flattened-prop-only, with no awareness of the form library. Full wiring, error-surfacing, and presentational-component rules: [forms-validation.md](references/forms-validation.md).
