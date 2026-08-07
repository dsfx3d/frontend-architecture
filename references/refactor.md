# Refactor mode

## Trigger

Refactor mode fires only on a request that is *both* (a) an ask to plan a refactor and (b) explicit naming of frontend-architecture or one of its topics/patterns — never inferred from a request that's merely refactor-shaped ("refactor this" alone does not trigger it), and never silently intercepting a bare `/wayfinder` invocation with no mention of frontend-architecture. Mirrors [Plan mode](plan.md#trigger)'s trigger philosophy one level up: planning intent must still be explicit and skill-named, refactor intent is no exception.

`` `/frontend-architecture refactor this` `` is the canonical example — a direct mirror of Plan mode's `` `/frontend-architecture plan this feature` ``, playing the same role as Audit's `"audit this project"`: a preferred, unambiguous form, not an exclusive requirement. Equivalent natural-language phrasing that explicitly names frontend-architecture is also valid.

`## Refactor mode` sits in [SKILL.md](../SKILL.md) right after `## Plan mode` and before the six placement topics — the same slot Audit mode and Plan mode themselves took when added, reading naturally in that order since Refactor composes Audit then Plan.

## Dependency gate

Two tiers, not one combined gate — they fail differently and at different times.

### Hard, chart-time gate

Fires at the very top of Refactor mode's invocation, before Audit's scan starts — earlier than where Plan mode's own gate would naturally fire during charting, so a doomed run doesn't burn an Audit scan first. Checks four things:

- `wayfinder` skill present
- `grilling` skill present
- `domain-modeling` skill present
- `docs/agents/issue-tracker.md` present (i.e. `/setup-matt-pocock-skills` has been run)

The first three reuse [Plan mode](plan.md#dependency-gate)'s exact skill-lookup mechanism verbatim (the same 5 lookup paths, project-then-user level) — not reinvented. `/setup-matt-pocock-skills`'s output is a *different* fact than skill-folder presence (it configures the issue tracker/labels/domain docs, doesn't install skills), so it's checked separately rather than used as a proxy for the others.

**On failure**: full bail-out — no scan, no map, no partial anything. Shown once per session (in-memory flag, nothing written to disk), pinpointing exactly which of the four are missing (three render shapes — skill(s)-only, tracker-only, both — never listing all four regardless, a deviation from [init.md](init.md#dependency-gate)'s list-both-regardless style that Plan mode already established):

> "Refactor mode requires the `wayfinder`, `grilling`, and `domain-modeling` skills, plus a configured issue tracker (`docs/agents/issue-tracker.md`). Missing: `<missing skill(s), if any>`, not found under `.agents/skills/`, `.claude/skills/`, `.codex/skills/`, `.opencode/skills/`, or `.agent/skills/` (project or user level)`<; and/or>` tracker not configured — run `/setup-matt-pocock-skills` first`<if applicable>`. Resolve these, then retry — this doesn't affect frontend-architecture's ordinary placement guidance, project-initialization, or audit mode, which all work the same without them."

Audit's own `docs/frontend-architecture/` leg is untouched by this gate — it proceeds exactly as [audit.md](audit.md) already specifies (self-healing via silent auto-init if missing), and by the time it runs, the hard gate has already guaranteed `grilling`/`domain-modeling` are present, so its own internal init-gate trivially passes.

### Soft, deferred check

`/implement` (preferred) then `/tdd` (fallback) — live-checked only at the moment the [end-of-map execution offer](#execution-offer) would fire, potentially sessions or days after charting started, once the map's frontier is exhausted. Not part of the hard gate: failure here doesn't block charting, doesn't block the map, doesn't invalidate anything already resolved. It only skips that one offer:

> "Map complete — no tickets left to resolve. Execution offer skipped: neither `/implement` nor `/tdd` found under `.agents/skills/`, `.claude/skills/`, `.codex/skills/`, `.opencode/skills/`, or `.agent/skills/` (project or user level). Install one to enable execution, or work the resolved plan manually — the map itself stands as-is."

The preference order is `/implement` first, `/tdd` fallback, since `/tdd` may be deprecated. The check is presence-based, live-checked at offer time against the same 5 skill-lookup paths — it self-adjusts automatically if either skill's folder disappears, no spec change needed if `/tdd` is later removed. No need to distinguish which of the two was actually tried first in the skip message — "neither found" is sufficient.

**Why two tiers, not one**: the hard gate covers what's needed to produce the map itself — this mode's actual deliverable. The soft check only backs an optional, one-time bonus offer at the very end; gating charting on it would block the real deliverable behind a feature that might not even matter yet.

## Scope

Refactor mode has **exactly one** scope axis — path — not the two independent axes [Audit mode](audit.md#scope) has.

- **Path**: inherited from Audit verbatim. An explicit path narrows the diagnose-phase scan (e.g. "refactor the checkout feature" → scoped to `checkout/`); defaults to package root; never picked up from ambient/conversational context.
- **Topic-set does not exist as a narrowable concept here.** The diagnose phase always scans every topic currently registered in [index.md](index.md), in full, regardless of phrasing — the count tracks the registry as it grows, never hardcoded. This diverges from Audit deliberately: Refactor's diagnose-then-plan composition depends on the grilling pass (below) to decide the real ticket set from the *full* finding set — pre-filtering to one topic before grilling starts would risk starving that pass of findings that matter once a human looks at the whole picture.
- **Monorepo/package-root handling** reuses project-init's [package/monorepo detection](init.md#packagemonorepo-detection) verbatim, same reuse-not-redesign pattern as [Audit mode](audit.md#scope) and [Plan mode](plan.md#scope). No new decision — Refactor's diagnose phase is an unmodified Audit run by construction, so this falls out automatically.
- **A named topic in the request** (e.g. "refactor the state-management pattern in checkout") is silently ignored for scope purposes — it neither narrows nor emphasizes anything mechanically. The full registry still scans; any prioritization of that topic's findings happens naturally during the grilling pass, not as a second, fuzzier scope axis bolted onto the diagnose phase.

## Diagnose phase and findings handoff

Refactor mode invokes Audit mode exactly as [audit.md](audit.md) already specifies it — full default-scope scan, including its normal sequential per-topic `Apply this update? [y/n/edit]` write-back into `docs/frontend-architecture/<topic>.md`. No bypass, no second way of invoking the scan. A finding you reject (`n`) is excluded from everything downstream, exactly the same as any other audit run — `n` keeps meaning "don't persist," not a special second meaning invented for Refactor mode.

`wayfinder`'s charting then runs, wrapped exactly as [Plan mode](plan.md) already wraps it — its [enrichment pass](plan.md#enrichment-pass) runs completely unmodified, reading persisted Conflicts/Exceptions out of `docs/frontend-architecture/` into the map's Notes. Refactor mode doesn't add a second findings-handoff mechanism at the Notes level — Audit writes, Plan reads, in sequence, both unchanged.

One thing *is* new: a Refactor-mode-only briefing fires before `wayfinder`'s step 1 ("Name the destination"), separate from and in addition to Plan mode's existing after-step-1 enrichment. Once Audit's write-back completes, its own closing summary line (the [example-turn](audit.md#example-turn) shape: *"Audit complete — N topics scanned, M with new Conflicts to review: `<topics>`"*) plus what was actually accepted is handed to `wayfinder`'s step 1 as context, so the destination itself is shaped by what Audit found (e.g. "consolidate the three divergent `Button`-family tiering patterns") rather than named generically and retrofitted around findings later.

Full composed sequence:

1. Hard dependency gate — before Audit's scan starts.
2. Audit's full scan + interactive write-back — unmodified, findings persist into `docs/frontend-architecture/`.
3. *(new)* Refactor-mode briefing — Audit's closing summary handed to `wayfinder` as pre-step-1 context.
4. `wayfinder` step 1 — name the destination, informed by the briefing.
5. Plan mode's enrichment pass — unchanged, fires after step 1/before step 2, reads the freshly-written docs.
6. `wayfinder` step 2 — frontier-grilling, breadth-first, decides the real ticket set (grouping, prioritizing, or dropping findings — never a mechanical one-ticket-per-finding dump).

No change to `audit.md`, `plan.md`, or `wayfinder`'s own spec — Refactor mode's own glue code sits between three otherwise-untouched systems.

## Map seed shape

What lands in the resulting map's `## Destination` and `## Notes`, concretely — prototyped across three candidate shapes at [`references/refactor-briefing-prototype.md` on `prototype/refactor-briefing-shape`](https://github.com/dsfx3d/frontend-architecture/blob/prototype/refactor-briefing-shape/references/refactor-briefing-prototype.md).

**Destination** stays disciplined — a synthesized one-to-two-line statement, `wayfinder` step 1's normal output, just informed by the pre-step-1 briefing rather than transcribing it. E.g. "Consolidate the divergent `Button`-family component tiering and settle whether draft-form state belongs in a shared store or local component state." No verbatim findings dump — that blows past `wayfinder`'s Destination-length discipline and duplicates text that also lives in `docs/frontend-architecture/`'s Conflicts sections, drifting the moment a later Audit run refreshes them.

**Notes** gets [Plan mode](plan.md#notes-content-shape)'s existing enrichment pass completely unchanged — gist-by-default guidance, fuller distillation plus Conflicts/Exceptions callout for relevant topics. On top of that, exactly **one new line**, first in Notes: a provenance gist reusing Audit's own closing-summary phrasing verbatim — e.g. "Seeded from an Audit run (`<date>`): 6 topics scanned, 2 flagged — component-tiering, state-management (4 clean). Both findings accepted during write-back; see the Conflicts callouts below — this map exists to resolve them."

The provenance line earns its place because, for Refactor mode unlike Plan mode, a recorded Conflict/Exception isn't incidental background — it's the reason the map exists at all. Without a marker, a Refactor map's Notes would look identical to an ordinary Plan-mode map that merely happens to have a relevant pre-existing Conflict. The line is a tally-and-pointer, not a second copy of the findings — the findings themselves still live exactly once, in the unchanged Conflicts callout, matching `wayfinder`'s index-not-store principle.

## Execution offer

**Detection**: fires at the end of every "work through the map" session, right after `wayfinder` step 5's fog-graduation. The check requires *both* an empty frontier (no open/unblocked/unclaimed tickets) *and* an empty "Not yet specified" section (no fog left that could still graduate into more tickets) — an empty frontier alone doesn't prove the way is clear. That exhaustion check is additionally gated by the [drift re-scan checkpoint](#drift-during-ticket-resolution) below, at the same point: applied drift becomes new tickets/fog, which alone breaks the empty-frontier-and-fog precondition, so the offer simply doesn't fire that session.

**Handoff payload**: the whole resolved map — Destination plus the full Decisions-so-far index — goes to the execution tool as the spec, mirroring `/implement`'s own contract ("a spec or set of tickets"). No filtering heuristic for "which tickets describe an actual code change" — Decisions-so-far already excludes scoping-only chatter (Out of scope is a separate section that never graduates).

**Offer wording** (success-path twin of the [dependency gate](#dependency-gate)'s skip message):

> "Map complete — no tickets left to resolve. Execute the resolved plan now with `/implement` (preferred) or `/tdd`? [y/n]"

**Decline**: the map issue is closed, with a comment recording that execution was offered and declined. **Re-trigger later**: no new mechanism — reopening the closed map issue and re-invoking Refactor mode's "work through the map" on it re-runs the same exhaustion check and re-fires the offer if it still passes. "One-time" means once per session-reaching-exhaustion, not a permanent lockout; reopening the issue is the deliberate human signal that resets it.

## Output shape

Refactor mode produces exactly one visible turn beyond the map itself: a verbatim re-anchor of Audit's own closing headline, shown immediately before `wayfinder` step 1 begins. Per [Diagnose phase and findings handoff](#diagnose-phase-and-findings-handoff), that headline already appears once, as the lead line of Audit's own unmodified output, before its per-topic `Apply this update? [y/n/edit]` prompts — likely to have scrolled out of view by the time charting opens after several such prompts. Refactor mode redisplays it verbatim, no added transition prose, as a clean seam between "diagnosis is done" and "now we plan."

**Clean run short-circuits.** If Audit's scan comes back fully clean (zero topics with new Conflicts/Exceptions), Refactor mode redisplays the all-clean headline, then stops — it does not proceed into `wayfinder` charting. A map's Destination needs something to synthesize toward, and a Refactor map exists specifically to resolve accepted findings; zero findings means nothing to chart.

**Purely ephemeral.** No new persisted artifact beyond what's already locked: Audit's own normal writes into `docs/frontend-architecture/<topic>.md`, and the map's Notes provenance line ([Map seed shape](#map-seed-shape)). No separate report file or log entry — mirrors [audit.md](audit.md#output-shape-and-persistence)'s own "no separate report file" posture.

## Drift during ticket resolution

A Refactor map's tickets can take days or multiple sessions to resolve, and unlike Plan mode's Notes — which are just incidental background a ticket session can shrug off and zoom the linked doc for — a Refactor map's seeded Conflicts/Exceptions *are* the reason the map exists at all. If the codebase drifts while the map is being worked, the findings behind the Destination can go from "the case for refactoring" to actively wrong or incomplete. [Plan mode](plan.md#notes-content-shape)'s "no staleness detection needed" conclusion does not transfer here.

**Mechanism**: a single re-scan checkpoint gating the [execution offer](#execution-offer) above, reusing Audit's existing diff-proposal machinery — not a per-session check on every "work through the map" turn. It only matters once, right before someone acts on the resolved plan.

- **What re-runs**: a full Audit scan across whatever topics [index.md](index.md)'s registry currently lists at re-scan time — read live, not a count fixed at chart time — same path scope as the original chart-time invocation. A narrower rescan could miss drift in a topic that was clean, or didn't exist, at chart time.
- **Materiality bar**: none — reuse Audit's existing diff-proposal flow unfiltered. Every changed Conflict/Exception bullet surfaces with its `clearly`/`likely`/`borderline` hedge word via the normal `Apply this update? [y/n/edit]` turn. Audit itself has no confidence threshold; a second, uncalibrated materiality cutoff here would duplicate that filter rather than reuse it.
- **Effect on the offer**: no separate blocking rule. Applied drift becomes new tickets/fog on the map, same as any newly-surfaced decision — which alone breaks the execution offer's empty-frontier-and-fog precondition, so the offer simply doesn't fire that session. It re-fires naturally once the drift is resolved into decisions and the map empties out again.
- **Pruning**: add-only. The checkpoint never rewrites the map's Destination text or auto-closes/reopens tickets when a re-derived finding disappears — Destination is a synthesized snapshot, not a live mirror, and scope judgment calls (out-of-scope, moot-ticket calls) stay with a human reviewing a specific ticket, not an automated checkpoint.
- **Persistence**: no new persisted artifact. Accepted diffs already land durably in `docs/frontend-architecture/*.md` (Audit's existing write-back); new drift already lands durably as tickets/fog on the map.

Net: the checkpoint *is* Audit's diff-proposal flow, re-run over the live topic registry, gated at exactly the point the execution offer already checks — no new machinery invented anywhere.
