---
name: metabolomics-workflow-benchmark-reproduction
description: Use when you need to benchmark wall-clock run time of metabolomics normalization
  methods (e.g. tGAM vs TIGER/SERFF) across targeted and untargeted datasets under
  a fixed parallelization setting.
when_to_use_negative:
- When the goal is to assess normalization quality (replicate concordance, drift or
  batch-effect reduction) rather than computational cost — the article's performance
  evaluation uses pre-/post-normalization and PCA score plots, not timing benchmarks.
- When the dataset is a single metabolite or a very small feature set — Metanorm's
  example usage normalizes the first 5 compounds; the benchmark's timing curve is
  designed to compare behaviour across 53 vs 8913 features and would not be informative
  at that scale.
- When the downstream analysis requires statistical power or differential abundance
  testing — the article's benchmarking is scoped to normalization runtime, not to
  downstream inference.
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: Metanorm R package
  role: Implements the robust normalization methods (rLOESS, rGAM, tGAM, QC-RLSC,
    QC-RSC) whose run times are benchmarked; called via metanorm() with model= argument.
  repo: https://github.com/UGent-LIMET/Metanorm
- name: TIGER R package
  role: External normalization method whose run time is compared against Metanorm's
    methods on the same datasets.
- name: ShinySERRF web interface
  role: External (SERFF) normalization method whose run time is compared on the same
    datasets under matched parallelization.
- name: pak
  role: Used to install the Metanorm package from GitHub as a prerequisite to running
    the benchmark.
- name: R
  role: Runtime environment (version >= 4.4.0) required to execute the benchmark scripts.
  repo: https://cloud.r-project.org/index.html
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1101/2025.09.30.679445
    title: Robust metabolomics data normalization across scales and experimental designs
  source_content_hash: sha256:07daa69a56e7b63085599fe74ab8b0f6355bf12b24163b74ec8e21652b220236
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/UGent-LIMET/Metanorm
  tools:
  - Metanorm R package
  - R
  - ShinySERRF web interface
  - TIGER R package
  - metanorm
  - pak
  - rGAM
  - rLOESS
  - tGAM
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
merged_aliases:
- metabolomics-workflow-reproduction
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 14
---

# metabolomics-workflow-benchmark-reproduction

## Summary

Reproduce the computational-time benchmark of QC-based metabolomics normalization methods (rLOESS, rGAM, tGAM, QC-RLSC, QC-RSC vs TIGER and SERFF) on a targeted 53-compound dataset and an untargeted 8913-feature dataset, quantifying how run time scales with feature count and core count.

## When to use

Use when you need to benchmark wall-clock run time of metabolomics normalization methods (e.g. tGAM vs TIGER/SERFF) across targeted and untargeted datasets under a fixed parallelization setting.

## When NOT to use

- When the goal is to assess normalization quality (replicate concordance, drift or batch-effect reduction) rather than computational cost — the article's performance evaluation uses pre-/post-normalization and PCA score plots, not timing benchmarks.
- When the dataset is a single metabolite or a very small feature set — Metanorm's example usage normalizes the first 5 compounds; the benchmark's timing curve is designed to compare behaviour across 53 vs 8913 features and would not be informative at that scale.
- When the downstream analysis requires statistical power or differential abundance testing — the article's benchmarking is scoped to normalization runtime, not to downstream inference.

## Inputs

- Targeted LC-MS metabolomics data matrix (53 compounds, e.g. BioHEART) — numerical matrix rows=compounds, columns=samples
- Untargeted LC-MS metabolomics data matrix (8913 features, e.g. FAME) — numerical matrix rows=features, columns=samples
- Batch vector mapping each sample run to its acquisition batch
- Sample-type vector distinguishing QC from non-QC sample runs
- Parallelization setting (number of worker cores, e.g. six cores)

## Outputs

- Per-method wall-clock elapsed times (seconds) for rLOESS, rGAM, tGAM, QC-RLSC, QC-RSC, TIGER and SERFF on both datasets
- timing_benchmark.csv with columns for method, dataset (targeted/untargeted), sample count, feature count, ncores and elapsed seconds
- Timing curve over feature count and sample count reproducing the trend lines of Fig. 6

## How to apply

Install R (>=4.4.0) and the Metanorm R package from GitHub (UGent-LIMET/Metanorm) via pak, then load the example dataset example.RData from the package extdata folder. Run the metanorm() function with model="rLOESS", model="rGAM", model="tGAM", model="QC-RLSC" and model="QC-RSC" on the targeted (53-compound) and untargeted (8913-feature) data matrices, recording wall-clock time for each method with system.time(). Run the TIGER R package and the ShinySERRF web interface on the same datasets under the same parallelization settings (e.g. six cores where applicable), recording their run times as well. Aggregate per-method run times into a timing curve over feature count and sample count, reproducing the trend lines and reported values of Fig. 6. Write the aggregated timing table to timing_benchmark.csv with columns for method, dataset (targeted/untargeted), sample count, feature count, ncores and elapsed seconds, and validate that the reproduced timings fall within the same order of magnitude as the reported values.

## Related tools

- **Metanorm R package** (Implements the robust normalization methods (rLOESS, rGAM, tGAM, QC-RLSC, QC-RSC) whose run times are benchmarked; called via metanorm() with model= argument.) — https://github.com/UGent-LIMET/Metanorm
- **TIGER R package** (External normalization method whose run time is compared against Metanorm's methods on the same datasets.)
- **ShinySERRF web interface** (External (SERFF) normalization method whose run time is compared on the same datasets under matched parallelization.)
- **pak** (Used to install the Metanorm package from GitHub as a prerequisite to running the benchmark.)
- **R** (Runtime environment (version >= 4.4.0) required to execute the benchmark scripts.) — https://cloud.r-project.org/index.html

## Examples

```
load(system.file("extdata", "example.RData", package = "metanorm")); normdat <- metanorm(rawdata, model = "tGAM", type = metanorm.qc, QCcheck = TRUE, batch = batch)
```

## Evaluation signals

- At six cores the fast methods (QC-RSC, rLOESS, QC-RLSC) normalize the 53-compound targeted dataset in under 10 seconds and the 8913-feature untargeted dataset in under 7 minutes.
- The slower methods reproduce the reported Fig. 6 values: rGAM ~18 s / ~11 min, tGAM ~136 s / ~15 min, TIGER ~64 s / ~115 min on BioHEART and FAME respectively.
- timing_benchmark.csv is populated with rows for every method × dataset combination and includes ncores as a column so the parallelization effect can be traced.
- Elapsed times scale with dataset size (feature count and sample count) in the same direction as the article's Fig. 6 timing curves.
- Reproduced timings fall within the same order of magnitude as the article's reported values (order-of-magnitude agreement, not exact equality).

## Limitations

- tGAM carries a higher computational cost than rLOESS and rGAM; on current hardware normalization completes within minutes for thousands of metabolites and hundreds of samples, but the cost grows steeply with feature count.
- Benchmarking must be neutral — care should be taken not to favour any of the benchmarked methods (e.g. by simulating from a fitted model of one of them).
- The Metanorm README's example usage only normalizes the first 5 compounds of the example dataset; scaling the benchmark to 53 or 8913 features requires expanding that call.
- TIGER and ShinySERRF are external tools; no exact version pins are given in the article or README for these comparators.

## Evidence

- [discussion] Metanorm's tGAM computational cost exceeds rLOESS/rGAM but stays feasible at scale: "A practical limitation of tGAM is its higher computational cost relative to rLOESS and rGAM; however, normalization remains computationally feasible, completing within minutes on current"
- [readme] Metanorm implements five normalization methods callable through one function: "The R package implements three (new) robust normalization methods (tGAM, rGAM and rLOESS), alongside formerly proposed ones (QC-RLSC, QC-RSC)."
- [readme] Metanorm example dataset lives in the package extdata folder and contains rawdata, batch and QC vectors: "An example dataset can be loaded from the package:
```r
load(system.file("extdata", "example.RData", package = "metanorm"))
```"
- [readme] The metanorm() function normalizes the data matrix and accepts a model argument: "normdat <- metanorm(rawdata[1:5,],       # numerical data matrix to normalize
	                model = "tGAM",      # default tGAM method"
- [results] Timing measurement and parallelization are grounded in R's system.time and core count: [evidence span withheld — anchor status: not_found]
- [results] TIGER and SERFF are the external comparators whose run time is benchmarked: [evidence span withheld — anchor status: not_found]
- [intro] Benchmarking pitfalls include using reduction of QC RSD as a proxy for good normalization: "Given common pitfalls in benchmarking normalization strategies, such as using a reduction of the relative standard deviation of QC samples"
