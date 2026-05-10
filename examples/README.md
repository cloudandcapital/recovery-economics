# Examples

This directory contains two scenario CSVs: the current backup configuration and a proposed reduced-retention configuration.

## Analyze a single scenario

```bash
recovery-economics analyze \
  --input examples/workload-config-sample.csv \
  --output-format json
```

The output includes `cost_per_gb` and `cost_per_backup` per workload so you can rank workloads by efficiency — useful when deciding which ones to prioritize for retention reduction.

## Compare two scenarios

```bash
recovery-economics compare \
  --baseline examples/workload-config-sample.csv \
  --proposed examples/proposed-config-sample.csv
```

Expected output:

```
Scenario comparison: examples/workload-config-sample.csv  vs  examples/proposed-config-sample.csv
Workload                   Baseline      Proposed         Delta
user-db                   $4993.20     $2500.20     -$2493.00
analytics-warehouse       $1766.40      $883.20      -$883.20
audit-logs                $3456.00     $1152.00     -$2304.00
media-assets               $386.40      $193.20      -$193.20
orders-api                 $277.60      $138.80      -$138.80
TOTAL                    $10879.60     $4867.40     -$6012.20
```
