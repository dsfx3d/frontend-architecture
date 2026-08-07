# PROTOTYPE — throwaway, answers issue #34

Not part of the spec. Three candidate shapes for what plan mode's enrichment
pass (mechanism decided in #33) actually writes into a wayfinder map's
`## Notes` field. Scenario: `/frontend-architecture plan this feature` for
"checkout wizard — shipping → payment → review, shared cart state" in a
project that already has `docs/frontend-architecture/` with one recorded
Conflict in `state-management.md` (reusing audit.md's own example finding).

---

## Variant A — full snapshot, everything pasted

```markdown
## Notes

Domain: checkout feature in <project>, planned via frontend-architecture.

frontend-architecture guidance (captured 2026-08-07, plan-mode invocation):
- Component tiering: placing a new shared component — four tiers, ui→atom→
  molecules→organisms, importing downward only; features and pages sit
  above them. [full rule text, ~10 lines pasted from component-tiering.md]
- Feature-folder organization: [full rule text, ~15 lines pasted from
  feature-folders.md]
- Data/service boundary: [full rule text, ~12 lines pasted]
- State-management layering: [full rule text, ~15 lines pasted]
- Page composition & data loading: [full rule text, ~12 lines pasted]
- Forms & schema validation: [full rule text, ~15 lines pasted]
```

Every topic, gist *and* full reference body, pasted verbatim into Notes
regardless of relevance to the destination. No links back to source.

---

## Variant B — gist-by-default, link for relevant, Conflicts called out (recommended)

```markdown
## Notes

Domain: checkout feature (shipping → payment → review wizard, shared cart
state) in <project>. Consult /grilling and /domain-modeling for tickets by
default.

frontend-architecture guidance (references/index.md, captured at chart time
2026-08-07 — zoom a link below for the live rule if a ticket needs more):
- Component tiering — placing a new shared component: ui→atom→molecules→
  organisms, importing downward only. [component-tiering.md]
- Feature-folder organization — one fixed layering order (contract→DI seam→
  service→schema→store→hooks→lib→root surface); cross-feature state
  graduates to the nearest common composition root. [feature-folders.md]
- State-management layering — server/local/shared tiers, promoted only once
  something outside the owner needs it; refresh-survival is the promotion
  test. [state-management.md]
- Forms & schema validation — schema folder holds DTO + form-input schemas;
  a hook owns the form instance, fields stay flattened-prop-only.
  [forms-validation.md]
- (gist only, not judged relevant to this destination) Data/service
  boundary [data-service-boundary.md], Page composition & data loading
  [page-composition.md]

Project reality (docs/frontend-architecture/, last full scan 2026-08-02 @
a1b2c3d):
- Recorded Conflict — state-management.md: *borderline*, `useDraftStore`
  holds in-progress form state that may not need to survive a refresh,
  conflicting with this skill's refresh-survival promotion test
  (src/features/onboarding/state/draftStore.ts). Worth weighing when this
  map decides where the wizard's step state lives.
- No other Conflicts/Exceptions recorded for the topics above.
```

Every topic gets its one-line gist (cheap, always included, per #33's
"gist by default"). Topics judged relevant to the destination additionally
get a slightly fuller distillation *and* a link to the live reference file
— not the file's full body pasted in. `docs/frontend-architecture/` folds
in the same way: gist-level Stack-facts/Conventions stay implicit (a ticket
zooms the doc if it needs them), but a recorded Conflict/Exception for a
*relevant* topic gets pulled up into an explicit callout, because that's
exactly the "does this decision fight our own codebase" fact a grilling
session needs already in view, not buried in a linked doc.

---

## Variant C — no snapshot, standing reconsult instruction only

```markdown
## Notes

Domain: checkout feature (shipping → payment → review wizard, shared cart
state) in <project>.

Every ticket session on this map: before grilling, re-invoke
frontend-architecture's plan-mode enrichment step live (re-read
references/index.md + relevant reference files + docs/frontend-architecture/
Conflicts/Exceptions) — do not rely on a chart-time snapshot, since
frontend-architecture's own guidance may have moved on since this map was
charted.
```

Notes carries no content of its own, only a standing instruction. Every one
of the map's ticket sessions (each a separate agent session, per wayfinder's
one-ticket-per-session rule) re-runs the full gather step before it can even
start grilling.

---

## Why B

- **Matches wayfinder's own idiom already on this map.** Map #31's own
  Notes names skills to consult and standing decisions by reference, not
  pasted content; Decisions-so-far gists a closed ticket and links out for
  detail. B applies that exact pattern to frontend-architecture content
  instead of inventing a new one.
- **The link *is* the reconsult path — no separate mechanism needed.**
  Wayfinder's "work through the map" step 3 already says to zoom any
  related content on demand. A gisted line + a live link lets a ticket
  session pull the current rule if it suspects drift, without paying a
  live-reconsult cost on every session (C) or bloating Notes with content
  that's stale the moment frontend-architecture itself iterates (A).
- **Conflicts/Exceptions are surfaced, not buried, but only where relevant.**
  Pulling the *whole* per-topic ledger in (Stack facts + Conventions +
  Conflicts + Exceptions, all six topics) would re-create the audit report
  inside Notes. Surfacing just the Conflict/Exception bullets for topics
  already judged relevant keeps it targeted at what a planning decision
  actually needs to weigh.
