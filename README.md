# dbt-hub-mirror

Byte-exact copies of two public dbt Hub API files (hub.getdbt.com, fetched 2026-10-09), so `dbt deps` can resolve `dbt-labs/dbt_utils` from sandboxes whose egress allows GitHub but not hub.getdbt.com (Daytona Tier 1/2).

Use: `DBT_PACKAGE_HUB_URL=https://raw.githubusercontent.com/evanokeefe39/dbt-hub-mirror/<commit>/`. Package tarballs still download from codeload.github.com.
