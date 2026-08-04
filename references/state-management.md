# State-management layering

## The three tiers

1. Server state — anything sourced from a service/API call. Owned exclusively by the [query-caching/mediation layer](data-service-boundary.md)'s cache. Nothing in this tier is duplicated into another store.
2. Local UI state — state that never needs to be read or reacted to outside the one component that owns it. Plain framework-native component state. This is the default for any new piece of state — it only leaves this tier when something outside the owner needs it.
3. Shared client state — client-only state (never server-sourced) that's read or reacted to by more than its owning component. A shared-atom-style primitive.

Statefulness alone never picks the tier — a component holding local state and a component holding a shared atom can look identical from the outside. The question is always what kind of data this is, and who else needs it, matching [component tiering](component-tiering.md)'s generic-UI-state vs. feature/domain-state split: tier 2 is local UI state, tier 3 covers both shared UI state and feature/domain state once more than the owner needs to see it, and tier 1 is exclusively feature/domain state sourced from a service.

## Promoting local UI state to shared client state

State starts in tier 2. It's promoted to a shared-state primitive the moment a sibling, ancestor, or a different subtree needs to read or react to it — not because the state "feels complex," and not pre-emptively for state that might be shared later.

## Placing a shared-client-state atom

A shared-state unit lives as close as possible to its owner, at the nearest common ancestor of everything that shares it:

- Shared by components within one feature -> that feature's own state layer.
- Shared across features -> the composition root that renders all of them, per [feature-folder organization](feature-folders.md)'s already-fixed graduation rule (nearest common ancestor, not the global store).
- Generic, key-parameterized primitives with no domain awareness and reused across unrelated features (e.g. a generic filter/sort/pagination atom) -> global state layer.

## Deriving from server state

Shared client state never copies server data out of the query cache — the query cache (tier 1) stays the single source of truth. When a shared-state unit needs to compose server data with other client-state units (e.g. a derived, reactive view), it adapts the query cache rather than mirroring a snapshot of it.

## URL as a state channel

State that must survive a refresh or be bookmarkable/shareable (search text, sort order, page/page-size, "which modal/drawer is open") syncs to the URL rather than living only in memory. State that's transient (a one-off selection) or persisted-but-non-shareable does not.

A meta-framework that owns its own client-side router for route transitions can desync from a URL-sync library that operates on the browser's native history API directly instead of going through the router — pick (or confirm) a URL-sync mechanism that integrates with the router rather than bypassing it.

URL-synced state is its own channel, deliberately outside the shared-state tier — read and written directly in whatever component needs it. There's no shared-state unit and no nearest-common-ancestor placement question: the URL is already the shared location, so any component can read the live value.

## Composing URL state with shared state or server state

Default: compose at the component level. Read the URL value and the shared-state unit (or server-state query) separately, and combine them in whichever component or custom hook needs both. Nothing gets copied into a new store, so there's no bridge unit, no sync effect, and no staleness risk to design around. A query whose key depends on a URL param (e.g. an "active tab" that selects which query runs) follows the same default: read the param, pass it straight into the query's key at the call site. The query cache already makes the result shared across every component that queries the same key, so a shared-state adapter isn't needed just to "share" a URL-parameterized query.

Escape hatch: a shared-state unit that mirrors a URL value into itself via an effect-driven sync (the URL stays the source of truth) is sanctioned only when another shared-state unit's own derivation needs the URL value — i.e. the composition has to happen inside the shared-state graph because more than one unit derives from the same URL param, with no single component to compose it at.

A URL-sync library may default to updating the URL without a server round-trip (a "shallow" update) — meaning a server-rendered page's own params prop does not update, so a server-side read of that param sees a stale value until a real navigation happens. Escape hatch: any URL param a server-rendered page's data-fetch depends on must be declared to trigger a real navigation/round-trip so the param propagates and the page re-fetches. This interacts with [page composition & data loading](page-composition.md).

Any such non-shallow param defines its parser/schema once and reuses it on both sides (client and server), rather than parsing it independently server-side — parsing it twice defeats the reason the round-trip exists: keeping client and server in sync for that param. The parser definition is a feature-local pure helper, not a data-boundary or form-input schema (see [forms & schema validation](forms-validation.md)) — it coerces a string into a typed value with a default; it doesn't validate/reject.

## Persisted, non-shareable local prefs

State that should survive a refresh but isn't meant to be shared or bookmarked (theme, sidebar collapsed/expanded) uses a storage-backed shared-state primitive (e.g. a `localStorage`-backed atom), which has no router entanglement to flag, unlike the URL channel.
