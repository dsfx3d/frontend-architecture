# Page composition & data loading

## A page may fetch — "thin" means no logic, not no data

A server-rendered route's entry point is a server-rendered component by default and can await its feature's service call directly — that's the framework's idiomatic mechanism, not a violation of the "thin composition root" rule (see [feature-folder organization](feature-folders.md)'s thin-pages-and-routes rule). "Thin" means no business logic or transformation: a page acquires data via a direct fetch and immediately hands the raw result down as props. Any branching, grouping, or shaping of that data belongs to the feature's hooks/lib layers ([feature-folder organization](feature-folders.md)), not the page.

## Auth guard: centralized in middleware

Per-route auth checks (verify a session, redirect if absent) live in a centralized request-middleware layer, not inline in each page. Middleware runs before a route renders and redirects before any page code executes, so a page can assume it's already authenticated — its own fetch calls are 100% about its feature's data, nothing else.

## Pending/error states: framework-native, not manual

A route segment that needs a loading or error state gets the framework's native convention for it (e.g. a sibling file that auto-wraps the segment in a suspense/loading boundary, and another that auto-wraps in an error boundary). A page's data-fetch should simply throw on failure and let the native error boundary catch it — no manual `isLoading` flag or inline try/catch for acquisition errors. This keeps a page down to pure happy-path composition: fetch, then render.

## Server/client boundary: push client-interactivity to leaves

Server-rendered components are the default all the way down the tree — free of shipped JS, where the framework supports this. A feature's root component (what the page renders) stays server-rendered too. Only the specific leaf molecules/organisms that actually need interactivity (event handlers, local state, browser APIs) get marked as client components. Data flows down as plain props through server components until it reaches the first interactive leaf; that leaf is where the client boundary starts, not any concrete tier from [component tiering](component-tiering.md) in particular.

## A query-mediation layer is opt-in per page, not automatic

The [data/service boundary](data-service-boundary.md) topic fixes that components never fetch directly — they're always mediated by a query-caching/mediation layer. That rule governs component-initiated fetching; it does not extend to a route's own initial server-side fetch, which isn't component-initiated at all.

- A page with no follow-on client interactivity (no refetch, no mutate, no polling) stays a plain fetch -> props hand-off. The query-mediation layer never enters — there's nothing for its cache to do.
- A page whose data is later owned by a client component that needs live query capabilities uses a prefetch+hydrate bridge: the server-rendered component prefetches, dehydrates the cache, and the client component's query call (same key) reads the pre-seeded cache instead of double-fetching.

Whether a page needs the bridge is decided per page, not mandated globally — the query cache only matters once a client component genuinely needs to independently refetch, invalidate, or mutate that data after first render.

## URL params a page's fetch depends on

A page's server-side params only reflect a URL-sync-library-managed param when that param is written to trigger a real navigation/round-trip (a URL-sync library may default to a shallow update that skips this). If a page's fetch reads a URL param, that param must be declared to trigger the round-trip on the client side, or the page keeps rendering stale data after the URL changes. See [state-management layering](state-management.md)'s "URL as a state channel".
