---
name: per-molecule-performance-aggregation
description: Use when you have mzRAPP benchmark (Benchmark.csv) and NPP tool outputs
  (e.g. XCMS unaligned/aligned CSVs) already matched into comparison_data and need
  stage-wise per-molecule metrics with uncertainty bounds rather than single point
  estimates.
when_to_use_negative:
- The NPP outputs have not yet been matched against the benchmark (no comparison_data
  exists) — run match_peaks_to_benchmark/compare_peaks first.
- You need sample-level or feature-table QC statistics rather than per-molecule reliability
  of a data pre-processing workflow.
- The input is a targeted/SRM quantitation table without mzRAPP benchmark boundaries
  — mzRAPP's metrics (found/not-found peaks, split peaks, degenerated IR) are defined
  only against its benchmark matching.
edam_topics:
- http://edamontology.org/topic_2269
tools:
- name: mzRAPP
  role: computes per-molecule performance metrics via derive_performance_metrics after
    benchmark matching
  repo: https://github.com/YasinEl/mzRAPP
- name: R
  role: implementation environment for the aggregation and bootstrap resampling
- name: boot
  role: R package used to draw molecule resamples and derive percentile 95% confidence
    intervals
- name: XCMS
  role: supplies the NPP outputs (unaligned and aligned run CSVs) that are matched
    to the benchmark
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bioinformatics/btab231
    title: 'mzRAPP: a tool for reliability assessment of data pre-processing in non-targeted
      metabolomics'
  source_content_hash: sha256:8bf56044a6fc47cd0cffd56356c32c947aca2272d308178bdcc6420f6d3cf0ea
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/YasinEl/mzRAPP
  tools:
  - R
  - XCMS
  - boot
  - mzRAPP
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# per-molecule-performance-aggregation

## Summary

Aggregates mzRAPP NPP performance metrics (found-peak fraction, isotopologue-ratio degeneration, split-peak %, alignment loss) into a molecules-by-metrics matrix and derives empirical 95% bootstrap confidence intervals by resampling benchmark molecules, as described for mzRAPP's reliability-assessment workflow.

## When to use

Use when you have mzRAPP benchmark (Benchmark.csv) and NPP tool outputs (e.g. XCMS unaligned/aligned CSVs) already matched into comparison_data and need stage-wise per-molecule metrics with uncertainty bounds rather than single point estimates.

## When NOT to use

- The NPP outputs have not yet been matched against the benchmark (no comparison_data exists) — run match_peaks_to_benchmark/compare_peaks first.
- You need sample-level or feature-table QC statistics rather than per-molecule reliability of a data pre-processing workflow.
- The input is a targeted/SRM quantitation table without mzRAPP benchmark boundaries — mzRAPP's metrics (found/not-found peaks, split peaks, degenerated IR) are defined only against its benchmark matching.

## Inputs

- Benchmark.csv (mzRAPP benchmark of ~47 target molecules with isotopologue peaks)
- XCMS_unaligned_run1.csv and XCMS_aligned_run1.csv (NPP tool outputs)
- comparison_data object built via match_peaks_to_benchmark/compare_peaks

## Outputs

- bootstrap_ci_metrics.csv with columns (metric, stage, estimate, ci_lower, ci_upper)
- Molecules-by-metrics matrix of per-molecule stage-wise performance metrics

## How to apply

Build comparison_data with mzRAPP's match_peaks_to_benchmark/compare_peaks, then call derive_performance_metrics(comparison_data) to compute per-molecule metrics for each pre-processing stage (pre-alignment, post-alignment). Assemble a molecules-by-metrics matrix where each of the ~47 benchmark molecules is a row and columns hold stage-wise metrics. Using the R boot package, draw B resamples with replacement of the molecules, recompute each metric as the mean over the resampled molecules per replicate, and extract percentile confidence intervals at alpha = 0.95 (2.5th and 97.5th percentiles). Report point estimate plus lower and upper CI bounds per metric and stage; bootstrapping over molecules is the rationale because metrics are averages over a finite benchmark set and their stability depends on the number of molecules.

## Related tools

- **mzRAPP** (computes per-molecule performance metrics via derive_performance_metrics after benchmark matching) — https://github.com/YasinEl/mzRAPP
- **R** (implementation environment for the aggregation and bootstrap resampling)
- **boot** (R package used to draw molecule resamples and derive percentile 95% confidence intervals)
- **XCMS** (supplies the NPP outputs (unaligned and aligned run CSVs) that are matched to the benchmark)

## Examples

```
ci <- boot.ci(boot(data = metric_matrix, statistic = function(d, i) mean(d[i, "found_frac"]), R = 1000), conf = 0.95, type = "perc")
```

## Evaluation signals

- Each metric's 95% bootstrap interval brackets its own point estimate (ci_lower <= estimate <= ci_upper).
- Reproduced ranges overlap the vignette values, e.g. 83-94% detected peaks and 28-53% degenerated isotopologue ratios in post-alignment.
- The molecules-by-metrics matrix contains one row per benchmark molecule (~47 rows for the deposited benchmark).
- The script runs end-to-end on the deposited inputs (Benchmark.csv, XCMS_unaligned_run1.csv, XCMS_aligned_run1.csv).
- Some variability across replicate runs is expected since resampling is random; identical CI bounds across seeds would be suspicious.

## Limitations

- The bootstrap is over a finite set of benchmark molecules (~47), so CI stability depends on benchmark size; more molecules are better.
- The paper describes the bootstrapping mechanism as part of the reliability workflow without separately reporting validation results for it.
- Metrics are only defined for the pre-processing stages mzRAPP can assess (found/not-found peaks, split peaks, missing values, degenerated IR, alignment errors).

## Evidence

- [abstract] mzRAPP produces several performance metrics for different steps of the pre-processing workflow: "mzRAPP produces several performance metrics for different steps of the pre-processing workflow"
- [readme] Only isotopologues for which the theoretically most abundant and at least one additional isotopologue are found are considered for the final benchmark.: [evidence span withheld — anchor status: external_doc]
- [abstract] Here, we present a tool named mzRAPP, which generates and validates a benchmark from user-supplied information: "Here, we present a tool named mzRAPP, which generates and validates a benchmark from user-supplied information"
