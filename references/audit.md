# Audit mode

## Trigger

Audit fires only on an explicit, unambiguous request to assess existing code against this skill's own patterns — e.g. "audit this project," "check compliance/adherence," "how much does this diverge," "review this against the architecture rules." Worded narrowly enough that it doesn't misfire on an ordinary placement request — "check where this component should go" stays a placement question, not an audit trigger. No slash-command-only restriction; natural language is fine as long as it's unambiguous.

Unlike [project-initialization](init.md), which auto-triggers silently the first time a placement topic is used, audit never runs unasked — its output is read-only reporting, not something to default into.

Audit requires `docs/frontend-architecture/` to exist, since its findings write into project-initialization's existing per-topic `Conflicts`/`Exceptions` sections (see [Relationship to project-initialization](#relationship-to-project-initialization) below) rather than a separate record kind. If it's missing, the agent silently runs [project-initialization](init.md) first, then proceeds straight into the audit — no error, no separate confirmation step for the init leg.

## Scope

An audit run is scoped along two independent axes, either narrowable by how the request is phrased:

- **Topic-set** — which of the six topics to scan. Default: all six.
- **Path** — which subtree of the package to scan. Default: package root. A path only narrows when named explicitly (e.g. "audit the checkout feature") — never picked up from ambient/conversational context (e.g. "the file I was just editing"), so what got audited is always exactly what was said.

The two axes are orthogonal: naming a path doesn't implicitly filter topics. "Audit checkout" still scans all six topics against that path, unfiltered — a topic with nothing found there just reports no new findings, it isn't skipped. Mirrors project-init's "scan all six, not just the triggering one" reasoning ([init.md](init.md#trigger)).

**Monorepo boundary**: audit reuses project-init's existing [package/monorepo detection](init.md#packagemonorepo-detection) — one package per scan, same as init. A bare "audit this project" at a detected monorepo loops all packages internally and rolls the results up into the root-level `index.md`'s aggregate conflict/exception counts — the same aggregation project-init's root map already does (see [index.md and the root-level monorepo map](init.md#indexmd-and-the-root-level-monorepo-map)), just triggered from the audit side rather than only after individual package init/refresh. No disambiguation prompt for the common, unqualified case.

**Ledger write boundary for scoped runs**: a narrowed-path audit only refreshes the `Conflicts`/`Exceptions` bullets whose cited sources fall under that path — it never touches or drops bullets for parts of the codebase this run didn't look at. A bullet's citation is what determines a given run's jurisdiction over it.

## Scan mechanics

Audit reuses project-init's [six-topic scan](init.md#the-six-topic-scan) verbatim — no new detection technique. Init's step 3 already collects hits at path+snippet (instance) granularity, and step 4 already groups hits into one bullet per distinct pattern — the exact mechanism [finding granularity](#finding-granularity) below needs. Enumerating every component/feature instance rather than sampling was never a real alternative here: init's scan is already uncapped (no depth/file-count limit), not a sample.

Three refinements layer on top of the unchanged procedure, rather than replacing it:

1. **Write-back scope**: audit runs the full six steps — all four sections (Stack facts, Project conventions, Conflicts, Exceptions) get derived, since classifying Conflicts/Exceptions in step 5 requires deriving the convention first anyway — but only *writes back* Conflicts and Exceptions. Stack facts and Project conventions are computed as a byproduct, not persisted by an audit run.
2. **Path-scoped runs** (per [Scope](#scope) above): the narrowed path constrains step 2's source walk itself — globs/grep intersected with the scope path — rather than a post-filter applied after a full-package walk.
3. **Write-back mechanism**: reuses init's existing [staleness-and-refresh diff-proposal flow](init.md#staleness-and-refresh) unchanged — Conflicts/Exceptions always get the full diff block when they change, ending in `Apply this update? [y/n/edit]`. No new write path: audit is the trigger that re-runs the scan on demand, not a competing output.

### Finding granularity

Findings are **instance-level at the atomic unit** — anchored to a specific file/component — with **automatic aggregation** into one pattern-level bullet whenever multiple instances share the same divergent pattern. There's no per-topic granularity switch, no six special-cased rules: a single uniform mechanism (group by pattern) naturally produces topic-level variation as an emergent property — component tiering tends to stay near-instance-level (misplacements are individually judged), while state-management tends to aggregate heavily (the same layering mistake repeats project-wide).

A pattern bullet cites every backing instance's source — file path, ADR link, or "inferred from repo structure" — consistent with the per-topic template's existing per-bullet sourcing rule ([per-topic project-doc template](init.md#per-topic-project-doc-template)). No truncation scheme for widely-repeated patterns.

## Relationship to project-initialization

Audit scans fully independently every run — no cross-check, no memory. Step 5's classification (comparing a found pattern against the skill's own static reference pattern) never reads the previously-recorded `Project conventions`/`Conflicts`/`Exceptions` content; every run re-derives from source plus the skill's reference pattern as if starting from zero.

The old doc re-enters the picture only afterward, as the "before" side of init's existing diff-proposal flow (unchanged mechanism, not new to audit). This produces the right behavior without any new suppression logic:

- An already-accepted, still-true **Exception** re-derives to an identical bullet → the diff is empty → nothing is proposed, stays silent. Not because of a "remembered acceptance," but because the fresh derivation and the recorded state happen to match.
- A previously-**rejected** Conflict/Exception (the user said `n`) was never written, so there's nothing to diff against → it resurfaces and gets re-proposed on every subsequent run, identical to before.

No dismissal/rejection memory is introduced. `n` continues to mean exactly what it already means in init's shared refresh flow — "don't persist," full stop, everywhere. The escape hatch for a genuine permanent carve-out is unchanged: say `y` to record it as an Exception, which is what that section is for — the fresh scan will then match it and stay quiet on future runs.

## Confidence and hedging

No confidence threshold — audit flags every divergence it can construct an argument for, mirroring project-init's scan, which has no convention-vs-exception cutoff either. A threshold would need bespoke tuning per topic (component-tiering's signal looks nothing like state-management's), which is real design weight with no principled cutoff.

Uncertainty is communicated with a hedge word drawn from a small **fixed, three-tier vocabulary — `clearly` / `likely` / `borderline`** — prefixed to a bullet. Not a structured confidence field or score (nothing here is actually calibrated; a numeric tag would fake precision), and not free-form phrasing: the `Conflicts` section always gets a full, exact-text diff on change ([staleness and refresh](init.md#staleness-and-refresh)), and an unchanged, already-accepted finding staying silent (see [Relationship to project-initialization](#relationship-to-project-initialization)) depends on that diff comparing *identical* bullet text run-to-run. Free-form hedge phrasing (e.g. "likely misplaced" one run, "probably misplaced" the next) would drift independently of the actual finding and spuriously resurface already-accepted findings for re-review. A fixed vocabulary keeps phrasing deterministic for a given confidence level, preserving that diff-silence property.

When a pattern bullet aggregates several backing instances of varying confidence, the bullet's hedge word reflects the **weakest** instance in the group — no per-instance hedging fragmenting the bullet. Keeps the bullet compact and doesn't overstate the pattern's case from its strongest member.

No separate false-positive suppression mechanism. The hedge word feeds directly into the existing accept/reject write-back flow (`Apply? [y/n/edit]`, non-persistent rejection) — that flow *is* the false-positive filter, now an informed one: the human sees `clearly`/`likely`/`borderline` before deciding, instead of a coin flip.

## Output shape and persistence

No separate report file. Findings refresh the *existing* per-topic `Conflicts`/`Exceptions` sections in `docs/frontend-architecture/<topic>.md`, via init's existing diff-proposal mechanics — a re-run diffs against the previously-recorded doc, it never overwrites or appends.

- **Silent scan, no streamed per-instance/per-pattern heads-up.** Audit runs finish in seconds — there's no long-running-job case for a progress signal — and streaming would just say the same thing twice: once live, once in the final proposal.
- **Sequential per-topic turns, not one combined prompt.** A whole-project run is the existing single-topic-doc refresh flow ([staleness and refresh](init.md#staleness-and-refresh): section-aware diff, `Apply this update? [y/n/edit]`) invoked once per topic that has changes, one after another — not merged into a single mega-diff. This reuses the existing per-doc mechanics verbatim and keeps each accept/reject unambiguously scoped to one doc, rather than inventing "edit-which-topic" semantics for a combined prompt.
- A scope-narrowed run (single topic, per [Scope](#scope)) never had a batching question in the first place — it's just this same single-topic turn.

### Example turn

A whole-project audit that finds a likely component-tiering conflict (three `Button`-family instances aggregating into one pattern) and a borderline state-management call, with the other four topics clean:

    Audit complete — 6 topics scanned, 2 with new Conflicts to review:
    component-tiering, state-management (4 clean: feature-folders,
    data-service-boundary, page-composition, forms-validation).

    First up — component-tiering.md:

    ```diff
     ## Conflicts

    -- None found.
    +- likely: `Button`-family components take no children/composition
    +  slot yet sit under `molecules/`, not `atoms/`, in 3 places —
    +  conflicts with this skill's tiering pattern.
    +  (src/features/checkout/ui/molecules/SubmitButton.tsx,
    +  src/features/auth/ui/molecules/LoginButton.tsx,
    +  src/shared/ui/molecules/IconButton.tsx)
    ```

    Apply this update? [y/n/edit]

    > y

    Applied. Next — state-management.md:

    ```diff
     ## Conflicts

    -- None found.
    +- borderline: `useDraftStore` holds in-progress form state that may
    +  not need to survive a refresh, which would argue for local
    +  component state instead of a shared store — this skill's layering
    +  pattern treats refresh-survival as the deciding test.
    +  (src/features/onboarding/state/draftStore.ts)
    ```

    Apply this update? [y/n/edit]
