---
name: frontend-architecture
description: Patterns for structuring a component-based frontend app — component tiering, feature-folder organization, data/service boundaries, state-management layering, page composition & data loading, and forms/schema validation. Use when adding or placing a new shared component, structuring a new feature folder, wiring a data or service call, choosing where state should live, composing a page or route, or wiring a form to validation.
license: Complete terms in LICENSE.txt
---

# Frontend architecture

Stack-agnostic placement rules for a component-based frontend: where a component, a data call, a piece of state, or a form validator belongs. Six topics, cross-referenced; each is a full reference file under `references/`.

| Topic | Use when… | Reference |
|---|---|---|
| Component tiering | placing a new shared component | [component-tiering.md](references/component-tiering.md) |
| Feature-folder organization | structuring a new feature folder | [feature-folders.md](references/feature-folders.md) |
| Data/service boundary | wiring a data or service call | [data-service-boundary.md](references/data-service-boundary.md) |
| State-management layering | choosing where state should live | [state-management.md](references/state-management.md) |
| Page composition & data loading | composing a page or route | [page-composition.md](references/page-composition.md) |
| Forms & schema validation | wiring a form to validation | [forms-validation.md](references/forms-validation.md) |

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
