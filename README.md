# dbt-hub-mirror

Byte-exact copies of nine public dbt Hub API files (hub.getdbt.com), so `dbt deps` can resolve the packages the `ade-bench` task suite declares — and everything those packages in turn depend on — from sandboxes whose egress allows GitHub but not hub.getdbt.com (Daytona Tier 1/2).

Use: `DBT_PACKAGE_HUB_URL=https://raw.githubusercontent.com/evanokeefe39/dbt-hub-mirror/<commit>/`. Package tarballs still download from codeload.github.com.

| file | fetched |
|---|---|
| `api/v1/index.json` | 2026-10-09 |
| `api/v1/dbt-labs/dbt_utils.json` | 2026-10-09 |
| `api/v1/fivetran/asana_source.json` | 2026-10-16 |
| `api/v1/fivetran/quickbooks_source.json` | 2026-10-16 |
| `api/v1/fivetran/intercom_source.json` | 2026-10-16 |
| `api/v1/fivetran/fivetran_utils.json` | 2026-10-17 |
| `api/v1/calogica/dbt_expectations.json` | 2026-10-17 |
| `api/v1/calogica/dbt_date.json` | 2026-10-17 |
| `api/v1/fishtown-analytics/dbt_utils.json` | 2026-10-17 |
| `api/v1/fishtown-analytics/spark_utils.json` | 2026-10-17 |

Every version's `downloads.tarball` resolves to `codeload.github.com`, which the sandbox already reaches — only the *index and metadata* lookups were ever blocked, which is why mirroring the metadata is enough.

## Why these files

Six of the 48 trials in the 2026-10-16 Daytona full suite scored 0 having executed **no test at all**: their task's `tests/test-setup.sh` runs a bare `dbt deps`, and the package it declares was not here — `fivetran/asana_source` for `asana002`, `asana005` and `asana005-hard`, `fivetran/quickbooks_source` for `quickbooks002`, `quickbooks003` and `quickbooks004`. Each carried a 404 at line 4 of its `verifier/test-stdout.txt` beside `actual_test_total=0`.

The 95 `packages.yml` files across the suite's 48 task checkouts declare exactly four packages — `dbt-labs/dbt_utils` (69), `fivetran/asana_source` (12), `fivetran/quickbooks_source` (8), `fivetran/intercom_source` (6) — so three of the four were absent. `intercom_source` is mirrored too even though `intercom001` happened to pass without it, because its task moves to a bare `dbt deps` as easily as any other.

Mirroring only those three was **not enough**, and the six-task re-run of 2026-10-17 is what showed it: the packages resolved, then `dbt deps` walked into their dependencies and five of the six trials 404'd again, this time on `fivetran/fivetran_utils.json` — the one package missing from the transitive closure:

| package | depends on |
|---|---|
| `fivetran/asana_source` | `fivetran/fivetran_utils` |
| `fivetran/quickbooks_source` | `fivetran/fivetran_utils` |
| `fivetran/intercom_source` | `fivetran/fivetran_utils`, `calogica/dbt_expectations` (older versions) |
| `fivetran/fivetran_utils` | `dbt-labs/dbt_utils` (newer versions), `fishtown-analytics/dbt_utils` (older versions) |
| `calogica/dbt_expectations` | `calogica/dbt_date`, `fishtown-analytics/spark_utils` |
| `calogica/dbt_date` | `dbt-labs/dbt_utils`, `fishtown-analytics/dbt_utils`, `fishtown-analytics/spark_utils` |

The closure is computed over **every version** of every package, not just `latest`, because a resolver that picks an older release follows that release's own requirements — which is how the legacy `fishtown-analytics/*` namespaces enter it. More files than a given task resolves are harmless; one missing file is a zero.

The copies are byte-identical to the hub's, checked by sha256 at fetch time.

## Known limit: the index advertises more than this repo serves

`index.json` is the hub's **full** index — 376 entries — while ten files are served. A task needing any other package still 404s. That is deliberate: the index is copied whole so it stays a faithful mirror of the hub's shape, and the answer to a missing package is to add that package here the same way, not to shrink the index.

Because a consumer pins a **commit**, adding files here does not change what an existing pin serves. A job that wants them must move its pin to the commit that carries them.
