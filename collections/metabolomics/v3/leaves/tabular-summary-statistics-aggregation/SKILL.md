---
name: tabular-summary-statistics-aggregation
description: Use when a benchmark/runtimes table must be reduced to per-tool summary
  statistics (means, speedup fold-change) and cross-checked against values reported
  in a publication's figure or abstract.
when_to_use_negative:
- The goal is to benchmark annotation accuracy or coverage (annotated metabolite counts)
  rather than computational runtime — different source data columns are needed
- Only a single tool's runtimes are available with no paired MetDNA2/MetDNA3 dataset
  column, so no fold-speedup can be computed
edam_operation: http://edamontology.org/operation_0306
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: pandas
  role: Loads the Source Data runtime table, drops incomplete entries, and aggregates
    per-tool runtimes to compute mean per-dataset runtimes and the speedup factor
- name: matplotlib
  role: Plots the per-dataset runtimes as a grouped bar chart replicating Fig. 2g
- name: MetDNA3
  role: Subject of the benchmark; its runtimes (≈77 min per dataset) are aggregated
    and can be spot-checked via the MrnAnnoAlgo3 pipeline on a demo dataset
  repo: https://github.com/ZhuMetLab/MrnAnnoAlgo3
- name: MetDNA2
  role: Baseline tool whose runtimes (≈1082 min per dataset) form the denominator
    of the >14-fold speedup comparison
  repo: https://github.com/ZhuMetLab/MetDNA2
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-025-63536-6
    title: Knowledge and data-driven two-layer networking for accurate metabolite
      annotation in untargeted metabolomics
  source_content_hash: sha256:4865bda54f2d1c2693ab336c76252fab90f1c67e0cf6d9b3ac2dceaa739813b1
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/ZhuMetLab/MrnAnnoAlgo3
  repo_url: https://github.com/ZhuMetLab/MrnAnnoAlgo3
  tools:
  - MetDNA2
  - MetDNA3
  - devtools
  - matplotlib
  - pandas
  edam_operation: http://edamontology.org/operation_0306
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# tabular-summary-statistics-aggregation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Aggregates per-dataset wall-clock runtime tables (e.g. MetDNA2 vs MetDNA3 across 12 datasets from 3 biological samples) into per-tool mean runtimes and a fold-speedup statistic, verifying that computed summaries match the reported benchmark (~1082 min vs ~77 min, >14-fold) within rounding tolerance.

## When to use

Use when a benchmark/runtimes table must be reduced to per-tool summary statistics (means, speedup fold-change) and cross-checked against values reported in a publication's figure or abstract.

## When NOT to use

- The goal is to benchmark annotation accuracy or coverage (annotated metabolite counts) rather than computational runtime — different source data columns are needed
- Only a single tool's runtimes are available with no paired MetDNA2/MetDNA3 dataset column, so no fold-speedup can be computed

## Inputs

- Source Data runtime table for Fig. 2g (wall-clock runtimes per dataset per tool, loadable with pandas)
- Optional: demo dataset from the MetDNA webserver for a single-dataset runtime spot-check

## Outputs

- Per-tool mean per-dataset runtime summary (MetDNA2 ≈ 1082 min; MetDNA3 ≈ 77 min)
- Fold-speedup statistic (>14x, computed as MetDNA2 mean / MetDNA3 mean)
- Grouped bar chart of per-dataset runtimes replicating Fig. 2g

## How to apply

Download the Source Data file accompanying the figure (here Source Data for Fig. 2g, DOI 10.1038/s41467-025-63536-6) and load the runtime table with pandas. Verify the table contains wall-clock runtimes for MetDNA2 and MetDNA3 over 12 datasets drawn from 3 biological samples and drop incomplete entries before aggregating. Compute per-dataset means per tool (expected ≈1082 min for MetDNA2 vs ≈77 min for MetDNA3) and the fold speedup (1082/77 ≈ 14.1, expected >14). Optionally spot-check the headline number by re-running the pipeline — e.g. installing MrnAnnoAlgo3 from GitHub and processing one demo dataset from the MetDNA webserver, which should complete in about one hour. Reproduce the figure as a grouped bar chart with matplotlib and write out both the summary statistics and the figure file.

## Related tools

- **pandas** (Loads the Source Data runtime table, drops incomplete entries, and aggregates per-tool runtimes to compute mean per-dataset runtimes and the speedup factor)
- **matplotlib** (Plots the per-dataset runtimes as a grouped bar chart replicating Fig. 2g)
- **MetDNA3** (Subject of the benchmark; its runtimes (≈77 min per dataset) are aggregated and can be spot-checked via the MrnAnnoAlgo3 pipeline on a demo dataset) — https://github.com/ZhuMetLab/MrnAnnoAlgo3
- **MetDNA2** (Baseline tool whose runtimes (≈1082 min per dataset) form the denominator of the >14-fold speedup comparison) — https://github.com/ZhuMetLab/MetDNA2

## Examples

```
import pandas as pd, matplotlib.pyplot as plt; df = pd.read_excel('SourceData_fig2g.xlsx').dropna(); means = df.groupby('tool')['runtime_min'].mean(); print(means, means['MetDNA2']/means['MetDNA3'])
```

## Evaluation signals

- Computed mean per-dataset runtime for MetDNA2 matches the reported 1082 min within rounding tolerance
- Computed mean per-dataset runtime for MetDNA3 matches the reported 77 min within rounding tolerance
- Fold speedup (MetDNA2 mean / MetDNA3 mean) exceeds 14, consistent with the abstract's 'over 10-fold' claim being conservative
- Input table verified to span 12 datasets from 3 biological samples; no missing rows after dropping incomplete entries
- Both a summary statistics file and a grouped bar chart figure file are written

## Limitations

- The abstract and discussion state the efficiency improvement as 'over 10-fold', while the exact figure-level benchmark is >14-fold; reported magnitudes differ by granularity
- Wall-clock runtimes depend on the number of features and MS/MS spectra; the README for MetDNA2 notes 4–8 h runtimes vary with dataset size, so single-run spot-checks may deviate from the mean
- Runtimes in the table are dataset-level aggregates and do not capture variance in hardware or software versions used in the original benchmark

## Evidence

- [results] MetDNA3 reduced the mean computation time per dataset from 1082 minutes (MetDNA2) to 77 minutes: [evidence span withheld — anchor status: not_found]
