# bigquery_cmd

## Summary
Provides a workspace-scoped skill for running quick BigQuery queries using the `bq` CLI with Application Default Credentials (ADC). Helps data explorers run ad-hoc queries, extract samples, and troubleshoot auth/performance issues with minimal setup.

## Intent
- Run `bq` commands authenticated via ADC
- Extract sample rows, inspect schemas, and run small aggregations
- Prefer fast reads (Storage API / `bq head` / limited SELECT) and surface fallback guidance when queries require full scans

## Requirements
- `gcloud` and `bq` CLIs installed and in PATH
- ADC set up via `gcloud auth login` or `GOOGLE_APPLICATION_CREDENTIALS` env pointing to a service account key

## Usage Patterns
- Sample extraction:
  - `bq query --use_legacy_sql=false --format=csv 'SELECT col1,col2 FROM `project.dataset.table` LIMIT 100' > sample.csv`
  - Use `--dry_run` or `--max_rows` when appropriate
- Schema inspection:
  - `bq show --schema --format=prettyjson project:dataset.table`
- Fast row preview (small, cheap):
  - `bq head --max_rows=20 project:dataset.table`

## Prompts / Example Requests
- "Get 200 sample rows for `project.dataset.table` selecting columns `a,b,c` and save to outputs/sample.csv."
- "Show me the schema for `dataset.table` and highlight nullable columns."
- "Run a dry-run for this query and return estimated bytes processed: SELECT ..."

## Behavior & Decisions
- If ADC is not available, the skill instructs how to run `gcloud auth login` or set `GOOGLE_APPLICATION_CREDENTIALS`.
- For native BigQuery tables, recommend `bq head` or small `SELECT` with `LIMIT` to avoid scanning large amounts.
- For views / federated sources, warn that `bq` will scan underlying sources and suggest adding LIMIT, filters, or using partition/preview techniques.

## Safety & Cost Guidance
- Always prefer `--dry_run` to estimate bytes processed on larger queries.
- Use `LIMIT`, `WHERE`, or `SELECT`-restricted columns for exploratory queries.

## Troubleshooting
- Auth errors: run `gcloud auth login` or set `GOOGLE_APPLICATION_CREDENTIALS`.
- Permission denied: request `bigquery.jobs.create` and `bigquery.tables.get` on the project/dataset.
- Slow queries: check if target is a view or external table; if so, recommend materializing or sampling with filters.

## Example workflows to save as quick prompts
- "Sample table columns a,b,c from project.dataset.table with LIMIT 200 and save as CSV"
- "Estimate bytes for this query: SELECT ..."

## Where to store output
- Recommend `outputs/samples/` for CSV samples and include file naming conventions: `project_dataset_table_sample.csv`.

## Related skills
- `agent-customization` — guidelines for writing SKILL.md files
