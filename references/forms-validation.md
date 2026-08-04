# Forms & schema validation

## Two purposes, one folder

A feature's schema folder (per [feature-folder organization](feature-folders.md)'s skeleton) holds two distinct kinds of schema — same folder, same idiom, unrelated purposes:

1. **Data-boundary schemas** — validate DTOs/API responses/cache reads. These are the schema layer the [data/service boundary](data-service-boundary.md) topic's services validate/parse through. Most schemas in a codebase are this kind, and have no connection to any form.
2. **Form-input schemas** — validate user-entered data, wired to a form library's resolver. A minority; only add one when a form actually needs client-side validation before submit.

There is no global schema directory — every schema is feature-local, named for what it validates (a `*Dto`/`*Schema` suffix for data-boundary shapes, `*InputSchema` for a form's own fields).

Schema reuse between the two is allowed but not required: if a form's fields exactly match an existing DTO shape, the same schema can serve as the form's resolver. More often a form gets its own narrower input schema, scoped to just the fields it collects. Pick per form — this doesn't mandate either.

## Form-to-schema wiring

A hook (the feature's hooks layer) owns the form instance: creates it with the schema wired through a resolver adapter, wraps the submit handler with the feature's submit logic — calling a service per the [data/service boundary](data-service-boundary.md) topic, invalidating cache, mapping a thrown error onto form state — and exposes the form's control/errors/submit handle to the form's UI. The form's root surface doesn't create the resolver or instantiate the form itself if that hook layer exists.

## Error surfacing

- **Field-level**: sourced from the resolver's parse failure on that field, read from the form library's own field-error state.
- **Root/server-level**: set as a form-level (non-field) error for a submit-time failure — a thrown, typed service error (per the [data/service boundary](data-service-boundary.md) topic) that isn't attributable to one field.
- **Message content**: plain, user-facing strings by default. Write messages as short, stable identifiers rather than ad hoc prose, so a later switch to translation keys is a mechanical rename, not a redesign.

## Presentational field components

A hand-authored field-display molecule (label + error state + helper text) takes flattened primitives — `error: boolean`, `helperText: string` — and has zero awareness of the form/schema library: it never reads form state or imports the resolver/validation library itself. The container (the feature's form hook plus its root component) does the reading and flattening; the molecule only renders.

Exception: vendor/generated `ui`-tier primitives are exempt. Per the [component tiering](component-tiering.md) topic, `ui` is never hand-authored — and a generated form kit is deliberately coupled to the form library's context by the vendor. The presentational-only rule binds any hand-authored field molecule built next to or on top of that primitive, not the generated primitive itself.
