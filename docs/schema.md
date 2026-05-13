# Schema reference

All schemas live in `schemas/` and use JSON Schema Draft 2020-12. They share a few conventions:

- `additionalProperties: false` everywhere - unknown fields are rejected, so adding a field requires updating the schema.
- Dates are ISO 8601 (`YYYY-MM-DD`).
- Slugs (`plan_name`, setting `key`) are lowercase ASCII matching `^[a-z0-9][a-z0-9_-]*$`.
- All JSON files in `services/` are UTF-8, 2-space indented, with a trailing newline.

## Data depth levels

The `level` field on `metadata.json` describes how complete an entry is. Tools should treat the level as a contract:

| Level | Required files                                                                                    |
|-------|---------------------------------------------------------------------------------------------------|
| 1     | `metadata.json`                                                                                   |
| 2     | `metadata.json` + at least `documents/tos.json` and `documents/privacy.json`                      |
| 3     | All of Level 2 + at least one file under `plans/` + `recommended_settings.json`                   |

History files (`history/*.json`) are optional at every level.

## `metadata.schema.json`

Top-level descriptor of a service. One per service directory.

| Field          | Type      | Required | Notes                                                                              |
|----------------|-----------|----------|------------------------------------------------------------------------------------|
| `service_name` | string    | yes      | Human-readable name (e.g. `"Figma"`).                                              |
| `vendor`       | string    | yes      | Operating company name (e.g. `"Figma, Inc."`).                                     |
| `category`     | enum      | yes      | One of: design, communication, productivity, ai-assistant, storage, code-hosting, observability, identity, payment, hr, crm, marketing, security, other. |
| `domains`      | string[]  | yes      | All domains/subdomains that should map to this entry. Order is not significant.    |
| `homepage`     | uri       | no       | Primary marketing URL.                                                             |
| `first_seen`   | date      | no       | When the entry was added.                                                          |
| `last_updated` | date      | no       | When the metadata last meaningfully changed.                                       |
| `level`        | 1\|2\|3   | yes      | Data depth, see above.                                                             |
| `tags`         | string[]  | no       | Free-form, deduplicated.                                                           |
| `aliases`      | string[]  | no       | Alternative product names under the same vendor (e.g. `"FigJam"`).                 |

### Example

```json
{
  "service_name": "Figma",
  "vendor": "Figma, Inc.",
  "category": "design",
  "domains": ["figma.com", "www.figma.com"],
  "level": 3
}
```

## `plan.schema.json`

One file per subscription tier under `plans/<plan_name>.json`. The filename slug must equal `plan_name`.

`conditions` is a closed object: each field has a constrained vocabulary so downstream tools can rely on it without string parsing. Use `"unspecified"` (for `ai_training`) or `null` (for numeric fields) when the vendor has not published a clear answer rather than guessing.

The `source_urls` array should always contain at least one vendor-published page that supports the claims in `conditions`.

## `recommended_settings.schema.json`

A single file per service: `recommended_settings.json`. Top-level shape is `{ "settings": [ ... ] }`.

Each setting has:

- a stable `key` slug (so downstream tools can persist user dismissals against it),
- a `category` to group similar settings in UIs,
- `default` and `recommended` values that may be strings, booleans, objects, or null,
- `applicable_plans`, which must match the slugs in `plans/`,
- a `verification_method` describing how the local-sspm Chrome extension can check the live value:
  - `dom_selector` with a CSS/XPath selector,
  - `text_match` with an `expected_text` substring,
  - `api` for verifiable via documented vendor API,
  - `manual` when no automated check is possible.

Severity is an explicit policy choice: `required` means "if this is wrong, surface it as a problem"; `recommended` is a softer nudge; `optional` is informational only.

## `document.schema.json`

One file per policy document under `documents/`. The filename convention is `<kind>.json`, where `<kind>` matches the `kind` field. `last_modified_observed` should reflect the date the vendor displays on the document itself; `last_checked` is when a contributor last verified the link works.

## `history_entry.schema.json`

Files in `services/<service>/history/` form an append-only log. Filename convention: `YYYY-MM-DD-<slug>.json`. Once committed they should not be edited (corrections go in a new entry).

## Writing tips

- Prefer fewer, well-sourced settings over many speculative ones.
- When in doubt about a `pricing_model`, choose `freemium` only if there is genuinely a free tier; otherwise use `paid` even for trial-able plans.
- For services that have undergone rebranding, keep the directory name stable and use `aliases` to record old names.
