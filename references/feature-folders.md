# Feature-folder organization

## The boundary: global vs. feature

Global (infra-level) code holds only domain-agnostic infrastructure — code with no awareness of any specific feature's data shape, endpoint, or form fields (an HTTP client, generic pagination/filter state, generic UI primitives; see [component tiering](component-tiering.md) for where those primitives live). Anything that knows a specific feature's shape — a DTO, an endpoint, a form's fields — lives inside that feature's own folder. Features compose global primitives; global code never imports from a feature.

## Internal layering within a feature

A feature is layered from its external contract down to its public surface, in one fixed dependency direction. Each layer's content is owned by its own topic — this only fixes the skeleton, the folder names, and the order:

1. **contract/types** — the feature's contract: the shape of its data and its service interface, independent of any concrete implementation.
2. **DI seam** — lets the feature's public surface receive a concrete implementation of the contract without importing it directly. Concrete wiring: [data/service boundary](data-service-boundary.md).
3. **service** — the concrete implementation of the contract (API calls, etc.). Content and rules: [data/service boundary](data-service-boundary.md).
4. **schema** — validation schema and inferred types for DTOs/form input. Content and rules: [forms & schema validation](forms-validation.md).
5. **store** — the feature's local state. Content and rules: [state-management layering](state-management.md).
6. **hooks** — glue: wires the DI seam, store, and schema together into what the feature's components actually consume.
7. **lib** — feature-local pure helpers with no reuse outside the feature.
8. **root surface** — the feature's public UI surface, and the only thing other code should import from it (plus anything deliberately exported per the cross-feature rule below).

Not every feature needs every layer — create a layer's folder the first time the feature actually needs it.

## Cross-feature state: graduate, don't cross-import

A feature's internals are private except for what its root exports. When two or more features genuinely need the same state, that state does not stay owned by one feature while the others import it directly — it graduates to the nearest common ancestor that composes those features. In practice that's usually the page/route rendering all of them; if three or more features need it, a shared grouping one level above them may be the nearer common ancestor instead.

This is not the same as promoting it to the global store: global store stays reserved for domain-agnostic infrastructure only (see [the boundary](#the-boundary-global-vs-feature) above). Shared-but-domain-specific state lives at the composition root that owns it, not in global code.

## Thin pages and routes

A page/route is a thin composition root: it wires the feature(s) it needs — including providing any graduated shared state down to those features — and owns route-level layout (breadcrumbs, tabs, route-specific shell chrome). Real business logic and domain state live in features, not in the page.

Exception: trivial one-off route glue with no reusable feature shape (e.g. an OAuth callback that redirects and sets a token) may hold real logic inline. This is a narrow, named exception, not a general license for pages to grow logic.

This fixes the underlying principle — thin composition root, real logic lives in features; how it's realized under a specific framework's server/client mechanics belongs to [page composition & data loading](page-composition.md).
