# Data/service boundary

## The service contract

A service module (in the feature's service layer) is a plain, framework-agnostic function — no query-library import. It takes typed input, calls the shared transport wrapper (global, domain-agnostic infra), validates/parses the response via the [schema layer](forms-validation.md), and returns typed data.

On failure, a service throws a typed, structured error — never a raw fetch `Response` or generic `Error` — so callers up the stack can discriminate on it. The exact error shape is an implementation detail for whoever writes the first service. There is no sanctioned alternative to throwing: a service that instead catches its own failure and returns a result union isn't a recognized pattern here.

## The query boundary: components never fetch

Components and hooks never call the transport layer, or a service function, directly. Every service call — including a one-off, fire-and-forget action — is mediated by a query-caching/mediation layer, so loading/error/success state is always tracked the same way and observable in one place.

Two flavors of that mediation exist:

1. **Direct** — the feature's hooks layer calls the query/mutation primitive with the service function as the fetcher, for data a single component (or a small, local group) consumes.
2. **Shared** — a derived, shared cache primitive backing query state consumed across multiple components. This only fixes that this flavor exists as the other valid consumption point; its mechanics belong to [state-management layering](state-management.md).

No third path: if a component needs server data and neither flavor fits yet, that's a gap in the query layer to fill, not a reason to fetch directly.

## Wiring a service into an organism

An organism never imports a service module — per [component tiering](component-tiering.md), it accepts an injected data contract instead. The concrete wiring: the feature's hooks layer calls the service through the query-mediation layer, and the resolved data/callbacks are passed into the organism as props, or supplied via a context provider the feature owns (the DI seam named in [feature-folder organization](feature-folders.md)'s skeleton).

So only the hooks layer and the DI seam ever import a service module directly. Organisms, a feature's root surface, and pages never do — they receive data, not the means to fetch it.

## Note on server-rendered frameworks

A query-caching library may support a prefetch-and-hydrate bridge: a server-rendered component can prefetch and hand the result to client components via a hydration boundary, so client-side reads see pre-fetched data instead of re-fetching on mount. It only matters when a server-rendered page's data also needs to keep living in client-side cache (for later refetch/mutation) — a plain server-rendered page with no client-side ownership afterward doesn't need it, and a direct server-side fetch is simpler. Which of these a given route needs is a framework-specific topic ([page composition & data loading](page-composition.md)).
