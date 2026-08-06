# frontend-architecture

An agent skill: tech-stack-agnostic patterns for structuring a component-based frontend app.

## What this is

`frontend-architecture` is a Claude agent skill for teams building component-based
frontends on any stack. It gives an agent (and a human reviewer) a shared set of
placement rules — where a component, a data call, a piece of state, or a form
validator belongs — stated independently of any specific framework or library.

## Topics

- [Component tiering](references/component-tiering.md) — where a new shared component belongs
- [Feature-folder organization](references/feature-folders.md) — the global/feature boundary
- [Data/service boundary](references/data-service-boundary.md) — how data and service calls are wired
- [State-management layering](references/state-management.md) — where state should live
- [Page composition & data loading](references/page-composition.md) — composing a page or route
- [Forms & schema validation](references/forms-validation.md) — wiring a form to validation

## Prerequisites

The six placement-rule topics above have no prerequisites — they work standalone.

Project-initialization (auto-detected `docs/frontend-architecture/` scanning) additionally requires the `grilling` and `domain-modeling` skills to be installed alongside this one. Without them, init won't run — the six topics still work, just without project-specific context.

## Installation

Via [`npx skills`](https://github.com/vercel-labs/skills):

```
npx skills add dsfx3d/frontend-architecture
```

Or manually: copy `SKILL.md`, `references/`, and `LICENSE.txt` into
`.claude/skills/frontend-architecture/` (project-level) or
`~/.claude/skills/frontend-architecture/` (user-level).

## License

Apache-2.0 — see [LICENSE.txt](LICENSE.txt).
