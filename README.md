# Recovery Economics

[![MIT License](https://img.shields.io/badge/license-MIT-blue.svg)](LICENSE)
[![Multi-cloud](https://img.shields.io/badge/cloud-AWS%20%7C%20Azure%20%7C%20GCP-orange)](https://github.com/cloudandcapital/recovery-economics)

**FinOps decision-stress model — estimate the true monthly cost and time of cloud recovery, per workload.**

Part of the [Cloud & Capital](https://github.com/cloudandcapital) FinOps pipeline.  
Resilience cost output feeds into [Cloud Cost Guard](https://github.com/cloudandcapital/cloud-cost-guard) — the unified FinOps dashboard.

---

**Features:**
- CSV-in, machine-readable output — no cloud API calls, no setup friction
- Per-workload monthly resilience cost: backup storage + restore compute + egress
- Scenario comparison — compare two configurations side by side with delta analysis
- RPO/RTO modeling — surface the cost of meeting your recovery objectives
- JSON, YAML, and CSV output — pipe-friendly with explicit exit codes
- Evals mode for automated validation of model assumptions

---

## Install

```bash
pip install "git+https://github.com/cloudandcapital/recovery-economics.git"
# or
pipx install .
```

---

## Usage

```bash
# Model recovery cost for a set of workloads
recovery-economics model --workload workloads.csv

# Compare two recovery configurations
recovery-economics compare --baseline current.csv --alternative proposed.csv

# JSON output (feeds Cloud Cost Guard report.json)
recovery-economics model --workload workloads.csv --format json

# YAML output
recovery-economics model --workload workloads.csv --format yaml
```

---

## Input CSV Format

One row per workload:

| Column | Description |
|--------|-------------|
| `workload` | Workload name |
| `data_gb` | Total data size in GB |
| `backup_frequency_per_month` | Number of backup cycles per month |
| `backup_storage_tier` | `standard`, `infrequent`, or `archive` |
| `restore_compute_hours` | Expected compute hours per restore |
| `egress_gb_per_restore` | Data egress per restore in GB |
| `rpo_hours` *(optional)* | Recovery point objective |
| `rto_hours` *(optional)* | Recovery time objective |

---

## Output

```json
{
  "total_workloads": 5,
  "total_monthly_resilience_cost": 4820.50,
  "top_workloads": [
    {
      "workload": "prod-postgres",
      "monthly_storage_cost": 1240.00,
      "monthly_restore_cost": 380.00,
      "total_monthly_resilience_cost": 1620.00,
      "rpo_hours": 4,
      "rto_hours": 2
    }
  ]
}
```

---

## Part of the Cloud & Capital Pipeline

| Tool | Role |
|------|------|
| [FinOps Lite](https://github.com/cloudandcapital/finops-lite) | Cost pull + FOCUS 2026 export |
| [FinOps Watchdog](https://github.com/cloudandcapital/finops-watchdog) | Anomaly detection |
| **Recovery Economics** | Resilience cost modeling |
| [Cloud Cost Guard](https://github.com/cloudandcapital/cloud-cost-guard) | Unified dashboard |
| [AI Cost Lens](https://github.com/cloudandcapital/ai-cost-lens) | AI/LLM spend tracking |
| [SaaS Cost Analyzer](https://github.com/cloudandcapital/saas-cost-analyzer) | SaaS license governance |
| [Tech Spend Command Center](https://github.com/cloudandcapital/tech-spend-command-center) | Executive reporting |

---

## License

MIT © 2025 Diana Molski, Cloud & Capital
