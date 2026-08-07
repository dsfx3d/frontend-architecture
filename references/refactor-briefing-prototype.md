# PROTOTYPE — throwaway, answers issue #42

Not part of the spec. Three candidate shapes for what actually lands in a
Refactor-mode map's `## Destination` and `## Notes` once Audit's findings —
handed over as a new, Refactor-mode-only pre-step-1 briefing (mechanism
decided in [#41](https://github.com/dsfx3d/frontend-architecture/issues/41))
— are folded into wayfinder's charting. Scenario: `/frontend-architecture
refactor this` on a project that already has `docs/frontend-architecture/`;
Audit's full six-topic scan runs first and finds two Conflicts (reusing
`audit.md`'s own "Example turn" findings, same as
[#34](https://github.com/dsfx3d/frontend-architecture/issues/34)'s
prototype did), both accepted (`y`) at write-back:

    Audit complete — 6 topics scanned, 2 with new Conflicts to review:
    component-tiering, state-management (4 clean: feature-folders,
    data-service-boundary, page-composition, forms-validation).

- likely: `Button`-family components take no children/composition slot yet
  sit under `molecules/`, not `atoms/`, in 3 places — conflicts with this
  skill's tiering pattern. (src/features/checkout/ui/molecules/SubmitButton.tsx,
  src/features/auth/ui/molecules/LoginButton.tsx,
  src/shared/ui/molecules/IconButton.tsx)
- borderline: `useDraftStore` holds in-progress form state that may not
  need to survive a refresh, which would argue for local component state
  instead of a shared store. (src/features/onboarding/state/draftStore.ts)

This closing summary plus the two accepted findings is what the new
pre-step-1 briefing hands to wayfinder's "Name the destination" step.

---

## Variant A — findings dumped verbatim into Destination

```markdown
## Destination

Audit findings driving this refactor:
- likely: `Button`-family components take no children/composition slot yet
  sit under `molecules/`, not `atoms/`, in 3 places — conflicts with this
  skill's tiering pattern. (src/features/checkout/ui/molecules/SubmitButton.tsx,
  src/features/auth/ui/molecules/LoginButton.tsx,
  src/shared/ui/molecules/IconButton.tsx)
- borderline: `useDraftStore` holds in-progress form state that may not
  need to survive a refresh, conflicting with this skill's refresh-survival
  promotion test. (src/features/onboarding/state/draftStore.ts)

Resolve both: retier the Button family correctly, and decide where
draft-form state should live.

## Notes

[Plan mode's existing enrichment, unchanged — gist-by-default guidance +
the same two Conflicts repeated again in the callout]
```

The destination-naming grill just transcribes the briefing instead of
synthesizing it. Two problems: it blows past wayfinder's own "Destination
... one or two lines" discipline, and it pastes the *exact same* Conflict
bullets `Notes` is about to record anyway via Plan mode's unchanged
enrichment pass ([#41](https://github.com/dsfx3d/frontend-architecture/issues/41)) —
a paste that goes stale the moment `docs/frontend-architecture/` is refreshed
by a later audit run, since nothing keeps Destination's copy in sync with
the ledger's.

---

## Variant B — Destination synthesized, Notes unchanged from Plan mode

```markdown
## Destination

Consolidate the divergent `Button`-family component tiering and settle
whether draft-form state belongs in a shared store or local component
state.

## Notes

Domain: <project>'s frontend architecture, refactor via frontend-architecture.
Consult /grilling and /domain-modeling for tickets by default.

frontend-architecture guidance (references/index.md, captured at chart time
2026-08-07):
- Component tiering — placing a new shared component: ui→atom→molecules→
  organisms, importing downward only. [component-tiering.md]
- State-management layering — server/local/shared tiers, promoted only once
  something outside the owner needs it; refresh-survival is the promotion
  test. [state-management.md]
- (gist only, not judged relevant to this destination) Feature-folder
  organization [feature-folders.md], Data/service boundary
  [data-service-boundary.md], Page composition [page-composition.md],
  Forms & schema validation [forms-validation.md]

Project reality (docs/frontend-architecture/, last full scan 2026-08-07 @
<sha>):
- Recorded Conflict — component-tiering.md: likely, `Button`-family
  components take no children/composition slot yet sit under `molecules/`,
  not `atoms/`, in 3 places. (src/features/checkout/ui/molecules/SubmitButton.tsx,
  src/features/auth/ui/molecules/LoginButton.tsx,
  src/shared/ui/molecules/IconButton.tsx)
- Recorded Conflict — state-management.md: borderline, `useDraftStore`
  holds in-progress form state that may not need to survive a refresh.
  (src/features/onboarding/state/draftStore.ts)
```

Destination is a proper synthesized line — the grilling session's normal
job, just informed by the briefing rather than transcribing it. `Notes`
is exactly [#34](https://github.com/dsfx3d/frontend-architecture/issues/34)'s
already-locked shape, unmodified, reading the same freshly-written
Conflicts the briefing was built from. No new mechanism, no duplication.

The gap: nothing on the map distinguishes *these two Conflicts* — the
whole reason this map exists — from the ordinary background-guidance
callout Plan mode already attaches to any planning map that happens to
have a relevant recorded Conflict. A reader landing on this map can't
tell "audit surfaced this, that's why we're here" from "this was just
sitting in the ledger" without reading Destination's prose closely and
inferring the connection themselves.

---

## Variant C — Destination synthesized, Notes gets one provenance line (recommended)

Identical to B, with exactly one addition: a single gist line, reusing
Audit's own closing-summary shape verbatim, as the first line of `Notes`.

```markdown
## Destination

Consolidate the divergent `Button`-family component tiering and settle
whether draft-form state belongs in a shared store or local component
state.

## Notes

Seeded from an Audit run (2026-08-07): 6 topics scanned, 2 flagged —
component-tiering, state-management (4 clean). Both findings accepted
during write-back; see the Conflicts callouts below — this map exists to
resolve them.

Domain: <project>'s frontend architecture, refactor via frontend-architecture.
Consult /grilling and /domain-modeling for tickets by default.

frontend-architecture guidance (references/index.md, captured at chart time
2026-08-07):
- Component tiering — placing a new shared component: ui→atom→molecules→
  organisms, importing downward only. [component-tiering.md]
- State-management layering — server/local/shared tiers, promoted only once
  something outside the owner needs it; refresh-survival is the promotion
  test. [state-management.md]
- (gist only, not judged relevant to this destination) Feature-folder
  organization [feature-folders.md], Data/service boundary
  [data-service-boundary.md], Page composition [page-composition.md],
  Forms & schema validation [forms-validation.md]

Project reality (docs/frontend-architecture/, last full scan 2026-08-07 @
<sha>):
- Recorded Conflict — component-tiering.md: likely, `Button`-family
  components take no children/composition slot yet sit under `molecules/`,
  not `atoms/`, in 3 places. (src/features/checkout/ui/molecules/SubmitButton.tsx,
  src/features/auth/ui/molecules/LoginButton.tsx,
  src/shared/ui/molecules/IconButton.tsx)
- Recorded Conflict — state-management.md: borderline, `useDraftStore`
  holds in-progress form state that may not need to survive a refresh.
  (src/features/onboarding/state/draftStore.ts)
```

---

## Why C

- **Answers "why does this map exist" without re-deriving it.** Refactor
  mode's Conflicts/Exceptions aren't incidental background the way they
  can be for an ordinary Plan-mode map — they're the seed. B's mechanism
  makes them look exactly like Plan mode's optional background callout,
  erasing a real distinction. One gist line fixes that: a session opening
  the map — chart-time or ticket-resolution weeks later — gets the origin
  story immediately, no inference required.
- **Costs exactly one line, and it's a line that already exists.** The
  provenance line reuses Audit's own closing-summary phrasing verbatim
  (`audit.md`'s "Example turn" shape) — no new format to design or keep
  in sync, just the sentence Audit already produces, relocated.
- **Doesn't duplicate the ledger.** The provenance line is a *tally and a
  pointer* ("see the Conflicts callouts below"), not a second copy of the
  findings — those still live exactly once, in the Conflicts callout B
  already carries forward unchanged from
  [#34](https://github.com/dsfx3d/frontend-architecture/issues/34)'s Notes
  shape. Consistent with wayfinder's index-not-store principle: a decision
  (or, here, a finding) lives in exactly one place, everything else gists
  and links.
- **Destination stays disciplined.** Same as B, Destination is one to two
  lines of synthesized prose — the grilling session's normal output,
  informed by the briefing rather than transcribing it. A's verbatim dump
  is rejected for both blowing past that discipline and going stale
  against the ledger it copied from.
