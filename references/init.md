# Project-initialization

## Trigger

Init fires when the agent has matched one of the six placement-rule topics and is about to act on it — not merely on this skill loading. At that point, check for `docs/frontend-architecture/` at the detected package root (see [package/monorepo detection](#packagemonorepo-detection) below). If it's missing, init runs and scans all six topics in one pass, not just the one that triggered it — so the index's topic-status table is honest from the moment it exists, rather than showing five "not yet scanned" rows next to one populated one.

Init runs silently — no confirmation prompt before it starts.

## Package/monorepo detection

Walk up from the file being worked on to the nearest `package.json` (or workspace-equivalent manifest) — that's the package root.

Check the repo root (via `.git`) for monorepo signals: a root `package.json` with a `workspaces` field, or a `pnpm-workspace.yaml`, `lerna.json`, `nx.json`, or `turbo.json`. If any of those exist and the package root is not the repo root, it's a monorepo; otherwise single-package, and package root = repo root.

## Dependency gate

Init is built around grilling-style interviewing (surfacing conflicts/gaps one question at a time) and domain-modeling-style decision recording — load-bearing for how init avoids guessing, not decorative. The six placement-rule topics stay dependency-free; only init gates on this.

Before init runs, check for the `grilling` and `domain-modeling` skills under, in order:

1. `.agents/skills/<name>/` — the shared cross-harness convention
2. `.claude/skills/<name>/` — Claude Code
3. `.codex/skills/<name>/` — Codex CLI
4. `.opencode/skills/<name>/` — OpenCode vendor path
5. `.agent/skills/<name>/` — Antigravity variant

Each checked at project level (`<repo>/<path>`) then user level (`~/<path>`). The gate passes once both skills are found at any one location.

**On failure**: full bail-out. Init creates nothing — no `docs/frontend-architecture/`, no scaffolding — because a skeleton with no real interview/decision-recording behind it would itself be a degraded, misleading artifact (looks initialized, isn't). The six topics keep working on the skill's own base patterns, same as before init existed.

Show this message once per session (an in-memory flag, nothing written to disk):

> "Project-initialization requires the `grilling` and `domain-modeling` skills, not found under `.agents/skills/`, `.claude/skills/`, `.codex/skills/`, `.opencode/skills/`, or `.agent/skills/` (project or user level). Install them for your harness, then retry — the six placement-rule topics work fine without them in the meantime."

## The six-topic scan

One general procedure, applied to each topic, parameterized by topic — not six bespoke strategies:

1. Look up the topic's parameter row: keyword hints and file globs (see [per-topic scan parameters](#per-topic-scan-parameters) below).
2. Walk sources in fixed order:
   - `package.json` / lockfile, grepped for the topic's keyword hints.
   - The topic's file globs.
   - A shared doc/ADR search (same procedure for every topic — see [ADR and doc discovery](#adr-and-doc-discovery)).
3. Collect hits as path + snippet.
4. No aggregation, no threshold: every distinct pattern found becomes its own sourced bullet. Conflicting signals (five files use pattern X, three use pattern Y) produce two separate bullets — no majority pick, no merging.
5. Classify each bullet into Stack facts / Project conventions / Conflicts / Exceptions, per the [per-topic doc template](#per-topic-project-doc-template) — Conflicts and Exceptions are decided by comparing the finding against this skill's own reference pattern for that topic.
6. Write the sourced bullets into that topic's doc.

Scan boundary: package-scoped (reuses the package-root boundary above), uncapped — no depth or file-count limit. A depth cap solves an unconfirmed performance problem; a sample would undermine the neutral-recording goal in step 4.

### ADR and doc discovery

Hybrid, same mechanism for all six topics: check a short fixed list of conventional paths first — `docs/adr/`, `docs/decisions/`, `adr/`, `docs/architecture/` — as high-confidence hits, then fall back to a keyword-filtered sweep of the rest of the package's `**/*.md` for anything the fixed list missed (same source-glob mechanism as step 2 above, aimed at markdown). This covers org opinions (README/CONTRIBUTING/style-guide docs) as the same kind of doc — no separate mechanism. Nothing found at either stage: "None found."

## Per-topic scan parameters

| Topic | Keyword hints (package.json/lockfile) | File globs |
|---|---|---|
| [Component tiering](component-tiering.md) | `storybook`, `@radix-ui/*`, `shadcn`, `class-variance-authority`, `chakra-ui`, `@mui/material`, `tailwind-variants` | `**/ui/**`, `**/atoms/**`, `**/molecules/**`, `**/organisms/**`, `**/components/**` |
| [Feature-folder organization](feature-folders.md) | `nx`, `turbo`, `@nrwl/*` | `**/features/**`, `**/modules/**`, `**/domains/**`, `**/src/pages/**`, `**/app/**` |
| [Data/service boundary](data-service-boundary.md) | `axios`, `ky`, `@tanstack/react-query`, `swr`, `urql`, `@apollo/client`, `trpc`, `@trpc/client`, `zod`, `yup` | `**/services/**`, `**/api/**`, `**/queries/**`, `**/*.service.ts`, `**/*Api.ts` |
| [State-management layering](state-management.md) | `zustand`, `redux`, `@reduxjs/toolkit`, `jotai`, `recoil`, `mobx`, `valtio`, `nuqs` | `**/store/**`, `**/stores/**`, `**/state/**`, `**/*.store.ts` |
| [Page composition & data loading](page-composition.md) | `next`, `react-router`, `@remix-run/react`, `astro`, `@sveltejs/kit`, `gatsby` | `**/app/**/page.*`, `**/pages/**/*.tsx`, `**/routes/**`, `**/middleware.ts` |
| [Forms & schema validation](forms-validation.md) | `react-hook-form`, `formik`, `@tanstack/react-form`, `@hookform/resolvers`, `zod`, `yup`, `vee-validate` | `**/schema/**`, `**/schemas/**`, `**/*Schema.ts`, `**/*.form.tsx`, `**/forms/**` |

`zod`/`yup` and the schema globs are shared between the data/service-boundary and forms-validation rows — the schema folder holds both DTO and form-input schemas ([forms & schema validation](forms-validation.md#two-purposes-one-folder)); a hit there is classified by what the schema actually validates, not by which row found it.

## Per-topic project-doc template

Recorded at `docs/frontend-architecture/<topic-slug>.md`, one per topic — slug matches the reference file name (e.g. `component-tiering.md`). Five fixed sections, always present, in this order:

1. `## Stack facts` — discovered, neutral: framework/libraries/versions relevant to this topic.
2. `## Project conventions` — existing patterns found for this topic, recorded neutrally, before judgment.
3. `## Conflicts` — where a convention above contradicts this skill's own pattern. The skill's pattern wins; this section is the persistent flag.
4. `## Exceptions` — structural gaps this skill's tiers have no slot for at all. A carve-out, not a fight.
5. `## Last scanned` — a date and the git commit SHA the scan ran against, e.g. `Last scanned: 2026-08-05 (a1b2c3d)`. The SHA drives drift-based staleness (has anything topic-relevant changed since), not just calendar age.

Rules across all five sections:

- Every bullet cites its source — a file path, an ADR link, or "inferred from repo structure." Sourcing is uniform across the template, not special-cased per section, though it matters most for Conventions/Conflicts/Exceptions, where a human has to judge what's being overridden or carved out.
- A section with nothing to report is not omitted — it stays with an explicit "None found" line. This keeps the five-section shape identical across all six topic docs (needed for uniform rendering/diffing) and disambiguates "never scanned for this" from "scanned, found nothing."

## index.md and the root-level monorepo map

`docs/frontend-architecture/index.md` (package-level) loads on every invocation, so it stays a summary — never a rollup of full doc content:

1. Topic table — the six topics, each linking to its doc, with a status per topic: `populated` / `not yet scanned` / `stale`.
2. Flagged-conflicts summary — existence + count per topic (e.g. "state-management: 1 conflict flagged"), not the conflict detail itself, so a flag surfaces even when its topic doc isn't loaded.
3. Structural-exceptions summary — same shape, for exceptions.
4. Last full scan — the most recent SHA/date across all six topics, a repo-wide staleness signal distinct from each topic's own `Last scanned`.

For a monorepo (one `docs/frontend-architecture/` per package), a root-level map lives at `docs/frontend-architecture/index.md` at the repo root — same relative filename/pattern as a package-level index, distinguished by not sitting next to any per-topic docs of its own. Its content is a package table: one row per package, linking to that package's own `docs/frontend-architecture/index.md`, with conflict/exception counts pulled — not re-derived — from each package index. It carries no per-topic content itself: the six topics are only meaningful at the package level, so the root map never scans or judges, only aggregates links and counts.

## Load model per invocation

- Package-level `index.md` loads on every invocation.
- A per-topic doc loads only on demand, alongside its matching reference file, when that topic is in play.
- The root-level monorepo map is write-only from the agent's side — updated (or created) after init/refresh touches a package-level index — and is never read to inform a placement decision. The six topics are only meaningful at the package level, so there's nothing in the root map an invocation would need.

## Staleness and refresh

Staleness check: a batched `git log <lastScannedSHA>..HEAD -- <cited-path-1> <cited-path-2> ...` per topic doc, run only when that doc is loaded — not a repo-wide "any commit," which would fire on unrelated churn and train users to ignore it. Reuses the source citations the per-topic template already requires on every bullet; no new relevance heuristic.

- Path-cited bullets: real SHA-scoped staleness via the batched check above.
- ADR-link bullets (external, no repo path): excluded from the check entirely — no repo-wide fallback.
- New/uncaptured sources — a convention that emerged in a file the doc never cited — are an accepted blind spot, not solved by a diff-based staleness check.

Refresh (staleness-triggered or manual) never silently overwrites a recorded doc — it proposes the change and the user accepts. The proposal is a section-aware hybrid: it always opens with a one-line-per-section summary, but which sections get a diff underneath depends on the section, not on whether any section in the batch changed:

- **Stack facts, Project conventions** — summary line only (prose form, e.g. "Changed: Zustand 4.4.1 → 4.5.0"). Lower-stakes, high-volume facts; paraphrase is fine.
- **Conflicts, Exceptions** — always get the full diff block when they change, never summarized-only. These are the sections a human must scrutinize precisely (the skill overriding a project convention, or a structural carve-out), so the exact before/after wording is load-bearing.
- **Last scanned** — just the bump line (old date/SHA → new date/SHA), no diff either way.

The turn always ends with `Apply this update? [y/n/edit]`, regardless of shape.

## Recording and flagging divergence

When a scan finds a **Conflict** (an existing convention contradicts this skill's pattern) or an **Exception** (a structural gap this skill's tiers have no slot for), two things happen, and only two:

1. **Record it** — a sourced bullet in the per-topic doc's `Conflicts` or `Exceptions` section, stated neutrally as fact.
2. **Inform the user in-conversation**, framed as something worth their attention, not a recommendation to act on:

   > "Heads up: `<file/path>` uses `<existing convention>`, which conflicts with this skill's `<pattern>` (now recorded in `<topic>.md`). I'm not changing it — whether it's worth aligning is your call, not something to default into just because it's flagged."

The agent never drafts a fix, never offers to apply one, never asks "want me to update this?" This applies wherever a conflict/exception surfaces, not only during an init scan — see the additive-only rule in [SKILL.md](../SKILL.md).

This rule governs the project's own source code. It does not restrict the refresh flow's diff-proposal above, which targets the tracking docs themselves (`docs/frontend-architecture/*.md`) — a separate, already-approved surface.
