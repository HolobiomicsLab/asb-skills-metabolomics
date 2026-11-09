---
name: bootstrap-confidence-interval-estimation
description: Use when NPP performance metrics (found peaks, split peaks, alignment
  loss, isotopologue-ratio degeneration) computed against an mzRAPP benchmark need
  point estimates with bootstrap CI bounds per pre-processing stage.
when_to_use_negative:
- You need parametric or assumption-based intervals for a single statistic unrelated
  to benchmark-molecule resampling — mzRAPP's scheme is empirical and resamples molecules
  only.
- The benchmark and NPP outputs have not yet been matched (no comparison_data); peak/feature
  matching must precede CI estimation.
- Analyzing targeted peak-picking quality directly rather than assessing NPP output
  reliability against a benchmark.
edam_operation: null
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: mzRAPP
  role: computes per-molecule NPP performance metrics via derive_performance_metrics()
    and matching functions that feed the bootstrap matrix
  repo: https://github.com/YasinEl/mzRAPP
- name: boot
  role: R package providing the resampling mechanism used to derive percentile confidence
    intervals at alpha = 0.95
- name: R
  role: implementation language in which mzRAPP and the bootstrapping workflow are
    run
  repo: https://cran.r-project.org/
provenance:
  source_task_ids:
  - task_002
  - task_007
  source_papers:
  - doi: 10.1093/bioinformatics/btab231
    title: 'mzRAPP: a tool for reliability assessment of data pre-processing in non-targeted
      metabolomics'
  source_content_hash: sha256:b0cc24f4f745914504ce97e64a542d383a268c3fb2ae8f3cef33692f6d472033
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/YasinEl/mzRAPP
  tools:
  - R
  - boot
  - mzRAPP
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 18
---

# bootstrap-confidence-interval-estimation

## Summary

Estimates empirical 95% confidence intervals for mzRAPP's non-targeted pre-processing (NPP) reliability metrics by bootstrapping over the benchmark molecules, using the approach implemented in the R boot package.

## When to use

Use when NPP performance metrics (found peaks, split peaks, alignment loss, isotopologue-ratio degeneration) computed against an mzRAPP benchmark need point estimates with bootstrap CI bounds per pre-processing stage.

## When NOT to use

- You need parametric or assumption-based intervals for a single statistic unrelated to benchmark-molecule resampling — mzRAPP's scheme is empirical and resamples molecules only.
- The benchmark and NPP outputs have not yet been matched (no comparison_data); peak/feature matching must precede CI estimation.
- Analyzing targeted peak-picking quality directly rather than assessing NPP output reliability against a benchmark.

## Inputs

- Benchmark.csv (mzRAPP benchmark dataset with ~47 molecules)
- NPP output tables, e.g. XCMS_unaligned_run1.csv and XCMS_aligned_run1.csv
- comparison_data built via match_peaks_to_benchmark / compare_peaks
- molecules-by-metrics matrix of per-molecule stage-wise metrics

## Outputs

- Table (metric, stage, estimate, ci_lower, ci_upper) of 95% percentile bootstrap intervals, e.g. bootstrap_ci_metrics.csv
- Per-metric, per-stage point estimates bracketed by CI bounds (e.g. 83–94% post-alignment detection, 28–53% degenerated IR)

## How to apply

Build a comparison_data object with mzRAPP functions (match_peaks_to_benchmark/compare_peaks) from Benchmark.csv plus the NPP output tables (e.g., XCMS_unaligned_run1.csv and XCMS_aligned_run1.csv), then compute per-molecule metrics with derive_performance_metrics(comparison_data). Assemble a molecules-by-metrics matrix where each of the ~47 benchmark molecules is a row and columns hold stage-wise metrics such as found-peak fraction, isotopologue-ratio degeneration %, split-peak %, and alignment loss %. Using the R boot package, draw B resamples with replacement over molecules, recompute each metric as the mean over the resampled molecules per replicate, and take the 2.5th and 97.5th percentiles as the alpha = 0.95 interval. Report per metric and per pre-processing stage the point estimate with lower and upper CI bounds, writing a (metric, stage, estimate, ci_lower, ci_upper) table, e.g. to bootstrap_ci_metrics.csv. Bootstrapping over molecules rather than peaks is the rationale: molecule-level resampling captures the variability of the benchmark set underlying all derived metrics.

## Related tools

- **mzRAPP** (computes per-molecule NPP performance metrics via derive_performance_metrics() and matching functions that feed the bootstrap matrix) — https://github.com/YasinEl/mzRAPP
- **boot** (R package providing the resampling mechanism used to derive percentile confidence intervals at alpha = 0.95)
- **R** (implementation language in which mzRAPP and the bootstrapping workflow are run) — https://cran.r-project.org/

## Examples

```
library(mzRAPP); library(boot); comparison_data <- match_peaks_to_benchmark(BM, npp_unaligned, npp_aligned); metrics <- derive_performance_metrics(comparison_data); ci <- boot(metrics_matrix, mean_fun, R = 1000); boot.ci(ci, type = "perc", conf = 0.95)
```

## Evaluation signals

- Each metric's 95% bootstrap interval brackets its own point estimate.
- CI ranges overlap the variability ranges reported in the vignette/paper, e.g. 83–94% detected peaks and 28–53% degenerated isotopologue ratios post-alignment, allowing slight variation from bootstrap randomness.
- Output table has exactly the columns metric, stage, estimate, ci_lower, ci_upper for every stage-wise metric computed.
- Script runs reproducibly on the deposited Benchmark.csv and XCMS run-1 inputs.
- Interval width reflects resampling over ~47 benchmark molecules, not over individual peaks.

## Limitations

- Bootstrap randomness means reported CI ranges may vary slightly between runs; exact reproduction of published ranges (83–94% detection, 28–53% degenerated IR) is not expected.
- The paper describes the bootstrapping mechanism as part of the workflow without separately reporting validation results for it.
- Intervals depend on the size and quality of the user-supplied benchmark molecule set (the more molecules the better, per mzRAPP).

## Evidence

- [other] mzRAPP derives empirical confidence intervals (alpha = 0.95) for all of its NPP performance metrics by bootstrapping over the supplied molecules: [evidence span withheld — anchor status: not_found]
- [methods] post-alignment detection reported 83-94%: [evidence span withheld — anchor status: not_found]
- [methods] values may vary slightly due to bootstrap randomness: [evidence span withheld — anchor status: not_found]
- [readme] An overview of mzRAPPs capabilities: [evidence span withheld — anchor status: external_doc]
