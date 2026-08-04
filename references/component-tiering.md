# Component tiering

## The tiers

1. `ui` — vendor/generated primitives. Generic, unbranded, no app logic. Never hand-authored; regenerated/added via the vendor's own tooling.
2. `atom` — local, hand-authored leaves. Zero composition (no children, no sub-components), no app-domain awareness. A pure display/formatting component is the canonical example.
3. `molecules` — small reusable compositions of `ui`/`atom` (or an external design-system package, if one is ever adopted). May own generic app-level UI state but have no awareness of any domain/feature concern.
4. `organisms` — composites of molecules/atoms/primitives that additionally define their own internal structure (hooks, context, store, types as needed) and expose a data-integration contract rather than fetching or importing domain data themselves. Reusable across features while staying data-agnostic.

Above these, `features/*` (see [feature-folder organization](feature-folders.md)) compose organisms — occasionally a molecule directly — into a single-purpose unit, and pages/routes compose features as thin composition roots. Downward-import direction: `ui` / `atom` -> `molecules` -> `organisms` -> `features` -> `pages`.

## Placement criterion

Placement is decided by reuse/coupling scope and composition depth, in that order:

- Raw vendor/generated primitive with no custom composition? -> `ui`.
- Leaf with zero composition and no domain awareness? -> `atom`.
- Composes primitives into a small reusable unit with no awareness of any domain service or feature? -> `molecules`.
- Composes molecules/atoms/primitives into a larger unit that also defines its own internal structure, and exposes a data-integration contract instead of owning concrete domain data? -> `organisms`.

Statefulness is not the discriminator. A molecule may own state (a theme toggle, an open/closed flag, local form-field state) and still be a molecule — the question is never "does it hold state," it's "what class of state is it and what does it depend on to get it" (full state model: [state-management layering](state-management.md)).

Two classes of state, only enough to justify tier placement:

- Generic/app-level UI state — theme, open/closed, hover, local form-field value. Concerns the component's own presentation, not the business. Freely ownable by `molecules`/`organisms`.
- Feature/domain state — anything sourced from a service, API, or business rule. Never owned by `ui`/`atom`/`molecules`/`organisms` — only `features` (and above) own it.

## Organisms and data

An organism that needs domain data does not import a concrete feature's service or fetch logic. It accepts its data/service dependency as an injected contract — a context, a prop, or a callback supplied by the caller (typically a feature) — so the organism stays reusable across features while remaining data-agnostic. Organism defines the contract, feature fills it; the concrete wiring pattern belongs to [data/service boundary](data-service-boundary.md).

## Exception: app-shell organisms

Global layout chrome — top-level navigation, app-wide drawers/shells — lives under `organisms` by the same folder convention, but is allowed to import concrete feature code directly (e.g. to host a specific feature's content in a slot). This is a narrow, named exception for components whose entire purpose is hosting/chroming features, not a general license for organisms to reach into features. Reusable, non-chrome organisms (a data grid, a file uploader, a dropdown) still hold the rule above without exception.
