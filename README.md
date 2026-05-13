# saas-policy-catalog

A community-maintained, JSON-only knowledge base of SaaS services and how their privacy and security settings should be configured.

The catalog is the public, machine-readable layer of the **SaaS Settings Hygiene Toolkit**: a small set of tools that help an individual or a small team keep the privacy and security posture of the SaaS products they use in an acceptable state over time.

## Why this exists

Most SaaS vendors expose a long tail of privacy and security toggles. The defaults are rarely what a privacy-conscious user would choose, the toggles move around, and the same vendor often applies different terms to different subscription plans (for example, an AI-training opt-out that is the default on Enterprise but opt-in on a free tier). Today this knowledge lives in scattered blog posts, help-center pages, and tribal knowledge. This repository is an attempt to make it explicit, versioned, and reusable.

The catalog is intentionally narrow:

- It only describes *what the vendor offers* and *what we recommend the user choose*.
- It does **not** include any user-specific state. Tools that consume the catalog (such as the local-sspm Chrome extension) hold the user's actual configuration on the user's own machine.

## Data depth: Level 1 / 2 / 3

Each service entry advertises how deeply it has been researched via the `level` field in `metadata.json`:

| Level | What is guaranteed                                                                  |
|-------|-------------------------------------------------------------------------------------|
| 1     | Domain(s), service name, vendor, broad category.                                    |
| 2     | Level 1 + URLs to current Terms of Service and Privacy Policy (and DPA if public).  |
| 3     | Level 2 + per-plan privacy conditions and a set of recommended in-product settings. |

Downstream tools should degrade gracefully: a Level 1 service can still be surfaced as "this domain you visit is a known SaaS", even if no recommendations are available yet.

## Repository layout

```
saas-policy-catalog/
  schemas/                     # JSON Schemas (Draft 2020-12) for every file type below
  services/
    <service-domain>/          # one directory per service, keyed by primary domain
      metadata.json            # required, conforms to schemas/metadata.schema.json
      plans/
        <plan-slug>.json       # zero or more, conforms to schemas/plan.schema.json
      recommended_settings.json # optional, conforms to schemas/recommended_settings.schema.json
      documents/
        tos.json               # optional, conforms to schemas/document.schema.json
        privacy.json
        dpa.json
        ...
      history/
        YYYY-MM-DD-<slug>.json # append-only change log, conforms to schemas/history_entry.schema.json
  docs/
    schema.md                  # field-by-field schema reference
    contributing.md            # how to add or update a service
```

All filenames use lowercase ASCII with hyphens. The service directory name (e.g. `figma.com`) should be the registrable domain users will most commonly see in their address bar.

## Using the schemas

The files in `schemas/` are standard JSON Schema Draft 2020-12 documents. They can be referenced directly from a checkout of this repository or, once published, by their `$id`. Example with `ajv`:

```js
import Ajv from "ajv";
import addFormats from "ajv-formats";
import metadataSchema from "./schemas/metadata.schema.json" with { type: "json" };

const ajv = addFormats(new Ajv({ allErrors: true, strict: false }));
const validate = ajv.compile(metadataSchema);

const data = JSON.parse(await fs.readFile("services/figma.com/metadata.json", "utf8"));
if (!validate(data)) console.error(validate.errors);
```

The same schemas are intended to be consumed by:

- the **saas-policy-catalog-data** pipeline that bootstraps and updates entries,
- the **local-sspm** browser extension that compares the user's live settings against `recommended_settings.json`.

## Bootstrapping from ToSDR

The initial population of this catalog is bootstrapped from [Terms of Service; Didn't Read](https://tosdr.org) data, which is licensed CC BY-SA 3.0. For that reason the catalog itself is also released under CC BY-SA 3.0 (see `LICENSE`), so that derivative entries remain redistributable on the same terms.

Bootstrapped entries start at **Level 1** (domain, name, category) or **Level 2** (with ToS/Privacy URLs) and are promoted to Level 3 as contributors add per-plan and recommended-setting information.

## Contributing

See `docs/contributing.md`. In short: pick a service, follow the existing `services/figma.com/` example, run the schemas against your JSON, open a pull request.

## License

CC BY-SA 3.0. See `LICENSE`.
