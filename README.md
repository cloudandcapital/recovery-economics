# Recovery Economics

[![CI](https://github.com/cloudandcapital/recovery-economics/actions/workflows/test.yml/badge.svg)](https://github.com/cloudandcapital/recovery-economics/actions/workflows/test.yml)

**Part of the Visibility → Variance → Tradeoffs pipeline.**

| Tool | Role | Repo |
|------|------|------|
| FinOps Lite | Cost visibility — AWS/Azure/GCP spend, FOCUS 1.0 export | [cloudandcapital/finops-lite](https://github.com/cloudandcapital/finops-lite) |
| FinOps Watchdog | Anomaly detection — spend spikes from any cost CSV | [cloudandcapital/finops-watchdog](https://github.com/cloudandcapital/finops-watchdog) |
| **Recovery Economics** | Resilience modeling — backup/restore cost + scenario compare | [cloudandcapital/recovery-economics](https://github.com/cloudandcapital/recovery-economics) |
| AI Cost Lens | AI spend observability — model-level cost across OpenAI, Anthropic, Bedrock | [cloudandcapital/ai-cost-lens](https://github.com/cloudandcapital/ai-cost-lens) |
| SaaS Cost Analyzer | SaaS spend governance — unused licenses, per-seat costs, forecasting | [cloudandcapital/saas-cost-analyzer](https://github.com/cloudandcapital/saas-cost-analyzer) |
| Cloud Cost Guard | Dashboard — spend trends, savings coverage, rightsizing | [cloudandcapital/cloud-cost-guard](https://github.com/cloudandcapital/cloud-cost-guard) |
| Tech Spend Command Center | Executive summary — unified Cloud+AI+SaaS report | [cloudandcapital/tech-spend-command-center](https://github.com/cloudandcapital/tech-spend-command-center) |

Six tools. One pipeline. Full Cloud+AI+SaaS coverage for every scope the FinOps Foundation 2026 Framework defines.

---

**Recovery Economics** is a deterministic CLI for modeling monthly resilience cost from backup/restore configuration data — and comparing cost deltas between scenarios (e.g. current vs. proposed retention policy).

## v0.1 Scope

- CSV in
- machine-readable output (JSON, YAML, or CSV)
- `--compare` flag for side-by-side scenario comparison
- no cloud API calls
- explicit exit codes

## Analyze Command

Given a CSV with one row per workload, the CLI computes per-workload and aggregate monthly cost.

### Required Input Columns

- `workload` (or custom name via `--workload-column`)
- `data_gb`
- `backup_frequency_per_month`
- `retention_months`
- `storage_rate_per_gb_month`
- `restore_gb_per_month`
- `restore_rate_per_gb`

### Formulas

For each workload:

- `effective_backups_kept = backup_frequency_per_month * retention_months`
- `monthly_storage_cost = data_gb * backup_frequency_per_month * retention_months * storage_rate_per_gb_month`
- `monthly_restore_cost = restore_gb_per_month * restore_rate_per_gb`
- `total_monthly_resilience_cost = monthly_storage_cost + monthly_restore_cost`

Costs are rounded to 2 decimals.

## CLI Contract

### Analyze a Single Scenario

```bash
recovery-economics analyze \
  --input tests/fixtures/simple_config.csv \
  --output-format json
```

`python -m recovery_economics ...` is also supported.

### Compare Two Scenarios

Use `--compare` to show the monthly cost delta between two CSV configurations side by side — useful for evaluating changes to retention policy, backup frequency, or storage tier:

```bash
recovery-economics compare \
  --baseline current_config.csv \
  --proposed reduced_retention.csv
```

Output shows per-workload costs for each scenario and the delta (proposed minus baseline), sorted by absolute delta descending.

### Flags

**analyze** — Required:

- `--input` : path to CSV file
- `--output-format` : `json` | `yaml` | `csv`

Optional:

- `--workload-column` : defaults to `workload`

**compare** — Required:

- `--baseline` : path to baseline CSV file
- `--proposed` : path to proposed CSV file

Optional:

- `--workload-column` : defaults to `workload`

## Output Contract

### JSON / YAML

`json` and `yaml` emit the same structure:

- `schema_version` (constant: `"1.0"`)
- `metadata.generated_at` (UTC ISO-8601)
- `metadata.input_file`
- `summary`
- `workloads` (array)

### CSV

`csv` emits one row per workload with:

- `workload`
- `monthly_storage_cost`
- `monthly_restore_cost`
- `total_monthly_resilience_cost`

## Exit Codes

- `0` = success
- `2` = CLI usage error (missing/invalid flags)
- `3` = input file error (not found/unreadable)
- `4` = schema/data error (missing columns, non-numeric values, invalid values)
- `5` = internal/runtime error

## Development

Install locally:

```bash
pip install -e .
```

Run tests:

```bash
pytest
```

## Pipeline

Recovery Economics is step three. Typical flow:

1. **[FinOps Lite](https://github.com/cloudandcapital/finops-lite)** exports a FOCUS 1.0 CSV from AWS Cost Explorer
2. **[FinOps Watchdog](https://github.com/cloudandcapital/finops-watchdog)** detects anomalies in that CSV
3. **Recovery Economics** models resilience cost and compares scenarios
4. **[Cloud Cost Guard](https://github.com/cloudandcapital/cloud-cost-guard)** surfaces all of this in a dashboard
