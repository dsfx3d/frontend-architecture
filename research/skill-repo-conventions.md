# Research: skill-repo layout, license placement, and manifest conventions

Investigates GitHub issue [#2](https://github.com/dsfx3d/frontend-architecture/issues/2), to unblock the front-matter/file-layout decision in [#3](https://github.com/dsfx3d/frontend-architecture/issues/3).

All findings below are verified against primary sources (`gh api` calls against the actual repo contents, at the commit/ref noted) as of 2026-08-03, not blog posts or summaries.

## 1. `anthropics/skills`

Source: `gh api repos/anthropics/skills/contents/...`, `ref=main` (HEAD at time of research).

- **Layout is flat**: every skill lives at `skills/<skill-name>/` directly under the repo root — e.g. `skills/frontend-design/`, `skills/pdf/`, `skills/xlsx/`, `skills/skill-creator/`, `skills/mcp-builder/`, `skills/docx/`, etc. There are no category subfolders; `skills/` is a single flat listing of ~18 skill directories.
- **`skills/frontend-design/SKILL.md`** frontmatter (fetched directly, `https://raw.githubusercontent.com/anthropics/skills/main/skills/frontend-design/SKILL.md`):
  ```yaml
  ---
  name: frontend-design
  description: Guidance for distinctive, intentional visual design when building new UI or reshaping an existing one. Helps with aesthetic direction, typography, and making choices that don't read as templated defaults.
  license: Complete terms in LICENSE.txt
  ---
  ```
- **License placement**: the `license:` frontmatter field is a plain string pointing at a file, and that file (`LICENSE.txt`) sits in the **same folder as `SKILL.md`** — `skills/frontend-design/LICENSE.txt`, `skills/frontend-design/SKILL.md`. This pattern is consistent across every skill checked (`pdf`, `xlsx`, `docx`, `skill-creator`, `mcp-builder`, `frontend-design`) — each skill folder carries its own `LICENSE.txt` sibling to `SKILL.md`, rather than relying on a single repo-root license. (There is no repo-root LICENSE file in this repo at all; licensing is per-skill.)
- **Multi-file skills**: several skills ship more than just `SKILL.md` + license:
  - `skills/pdf/` has `SKILL.md`, `LICENSE.txt`, plus **`forms.md` and `reference.md` sitting directly beside `SKILL.md` in the same folder** (flat, no subfolder), and a `scripts/` subdirectory for executable helpers.
  - `skills/mcp-builder/` has `SKILL.md`, `LICENSE.txt`, a `reference/` subfolder, and a `scripts/` subfolder.
  - `skills/skill-creator/` has `SKILL.md`, `LICENSE.txt`, and `references/`, `assets/`, `scripts/`, `agents/`, `eval-viewer/` subfolders.
  - `skills/xlsx/` and `skills/docx/` are simpler: just `SKILL.md`, `LICENSE.txt`, `scripts/`.

  Conclusion: supporting reference material is placed either as flat sibling `.md` files next to `SKILL.md` (simple case, e.g. `pdf`) or grouped into a `references/`-style subfolder alongside other categorized subfolders (`scripts/`, `assets/`) when there's enough material to warrant it. Both patterns coexist within the same repo; there's no single hard rule beyond "keep everything under the skill's own folder."

## 2. `mattpocock/skills`

Source: `gh api repos/mattpocock/skills/contents/...`, `ref=main`.

- **Layout uses category subfolders** under a top-level `skills/`, confirming the shape referenced by `skillPath`s like `skills/engineering/code-review/SKILL.md` and `skills/productivity/grill-me/SKILL.md`. The actual subfolders present are: `skills/engineering/`, `skills/productivity/`, `skills/personal/`, `skills/misc/`, `skills/deprecated/`, `skills/in-progress/`. Each category folder then contains one directory per skill (e.g. `skills/engineering/code-review/`, `skills/productivity/grill-me/`).
- **License placement**: there is a **single repo-root `LICENSE`** file (MIT, "Copyright (c) 2026 Matt Pocock") and no per-skill license files or `license:` frontmatter field. Checked `skills/engineering/code-review/SKILL.md` frontmatter directly — it has only `name` and `description`, no `license` key. This is a different convention from `anthropics/skills`: one license for the whole repo, not one per skill.
- **Multi-file skills**: `skills/engineering/code-review/` contains `SKILL.md` + an `agents/` subfolder (holding `openai.yaml`, an agent-specific config, not a markdown reference doc). `skills/productivity/grill-me/` has the same `SKILL.md` + `agents/` shape. Supporting material here is grouped into named subfolders (`agents/`), not flat sibling files.
- **README structure**: there is a top-level repo `README.md` (marketing/install instructions, distinct from any skill), **plus a separate `README.md` inside each category folder** (e.g. `skills/engineering/README.md`, `skills/productivity/README.md`) that indexes the skills in that category with links like `[ask-matt](./ask-matt/SKILL.md)`. So there are three tiers of README-like documentation: repo root, category, and (implicitly) the SKILL.md itself — no additional per-skill README.

## 3. Other public skill repos (corroboration)

- **`agentskills/agentskills`** ("Specification and documentation for Agent Skills," `docs/specification.mdx`, fetched via `gh api repos/agentskills/agentskills/contents/docs/specification.mdx`) is a **formal, vendor-neutral spec** for the `SKILL.md` format and directly corroborates both repos above:
  ```
  skill-name/
  ├── SKILL.md          # Required: metadata + instructions
  ├── scripts/          # Optional: executable code
  ├── references/       # Optional: documentation
  ├── assets/           # Optional: templates, resources
  └── ...               # Any additional files or directories
  ```
  Per the spec: `name` and `description` are the only required frontmatter fields; `license` is optional ("License name or reference to a bundled license file... We recommend keeping it short (either the name of a license or the name of a bundled license file)," with the example `license: Proprietary. LICENSE.txt has complete terms in LICENSE.txt` — i.e. the same "reference a sibling file" pattern as `anthropics/skills`). Optional fields also include `compatibility`, `metadata` (arbitrary key-value), and an experimental `allowed-tools`. The `name` field must match the parent directory name and be lowercase-hyphenated.
- Search also surfaced `microsoft/skills`, `magnus919/agent-skills`, and `heilcheng/awesome-agent-skills` as further examples of the same `skill-name/SKILL.md` + optional `scripts/`/`references/`/`assets/` shape, reinforcing that this is a broadly-adopted convention rather than an Anthropic-only idiom.

**Synthesis**: both real-world repos and the formal spec agree on the core shape — a directory per skill containing `SKILL.md`, with any supporting files/subfolders (`references/`, `scripts/`, `assets/`, or flat sibling `.md` files) nested under that same skill directory. Where they diverge is license placement (per-skill `LICENSE.txt` referenced from frontmatter, vs. one repo-root `LICENSE` with no per-skill reference) and whether skills are grouped into category subfolders. Both are valid, spec-compliant choices — the spec is silent on multi-skill repo organization and license placement is explicitly left to "short reference," not mandated to any specific path.

## 4. Manifest/lock-file convention: `skills-lock.json`

The `source` / `sourceType: "github"` / `skillPath` / `computedHash` shape **is a documented, tool-specific convention** — it belongs to the `vercel-labs/skills` project (the `npx skills` CLI, "the open agent skills tool"), not a general cross-tool standard.

Primary source: `gh api repos/vercel-labs/skills/contents/src/local-lock.ts` (fetched directly from `main`). This file implements exactly the **project-level** lock file (as opposed to a separate *global* lock file at `~/.agents/.skill-lock.json` implemented in `src/skill-lock.ts`, which uses different field names). Key excerpt:

```ts
const LOCAL_LOCK_FILE = 'skills-lock.json';
...
export interface LocalSkillLockEntry {
  /** Where the skill came from: npm package name, owner/repo, local path, etc. */
  source: string;
  sourceUrl?: string;
  ref?: string;
  /** The provider/source type (e.g., "github", "node_modules", "local") */
  sourceType: string;
  /**
   * Path to the skill's SKILL.md within the source repo (e.g., "skills/pdf/SKILL.md").
   */
  skillPath?: string;
  /**
   * SHA-256 hash computed from all files in the skill folder.
   */
  computedHash: string;
  ...
}
```

This matches the ticket's `skills-lock.json` fields field-for-field (`source`, `sourceType: "github"`, `skillPath`, `computedHash`), confirming the local file referenced in the ticket was produced by `npx skills` (or a fork/version of it) installing skills from `mattpocock/skills`-style paths. A real-world example of this exact file shape, fetched from `gh api repos/vercel-labs/open-agents/contents/skills-lock.json`:

```json
{
  "version": 1,
  "skills": {
    "ai-sdk": { "source": "vercel/ai", "sourceType": "github", "computedHash": "58ce68f6..." }
  }
}
```

(`skillPath` is simply omitted there because those particular skills live at the source repo root — the field is optional, present only when the skill is nested.)

**Implication for the new repo**: `skills-lock.json` is an artifact this *consuming* tool generates in *whatever repo installs* a skill — it is not something a skill's *source* repo needs to produce, contain, or conform to. The new `dsfx3d/frontend-architecture` repo only needs to be a normal git repo with `SKILL.md` file(s) at a discoverable, sensible path (e.g. `SKILL.md` at root for a single skill, or `skills/<name>/SKILL.md` if it ever grows into a multi-skill repo) so that tools like `npx skills add dsfx3d/frontend-architecture` can locate and hash it. No manifest file needs to be authored by hand in this repo to satisfy that convention.

## Summary of options for ticket #3

| Question | `anthropics/skills` | `mattpocock/skills` | Formal spec (`agentskills/agentskills`) |
|---|---|---|---|
| Skill location | `skills/<name>/SKILL.md` (flat) | `skills/<category>/<name>/SKILL.md` (categorized) | `<name>/SKILL.md` (location within repo unspecified) |
| License placement | Per-skill `LICENSE.txt` beside `SKILL.md`, referenced via `license:` frontmatter | Single repo-root `LICENSE`, no per-skill reference | Optional `license:` field; recommends a short reference to a "bundled license file" (implies sibling placement) but doesn't mandate |
| Supporting docs | Flat sibling `.md` files (e.g. `forms.md`) or `references/`/`scripts/`/`assets/` subfolders, all under the skill's own directory | `agents/` subfolder under the skill's own directory | Optional `scripts/`, `references/`, `assets/` subfolders under the skill's own directory |
| Manifest/lock file | None | None (npm `package.json`/`package-lock.json` present, but that's for the repo's own build tooling, not a skill manifest) | Not part of the spec — lock files are a consumer-side (installer) concern |

Given `dsfx3d/frontend-architecture` will hold a single skill (in the same family as `frontend-design`/`responsive-design`), the closest structural precedent is `anthropics/skills`' per-skill layout: `SKILL.md` at a predictable path with any supporting reference files nested under the same directory, and (if a license is desired) a sibling `LICENSE`/`LICENSE.txt` referenced from the `license:` frontmatter field. No `skills-lock.json` or other manifest file is required in the skill's own repo.
