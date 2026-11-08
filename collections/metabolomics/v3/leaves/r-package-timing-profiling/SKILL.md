---
name: r-package-timing-profiling
description: Use when benchmarking the run time of R-based metabolomics normalization
  methods across feature-count and sample-count scales, and you need wall-clock timings
  per method under fixed core allocations.
when_to_use_negative:
- Your goal is to assess normalization quality (drift, batch-effect removal, replicate
  concordance) rather than run-time cost — use Metanorm's plotPCA and pre-/post-normalization
  intensity-vs-order plots instead.
- Your input is a single small matrix (< tens of features) where timing differences
  between methods are negligible and profiling would not be informative.
- You need statistical or biological validation of the normalized output, not a computational
  benchmark.
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: R
  role: Runtime environment (version >= 4.4.0) in which the normalization calls are
    timed via system.time()
  repo: https://cloud.r-project.org/index.html
- name: pak
  role: Package installer used to fetch Metanorm from GitHub before profiling
  repo: github.com/UGent-LIMET/Metanorm
- name: Metanorm R package
  role: Provides metanorm() with the five model variants (rLOESS, rGAM, tGAM, QC-RLSC,
    QC-RSC) being timed and the example.RData test data
  repo: github.com/UGent-LIMET/Metanorm
- name: rLOESS
  role: One of the fast, cross-validated robust LOESS normalization variants benchmarked
    for run time
  repo: github.com/UGent-LIMET/Metanorm
- name: rGAM
  role: Robust additive-model normalization variant benchmarked for run time; slower
    than rLOESS but faster than tGAM
  repo: github.com/UGent-LIMET/Metanorm
- name: tGAM
  role: Recommended default tGAM method; benchmarked for run time despite its higher
    computational cost
  repo: github.com/UGent-LIMET/Metanorm
- name: metanorm
  role: The function invoked with different model= arguments during each timing run
  repo: github.com/UGent-LIMET/Metanorm
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1101/2025.09.30.679445
    title: Robust metabolomics data normalization across scales and experimental designs
  source_content_hash: sha256:b75f196092cd1d734295e5d3b255bd9e8936d5066194acbce5ae224ba6dd8243
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Metanorm R package
  - R
  - metanorm
  - pak
  - rGAM
  - rLOESS
  - tGAM
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7333333333333333
n_claims: 15
---

# r-package-timing-profiling

## Summary

Wall-clock timing profiling of the Metanorm R package's normalization methods (rLOESS, rGAM, tGAM, QC-RLSC, QC-RSC) across targeted (53 compounds) and untargeted (8913 features) LC-MS metabolomics datasets, to characterize computational cost and parallelization behaviour.

## When to use

Use when benchmarking the run time of R-based metabolomics normalization methods across feature-count and sample-count scales, and you need wall-clock timings per method under fixed core allocations.

## When NOT to use

- Your goal is to assess normalization quality (drift, batch-effect removal, replicate concordance) rather than run-time cost — use Metanorm's plotPCA and pre-/post-normalization intensity-vs-order plots instead.
- Your input is a single small matrix (< tens of features) where timing differences between methods are negligible and profiling would not be informative.
- You need statistical or biological validation of the normalized output, not a computational benchmark.

## Inputs

- Raw intensity matrix (numerical, rows = compounds, columns = samples), e.g. the rawdata object from Metanorm's example.RData
- Sample-type vector metanorm.qc (QC vs. non-QC labels per column)
- Batch vector (batch labels per sample run)
- Method specification strings: model = "rLOESS" | "rGAM" | "tGAM" | "QC-RLSC" | "QC-RSC"
- External normalization tools for cross-comparison: TIGER R package and ShinySERRF web interface
- Core allocation (e.g. ncores = 6) for parallel timing

## Outputs

- Per-method elapsed-time table (method, dataset targeted/untargeted, sample count, feature count, ncores, elapsed seconds) suitable for export as timing_benchmark.csv
- Timing curves over feature count and sample count reproducing Fig. 6 trend lines

## How to apply

Install R (>=4.4.0) and pull the Metanorm package from GitHub via pak::pak("UGent-LIMET/Metanorm"). Load the bundled example.RData to obtain the rawdata matrix (rows = compounds, columns = samples), the batch vector, and the metanorm.qc type vector. Run metanorm() with model="rLOESS", "rGAM", "tGAM", "QC-RLSC", and "QC-RSC" on both a targeted (~53 compounds) and an untargeted (~8913 features) matrix, wrapping each call in system.time() to record elapsed seconds at a fixed core allocation (e.g. six cores). Repeat the same datasets through TIGER and the ShinySERRF/SERFF interface under identical parallelization settings so cross-method timings are comparable. Rationale: the article reports that at six cores the fast methods (QC-RSC, rLOESS, QC-RLSC) normalized 53 compounds in under 10 s and 8913 features in under 7 min, while slower methods took ~18 s (rGAM) to ~136 s (tGAM) on the targeted data and ~11 min (rGAM) to ~115 min (TIGER) on the untargeted data, so timing curves over feature count and sample count are the primary output and order-of-magnitude agreement with Fig. 6 is the acceptance criterion.

## Related tools

- **R** (Runtime environment (version >= 4.4.0) in which the normalization calls are timed via system.time()) — https://cloud.r-project.org/index.html
- **pak** (Package installer used to fetch Metanorm from GitHub before profiling) — github.com/UGent-LIMET/Metanorm
- **Metanorm R package** (Provides metanorm() with the five model variants (rLOESS, rGAM, tGAM, QC-RLSC, QC-RSC) being timed and the example.RData test data) — github.com/UGent-LIMET/Metanorm
- **rLOESS** (One of the fast, cross-validated robust LOESS normalization variants benchmarked for run time) — github.com/UGent-LIMET/Metanorm
- **rGAM** (Robust additive-model normalization variant benchmarked for run time; slower than rLOESS but faster than tGAM) — github.com/UGent-LIMET/Metanorm
- **tGAM** (Recommended default tGAM method; benchmarked for run time despite its higher computational cost) — github.com/UGent-LIMET/Metanorm
- **metanorm** (The function invoked with different model= arguments during each timing run) — github.com/UGent-LIMET/Metanorm

## Examples

```
Rscript -e 'library(metanorm); load(system.file("extdata","example.RData",package="metanorm")); print(system.time(normdat <- metanorm(rawdata, model="tGAM", type=metanorm.qc, batch=batch)))'
```

## Evaluation signals

- Elapsed times fall within the same order of magnitude as the article's reported values (tGAM ~136 s / ~15 min, rGAM ~18 s / ~11 min, QC-RSC <10 s / <7 min at six cores).
- The timing curve scales monotonically with dataset size: targeted (53 compounds) runs are consistently faster than untargeted (8913 features) runs for every method.
- All five Metanorm model variants and the external TIGER/SERFF timings are recorded at an identical ncores setting, so the comparison is apples-to-apples.
- The exported table (e.g. timing_benchmark.csv) contains non-empty columns for method, dataset, sample count, feature count, ncores, and elapsed seconds.
- system.time() output is captured for every method call, so no timing entry is missing or fabricated.

## Limitations

- tGAM has a substantially higher computational cost than rLOESS and rGAM; it remains feasible (minutes) but is not suitable when wall-clock budget is severely constrained.
- Timings are only meaningful at a stated, fixed core allocation; parallelization behaviour must be reported alongside the elapsed seconds.
- Benchmarking care is needed for neutrality — simulating from a fitted model of one of the benchmarked methods could favour that method.
- The article notes that machine-learning-based normalization methods' reliability on small datasets has not been broadly investigated, which constrains the generality of timing extrapolations to small n.

## Evidence

- [results] Fast methods under 10 s for targeted data: [evidence span withheld — anchor status: not_found]
- [results] Slow-method timings on BioHEART and FAME: [evidence span withheld — anchor status: not_found]
- [readme] R version requirement: "Make sure `R` (**version >= 4.4.0**) is installed on your computer"
- [readme] Loading example data: "load(system.file("extdata", "example.RData", package = "metanorm"))"
- [discussion] tGAM computational cost caveat: "A practical limitation of tGAM is its higher computational cost relative to rLOESS and rGAM; however, normalization remains computationally feasible, completing within minutes on current"
- [methods] Installing Metanorm: "pak::pak("UGent-LIMET/Metanorm")"
