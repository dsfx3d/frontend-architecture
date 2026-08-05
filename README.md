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

## Installation

_(pending — see [issue #16](https://github.com/dsfx3d/frontend-architecture/issues/16))_

## License

Apache-2.0 — see [LICENSE.txt](LICENSE.txt).
