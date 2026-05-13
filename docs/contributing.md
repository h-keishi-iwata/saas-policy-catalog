# Contributing to saas-policy-catalog

Thank you for considering a contribution. This catalog is only useful if it stays accurate and well-sourced.

## Ground rules

- **Cite primary sources.** Every claim in a `plan.json` or `recommended_settings.json` must be traceable to a vendor-published page. Put those URLs in `source_urls`.
- **Do not include user-specific data.** This is a public knowledge base. Do not commit account IDs, cookies, screenshots of personal accounts, or anything that would identify an individual user.
- **One service per pull request** unless changes are mechanical (e.g. a schema migration applied across all services).
- **Match the existing style.** UTF-8, 2-space JSON indentation, trailing newline, lowercase-ASCII slug filenames.

## Adding a new service

1. Pick the primary domain users will recognise (e.g. `figma.com`, not `static.figma.com`).
2. Create the directory `services/<domain>/`.
3. Start with **Level 1** if that is all you have:

   ```
   services/<domain>/metadata.json
   ```

   The catalog welcomes Level 1 entries: knowing "this domain belongs to a known SaaS in category X" is already useful for downstream tools.
4. Upgrade to **Level 2** by adding `documents/tos.json` and `documents/privacy.json` (and `documents/dpa.json` when public).
5. Upgrade to **Level 3** by adding at least one file under `plans/` and a `recommended_settings.json`.
6. Update `metadata.last_updated` and bump `metadata.level` accordingly.

The `services/figma.com/` directory is a complete Level 3 example you can copy from.

## Updating an existing service

1. Open the relevant file(s) and make your changes.
2. If your change reflects a real-world event (vendor changed their ToS, added a plan, etc.), also add a file under `services/<domain>/history/`:

   ```
   services/<domain>/history/2026-05-13-ai-training-policy-tightened.json
   ```

   See `schemas/history_entry.schema.json` for the required fields.
3. Update `metadata.last_updated`.

## Validating your changes

Every file under `services/` must validate against the matching schema in `schemas/`. A minimal local check with `ajv`:

```sh
npx ajv-cli validate \
  -s schemas/metadata.schema.json \
  -d "services/*/metadata.json" \
  --spec=draft2020 --strict=false --all-errors
```

Repeat for `plan.schema.json` (against `services/*/plans/*.json`), `recommended_settings.schema.json`, `document.schema.json`, and `history_entry.schema.json`.

## Pull request checklist

Before opening a PR, please confirm:

- [ ] New or changed JSON validates against its schema.
- [ ] Slugs in `applicable_plans` exist in `plans/`.
- [ ] At least one `source_urls` entry per plan and per recommended setting.
- [ ] `last_verified` / `last_checked` / `last_updated` reflect today's date.
- [ ] No personal account data, screenshots, or secrets are included.
- [ ] Description of the PR cites the vendor pages you used.

## Licensing

By contributing, you agree that your contribution is licensed under CC BY-SA 3.0, the same license as the rest of the repository.
