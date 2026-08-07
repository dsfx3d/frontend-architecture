# Plan mode

## Trigger

Plan mode fires only on a request that is *both* (a) planning intent and (b) explicit naming of frontend-architecture or one of its topics/patterns — never inferred from a request merely being frontend-domain-shaped ("plan how I'll build the checkout flow" alone does not trigger it). No slash-command-only restriction — natural language is accepted as long as it names frontend-architecture explicitly, mirroring [Audit mode](audit.md#trigger)'s trigger style.

`/frontend-architecture plan this feature` is the canonical example, playing the same role as Audit's `"audit this project"` — a preferred, unambiguous form, not an exclusive requirement. Equivalent natural-language phrasing that explicitly names frontend-architecture ("plan this with frontend-architecture") is also valid.

An ordinary `/wayfinder` invocation with no mention of frontend-architecture proceeds as wayfinder's own standalone map-charting — this skill never intercepts it.

## Dependency gate

Plan mode's own charting flow delegates to `wayfinder`, `/grilling`, and `/domain-modeling` — all three are load-bearing, not just wayfinder in the abstract. Before plan mode runs, check for all three skills under, in order:

1. `.agents/skills/<name>/` — the shared cross-harness convention
2. `.claude/skills/<name>/` — Claude Code
3. `.codex/skills/<name>/` — Codex CLI
4. `.opencode/skills/<name>/` — OpenCode vendor path
5. `.agent/skills/<name>/` — Antigravity variant

Each checked at project level (`<repo>/<path>`) then user level (`~/<path>`). The gate passes per-skill once found at any one location. It fires immediately on detecting the trigger above — before "Name the destination," before the enrichment pass below.

**On failure**: full bail-out, same posture as [init.md](init.md#dependency-gate). Plan mode does nothing — no map, no partial Notes, no enrichment gathering. The six placement-rule topics, project-initialization, and audit mode are all completely unaffected.

Show this message once per session (an in-memory flag, nothing written to disk), pinpointing exactly which skill(s) are missing — a deviation from init.md's gate, which lists both required skills regardless of which one actually failed; more useful here since a partial miss across three skills is plausible:

> "Plan mode requires the `wayfinder`, `grilling`, and `domain-modeling` skills. Missing: `<missing skill(s)>`, not found under `.agents/skills/`, `.claude/skills/`, `.codex/skills/`, `.opencode/skills/`, or `.agent/skills/` (project or user level). Install them for your harness, then retry — this doesn't affect frontend-architecture's ordinary placement guidance, project-initialization, or audit mode, which all work the same without them."

## Enrichment pass

Once the gate passes and wayfinder's charting begins, plan mode injects this skill's current guidance right after wayfinder's step 1 (destination named) and before step 2's frontier-grilling — so the breadth-first grilling itself is informed by relevant guidance, not just annotated onto the map afterward:

1. Parse [index.md](index.md)'s current entries at invocation time — never a fixed topic list baked into this spec.
2. Pull the one-line gist for every entry, by default — keeps the map's Notes compact, matching wayfinder's low-resolution-map principle.
3. For whichever topic(s) are judged — by semantic reading of the destination text against each entry's "use when"/gist, not keyword matching — relevant to the destination being charted, also read that topic's full reference file and fold a fuller distillation into Notes for just those.
4. Check for `docs/frontend-architecture/` at the detected package root, reusing [init.md](init.md#packagemonorepo-detection)'s package/monorepo detection verbatim — no new decision here. Fold it in per [Notes content shape](#notes-content-shape) below if present; otherwise surface the [heads-up](#no-docsfrontend-architecture-present) below.

## Notes content shape

What lands in the resulting map's `## Notes`, concretely — gist-by-default plus link, with relevant Conflicts/Exceptions called out. Chosen over pasting full reference bodies (goes stale the moment this skill iterates) or re-gathering on every ticket session (re-runs the whole enrichment pass before each of the map's many later sessions can even start grilling); prototyped across three candidate shapes at [`references/plan-notes-prototype.md` on `prototype/plan-notes-shape`](https://github.com/dsfx3d/frontend-architecture/blob/prototype/plan-notes-shape/references/plan-notes-prototype.md).

- Every topic from [index.md](index.md) gets its one-line gist, always.
- Topic(s) judged relevant to the destination additionally get a slightly fuller distillation *and* a link to the live reference file — never the full reference body pasted in.
- `docs/frontend-architecture/` folds in the same way when present: Stack facts/Project conventions stay implicit (a ticket zooms the doc if it needs them), but a recorded **Conflict/Exception belonging to a topic already judged relevant** gets pulled into an explicit callout — the "does this decision fight our own codebase" fact a grilling session needs already in view. Conflicts/Exceptions for topics judged *not* relevant stay out, same as the rest of that topic's content.

**Snapshot, not a standing pointer.** Notes is written once, at chart time — not a live instruction telling every later ticket session (each a separate agent session) to re-invoke frontend-architecture. Because relevant topics are captured as gist-plus-link rather than pasted text, a ticket session that suspects drift or needs the full rule zooms the linked reference file or `docs/frontend-architecture/<topic>.md` directly — this is just wayfinder's existing "work through the map" step 3 ("zoom as needed"), not a new mechanism.

## No docs/frontend-architecture/ present

Plan mode doesn't require `docs/frontend-architecture/` to exist — it works with or without, enriching with it when present ("no init gate"). But absence is surfaced, not silenced: staying silent would hide that project-specific enrichment (stack facts, conventions, recorded Conflicts/Exceptions) is available at all.

The surfacing is an informational heads-up, not an interactive "run it now?" offer — an offer would mean triggering init's own dependency-gated flow (its `grilling`/`domain-modeling` gate plus its topic scan) mid-charting, stacking a whole second gated subsystem into what's supposed to be a quick map-charting session. Mirrors [init.md](init.md#dependency-gate)'s own dependency-gate-failure message: shown once, no confirmation step, in-memory only. Fires exactly once, during the enrichment pass above, the same moment step 4 already checks for `docs/frontend-architecture/`'s presence. Chart-time only — does not resurface on later ticket-resolution sessions, consistent with Notes being a snapshot.

> "No `docs/frontend-architecture/` found — this map's enrichment uses frontend-architecture's static guidance only. Run project-initialization first for project-specific enrichment (stack facts, existing conventions, recorded Conflicts/Exceptions) on this and future plan-mode invocations."

## Scope

**Chart-time only.** Plan mode wraps wayfinder's map-*charting* step — a charting-time behavior, not a general map-maintenance mode. Retrofitting an existing map's Notes is out of scope: it would need a second operating shape (detect an existing map, diff/overwrite its Notes, a trigger phrasing distinguishing "refresh" from "create"), and it isn't needed to solve staleness — [Notes content shape](#notes-content-shape) above already handles that within a map's lifetime via zoom-the-link, not a refresh feature.

**Monorepo handling** fully inherits project-init's [package/monorepo detection](init.md#packagemonorepo-detection), verbatim — no new decision, same reuse pattern [Audit mode](audit.md#scope) already used.
