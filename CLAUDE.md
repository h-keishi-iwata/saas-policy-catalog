# saas-policy-catalog

Public knowledge base: SaaS service x settings x recommended values.

## Structure

```
services/{domain}/
  metadata.json               Level 1+
  documents/{tos,privacy,dpa}.json  Level 2+
  plans/{plan_name}.json      Level 3
  recommended_settings.json   Level 3
  history/                    Append-only changelog
schemas/                      JSON Schema Draft 2020-12
```

## Levels

- **Level 1**: domain + service name + category (ToSDR bootstrap)
- **Level 2**: + document URLs (ToS, Privacy, DPA)
- **Level 3**: + plans, recommended settings, DOM selectors

## Validation

```sh
npx ajv validate -s schemas/metadata.schema.json -d 'services/*/metadata.json'
```

## Branch conventions

- `main` — stable, reviewed data
- `claude/*` — Claude Code scheduled task PRs

## Cross-repo dependencies

- **saas-policy-catalog-data** writes Level 1/2 entries here via `make bootstrap`
- **local-sspm** reads `services/*/metadata.json` to match visited domains (Phase 2+)

## License

CC BY-SA 3.0 (derivative of ToSDR data)
