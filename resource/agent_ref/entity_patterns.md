# Entity Delegate Patterns (from HTML Data Dictionary)

Read the entity's `delegates to` line from the HTML page header to determine its pattern.
This drives which BQ columns exist and which filter to apply.

| Delegates to (in HTML) | Pattern Label | `BranchID`? | `FixedID`? | `Retired`? | `[cdc_meta]___operation`? | BQ active-row filter (ingress) | BQ active-row filter (curated) |
|---|---|---|---|---|---|---|---|
| `EffDated` (or `EffDatedBranch`) | **EffDated** | ✅ | ✅ | ❌ (no `retired` in D2) | ❌ | `Retired = 0 AND BranchID <> 0` | `beanversion = 0 AND branchid <> 0 AND dbt_valid_to IS NULL` |
| `Retireable` + `SimpleEffDated` | **SimpleEffDated config** | ❌ | ❌ | ✅ | ❌ | `Retired = 0` | `retired = 0 AND dbt_valid_to IS NULL` |
| `Retireable` only | **Retireable** | ❌ | ❌ | ✅ | ❌ | `Retired = 0` | `retired = 0` |
| Neither (no `Retireable`) | **Non-retireable** | ❌ | ❌ | ❌ | ❌ | *(no filter)* | `dbt_valid_to IS NULL` |
| `Versionable` (CDC-ingested) | **Versionable / CDC** | ❌ | ❌ | ❌ | ✅ | `[cdc_meta]___operation NOT IN (1)` | `dbt_valid_to IS NULL AND [cdc_meta]___operation NOT IN (1)` |

## Critical Rule

**CDC detection takes precedence over entity pattern.** If `[cdc_meta]___operation` is present in the BQ ingress table, always use `[cdc_meta]___operation NOT IN (1)` as the active-row filter regardless of what the data dictionary says about `Retired` or `beanversion`. The `beanversion` column is a GW optimistic-lock counter and must **never** be used as an active-row filter.

## Key Examples (confirmed from HTML dictionary)

- `pcx_producercommission_ext` → `Retireable` + `SimpleEffDated` → **no** `BranchID`/`FixedID`; standalone config table
- `pcx_productvariant_ext` → `Retireable` + `SimpleEffDated` → **no** `BranchID`/`FixedID`; standalone config table
- `pc_policyperiod` → `EffDatedBranch` (root branch) → **has** `BranchID`/`FixedID`
- `pc_policyline` → `EffDated` → **has** `BranchID`, `FixedID`, `BranchValue` (FK to `pc_policyperiod`)
- `pcx_cmlvehicle` → `EffDated` → **has** `BranchID`, `FixedID`
- `pcx_cmlvehmanuscriptform_ext` → `EffDated` → **has** `BranchID`, `FixedID`

## Virtual Properties

The HTML page marks many fields as `(virtual property)`. These have **no physical DB column**. Always skip them when assessing the BQ schema. Examples: anything ending in `_PROP`, `_DYNPROP`, derived `boolean` properties, computed arrays. Only `(non-null)`, `(writable)`, `(exportable)`, or `(loadable)` fields are physical columns.
