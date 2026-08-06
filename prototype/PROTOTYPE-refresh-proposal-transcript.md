# PROTOTYPE — throwaway, not part of the skill

Answers: "what does a refresh proposal concretely look like in an agent turn?"
(ticket: Refresh/diff-proposal interaction shape)

Not a running app — this repo has no execution environment for the skill
itself, so the reactable artifact is a sample transcript: three candidate
shapes for the same refresh turn, back to back, so they can be compared
directly. Pick one (or a hybrid) and fold the verdict into references/init.md.

Scenario held constant across all three variants: `docs/frontend-architecture/
state-management.md` was last scanned at commit `a1b2c3d`. The skill is
invoked on a `state-management`-topic task, does its batched `git log`
staleness check, and finds relevant drift: a new `stores/` directory and a
Zustand dependency bump.

---

## Variant A — inline unified diff per section

    Heads up: `state-management.md` was last scanned at `a1b2c3d` (12 days
    ago). I found changes since then relevant to this topic. Proposed update:

    ```diff
     ## Stack facts

    -- Zustand 4.4.1 (package.json)
    +- Zustand 4.5.0 (package.json)

     ## Project conventions

    -- No dedicated store directory found.
    +- Stores live under `src/stores/*.store.ts`, one file per domain
    +  (inferred from repo structure: src/stores/cart.store.ts,
    +  src/stores/session.store.ts)

     ## Conflicts

     - None found.

     ## Exceptions

     - None found.

     ## Last scanned

    -Last scanned: 2026-07-25 (a1b2c3d)
    +Last scanned: 2026-08-06 (f9e2b71)
    ```

    Apply this update? [y/n/edit]

**Reads like:** a code review. Precise, but a human has to parse diff
syntax to know what actually changed in prose terms — costly for a doc
whose whole point is human-readable project facts.

---

## Variant B — grouped prose summary, no diff syntax

    `state-management.md` looks stale (last scanned 12 days ago, and I see
    relevant changes since). Here's what I'd update:

    **Stack facts**
    - Changed: Zustand version 4.4.1 → 4.5.0 (package.json)

    **Project conventions**
    - Added: a dedicated store directory, `src/stores/*.store.ts`, one file
      per domain (inferred from repo structure)

    **Conflicts** — no change (still: None found)
    **Exceptions** — no change (still: None found)

    **Last scanned** — bump to 2026-08-06 (f9e2b71)

    Apply this update? [y/n]

**Reads like:** a changelog entry. No diff syntax to parse, but loses
exact before/after wording — "Changed: X → Y" is a paraphrase of the diff,
not the literal old and new bullet text.

---

## Variant C — hybrid: one-line-per-section summary, diff only where it bites

    `state-management.md` is stale (scanned 12 days ago @ a1b2c3d; relevant
    drift found @ f9e2b71). Proposed update:

    - Stack facts: 1 change (Zustand version bump)
    - Project conventions: 1 addition (new store directory convention)
    - Conflicts: no change
    - Exceptions: no change

    Full diff:
    ```diff
    -- Zustand 4.4.1 (package.json)
    +- Zustand 4.5.0 (package.json)
    -- No dedicated store directory found.
    +- Stores live under `src/stores/*.store.ts`, one file per domain
    +  (inferred from repo structure)
    ```

    Apply this update? [y/n/edit]

**Reads like:** the summary answers "do I even need to look closely?" at a
glance; the diff underneath is there for the section that actually needs
scrutiny (here, none are Conflicts, so this is a cheap accept — but if a
proposed Conflicts-section change appeared, the diff is what shows exactly
what's being flagged, which matters more the higher-stakes the section).

---

## Open sub-question surfaced while building this

Should **Conflicts** and **Exceptions** sections always get the full diff
treatment (Variant A style) even under Variant B/C, since those are the
sections a human most needs to scrutinize precisely — while Stack
facts/Project conventions can stay summarized? i.e. the hybrid could be
section-aware rather than uniform.
