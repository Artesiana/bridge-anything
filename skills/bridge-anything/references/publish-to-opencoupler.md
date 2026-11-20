# Publish to OpenCoupler

Read for step 7. OpenCoupler (https://opencoupler.com) is a marketplace for integrations into on-premise and legacy software. Providers list an integration per system; buyers compare them, contact the provider on the platform, and request systems nobody covers yet. A human reviews every listing before it goes public.

## One-time setup (the user does this in the browser)

1. Create an account at https://opencoupler.com/sign-up and choose "offering" in onboarding.
2. Create the provider profile at https://opencoupler.com/app/provider.
3. Create an API key at https://opencoupler.com/app/settings with the scope `write:listings` (add `write:requests` to propose a missing system). Store it as `OPENCOUPLER_API_KEY`.

## Connect the MCP server

Claude Code:

```bash
claude mcp add --transport http opencoupler https://opencoupler.com/mcp \
  --header "Authorization: Bearer $OPENCOUPLER_API_KEY"
```

Codex (`~/.codex/config.toml`):

```toml
[mcp_servers.opencoupler]
url = "https://opencoupler.com/mcp"
bearer_token_env_var = "OPENCOUPLER_API_KEY"
```

Without MCP, the same operations exist over REST at `https://opencoupler.com/api/v1` (OpenAPI document at `/api/v1/openapi.json`) with the header `Authorization: Bearer $OPENCOUPLER_API_KEY`. Use the server's own tool schemas over this file when they differ.

## Steps

1. `search_systems` with the system's name. Then `get_system` with its slug for the `systemId` and the data-area keys.
2. When the system is not in the catalog: `create_request` with `proposedSystem` (name, vendor, country, website) and the use case. Tell the user the listing can be created once the system is published, and stop here.
3. Map `BRIDGE.md` to the listing document (table below) and show the full document to the user.
4. `upsert_listing` with a stable `slug` (the system slug works) and the document. This saves a private working copy.
5. Ask the user to confirm that every statement is accurate and that they accept the marketplace terms (https://opencoupler.com/legal/terms). Only after a clear yes: `submit_listing` with `confirm: true`.
6. `list_my_listings`: the listing shows `in_review`. Reviewer feedback arrives in `reviewNote`; fix, `upsert_listing`, submit again.

## Field mapping

| Listing field | Source | Allowed values |
| --- | --- | --- |
| `status` | what step 5 delivered | `live` (customers use it), `beta` (works, few installs), `in_development` (with `expectedAt`, first day of the quarter) |
| `capabilities` | the access-path table, per data-area key from `get_system` | `none`, `read`, `read_write` |
| `accessMethod` | the main access path | `vendor_api_official`, `vendor_api_partner`, `local_database_readonly`, `file_export`, `ui_automation`, `other` |
| `writeSafeguards` | the bridge's write mode | `none`, `user_confirmation`, `approval_workflow` |
| `onSite` | the installer | `required`, `os`, `note` |
| `interfaces` | what the bridge exposes | `rest`, `mcp`, `cli`, `webhooks`, `sdk`, each with a `docsUrl` |
| `authMethods` | the bridge's client auth | `api_key`, `oauth`, `mtls`, `other` |
| `hosting` | where the bridge runs | `mode`: `provider_cloud`, `customer_on_prem`, `both`; `regions` |
| `dataHandling` | `DATA-PROTECTION.md` | `mode`: `pass_through`, `cached`, `stored`; `retentionDays`; `subprocessors` |
| `compliance` | only claims backed by a document the user holds | `type`, `scope`, `validUntil`, `evidenceUrl` |
| `pricing` | the user's actual price | `model`: `per_system_month`, `per_site_month`, `per_user_month`, `usage`, `one_time`, `free`, `on_request`; `currency`: `EUR`, `USD`, `CHF`, `GBP`; `billingPeriod`: `month`, `year`; `usageUnit`: `1k_calls`, `gb`; amounts in minor units |
| `onboarding` | the install steps | `goLive`: `same_day`, `days_1_5`, `weeks_1_4`, `longer`; up to ten `steps` |
| `summary`, `body` | `BRIDGE.md`, in plain words for a buyer | |

A listing states what exists today. Planned data areas go in `body` under a "Planned" heading, not into `capabilities`. A certification, customer or install count appears only when the user can show it.
