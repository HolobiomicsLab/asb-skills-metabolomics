---
name: automated-vs-manual-pipeline-comparison
description: Use when you must reproduce or verify an efficiency claim comparing an
  automated pipeline to a manual workflow, e.g. metScribeR's ~10 s per standard and
  26-fold time reduction over 603 IROA standards.
when_to_use_negative:
- The goal is accuracy comparison (e.g. RT correlation of 0.99 between manual and
  metScribeR-derived RTs) rather than processing-time benchmarking
- Input is raw LC-MS mzML data to be processed into a library rather than published
  timing figures to be recomputed
- The comparison involves a fully automated pipeline with no manual review step, so
  an automated+manual combined time cannot be constructed
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: metScribeR
  role: Semiautomated pipeline whose processing time (~31 min automated + ~72 min
    visual inspection for 603 standards, ~10 s per standard) is benchmarked against
    the manual baseline
  repo: https://github.com/ncats/metScribeR
- name: R
  role: Language used to compute baseline totals, derived fold reduction, and to write
    timing_benchmark_summary.csv; metScribeR itself is an R package
- name: MassHunter
  role: Manual MassHunter-based approach serving as the baseline at ~4.5 min per compound
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1021/acs.jproteome.5c00548
    title: 'MetScribeR: A Semiautomated Tool for Data Processing of In-House LC-MS
      Metabolite Reference Libraries'
  source_content_hash: sha256:ddd8d17989dcc52c472cdbefa66eaa278f4b2c8949ffbcd4659d84628364ac7f
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/ncats/metScribeR
  repo_url: https://github.com/ncats/metScribeR
  tools:
  - MassHunter
  - R
  - metScribeR
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# automated-vs-manual-pipeline-comparison

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A benchmarking method that quantifies the time savings of a semiautomated LC-MS library-building workflow (metScribeR) against a manual tool-based baseline (MassHunter at ~4.5 min/compound) by recomputing per-standard effort, total automated plus manual-review time, and the derived fold reduction in total processing time.

## When to use

Use when you must reproduce or verify an efficiency claim comparing an automated pipeline to a manual workflow, e.g. metScribeR's ~10 s per standard and 26-fold time reduction over 603 IROA standards.

## When NOT to use

- The goal is accuracy comparison (e.g. RT correlation of 0.99 between manual and metScribeR-derived RTs) rather than processing-time benchmarking
- Input is raw LC-MS mzML data to be processed into a library rather than published timing figures to be recomputed
- The comparison involves a fully automated pipeline with no manual review step, so an automated+manual combined time cannot be constructed

## Inputs

- Reported benchmark figures from the article (603 IROA compounds, ~10 s per standard, ~31 min automated, ~72 min manual inspection, 4.5 min/compound manual MassHunter baseline)
- R data frame assembling the timing metrics
- Optional: README timing figure (~30 min for 12000 mzML files) for cross-checking automated-step throughput

## Outputs

- timing_benchmark_summary.csv with one row per metric: per-standard effort, total automated time, total manual review time, total baseline time, fold reduction

## How to apply

Load the reported benchmark figures into an R data frame: 603 IROA compounds, ~10 s combined computational and manual effort per metScribeR standard, ~31 min total automated processing, ~72 min manual visual peak inspection, and a 4.5 min per compound manual MassHunter baseline. Compute the manual baseline total as 603 × 4.5 min (≈2713.5 min) and the metScribeR total as 31 + 72 min (≈103 min), then derive the fold reduction as baseline/metScribeR total. Validate that the recomputed fold reduction falls between 25 and 27 (article reports 26) and that per-standard combined effort rounds to ~10 s (103 min × 60 s / 603 standards ≈ 10 s). Write one row per metric (per-standard effort, total automated time, total manual review time, total baseline time, fold reduction) to timing_benchmark_summary.csv. Note the automated step alone was reported as ~30 min for 12000 mzML files in the README, so per-file scaling should be sanity-checked against the article's 603-standard figure.

## Related tools

- **metScribeR** (Semiautomated pipeline whose processing time (~31 min automated + ~72 min visual inspection for 603 standards, ~10 s per standard) is benchmarked against the manual baseline) — https://github.com/ncats/metScribeR
- **R** (Language used to compute baseline totals, derived fold reduction, and to write timing_benchmark_summary.csv; metScribeR itself is an R package)
- **MassHunter** (Manual MassHunter-based approach serving as the baseline at ~4.5 min per compound)

## Examples

```
metrics <- data.frame(n=603, per_std_s=10, auto_min=31, inspect_min=72, manual_per_cmpd_min=4.5); metrics$baseline_min <- metrics$n*metrics$manual_per_cmpd_min; metrics$metScribeR_min <- metrics$auto_min+metrics$inspect_min; metrics$fold_reduction <- metrics$baseline_min/metrics$metScribeR_min; write.csv(metrics,'timing_benchmark_summary.csv',row.names=FALSE)
```

## Evaluation signals

- Recomputed fold reduction (manual baseline total / metScribeR total) must fall between 25 and 27, matching the reported 26-fold reduction
- Per-standard combined effort must round to approximately 10 s
- Manual baseline total equals 603 compounds × 4.5 min/compound ≈ 2713.5 min
- metScribeR total time equals sum of automated (~31 min) and manual inspection (~72 min) ≈ 103 min
- timing_benchmark_summary.csv contains exactly one row per metric with all five metrics present

## Limitations

- The 4.5 min/compound manual baseline is an estimated figure, not a measured per-compound timing on the same 603 standards
- Fold reduction depends on the number of compounds; with far fewer standards the fixed ~31 min automated computation dominates and the ratio shrinks
- The README reports ~30 min for 12000 mzML files for the initial computation, which does not directly reconcile with the article's 603-standard benchmark without accounting for per-file versus per-standard differences
- Manual visual inspection time (~72 min) is still required, so the workflow is semiautomated, not fully hands-off

## Evidence

- [results] Benchmark task finding: [evidence span withheld — anchor status: not_found]
