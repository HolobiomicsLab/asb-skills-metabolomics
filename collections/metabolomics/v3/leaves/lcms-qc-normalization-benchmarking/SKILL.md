---
name: lcms-qc-normalization-benchmarking
description: Use when comparing the computational cost of QC sample-based normalization
  methods on targeted (e.g., 53-compound) vs untargeted (e.g., 8913-feature) metabolomics
  matrices, or when deciding which normalization method is feasible for high-throughput
  use.
when_to_use_negative:
- The goal is to assess normalization quality (replicate concordance, drift/batch-effect
  reduction) rather than computational cost — use performance evaluation on QC/replicate
  samples instead
- The dataset contains fewer features than can reveal scaling behavior (timing curves
  must scale with dataset size to be meaningful)
- The normalization method of interest is not QC sample-based (e.g., sample-based
  only normalization outside the Metanorm/TIGER/SERFF scope)
edam_operation: null
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: Metanorm R package
  role: Provides metanorm() implementing rLOESS, rGAM, tGAM, QC-RLSC, and QC-RSC normalization
    whose run times are benchmarked
  repo: github.com/UGent-LIMET/Metanorm
- name: R
  role: Runtime environment (>=4.4.0) in which metanorm() is executed and system.time()
    records wall-clock time
- name: pak
  role: Installs the Metanorm package from GitHub (UGent-LIMET/Metanorm)
- name: rLOESS
  role: Fast robust normalization method benchmarked as one of the low-cost options
    (<10 s targeted, <7 min untargeted at six cores)
  repo: github.com/UGent-LIMET/Metanorm
- name: rGAM
  role: Intermediate-speed robust normalization method benchmarked (~18 s targeted,
    ~11 min untargeted)
  repo: github.com/UGent-LIMET/Metanorm
- name: tGAM
  role: Slowest Metanorm method benchmarked (~136 s targeted, ~15 min untargeted)
    with highest computational cost per the article
  repo: github.com/UGent-LIMET/Metanorm
- name: metanorm
  role: Core normalization function called with model='rLOESS'|'rGAM'|'tGAM'|'QC-RLSC'|'QC-RSC'
    for each timed run
  repo: github.com/UGent-LIMET/Metanorm
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1101/2025.09.30.679445
    title: Robust metabolomics data normalization across scales and experimental designs
  source_content_hash: sha256:e797be80e20a4034923fd12ce10ef9b6a8c995f8915b9af4a6230ef5ea5b0b6b
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# lcms-qc-normalization-benchmarking

## Summary

Benchmarks the computational run time of QC-based LC-MS metabolomics normalization methods (Metanorm's rLOESS, rGAM, tGAM, QC-RLSC, QC-RSC, plus external TIGER and SERFF) across targeted and untargeted datasets, quantifying how parallelization and dataset size affect wall-clock time.

## When to use

Use when comparing the computational cost of QC sample-based normalization methods on targeted (e.g., 53-compound) vs untargeted (e.g., 8913-feature) metabolomics matrices, or when deciding which normalization method is feasible for high-throughput use.

## When NOT to use

- The goal is to assess normalization quality (replicate concordance, drift/batch-effect reduction) rather than computational cost — use performance evaluation on QC/replicate samples instead
- The dataset contains fewer features than can reveal scaling behavior (timing curves must scale with dataset size to be meaningful)
- The normalization method of interest is not QC sample-based (e.g., sample-based only normalization outside the Metanorm/TIGER/SERFF scope)

## Inputs

- Numerical rawdata matrix of unnormalized log-intensities (rows = compounds/features, columns = samples), e.g., 53 compounds (targeted, BioHEART) or 8913 features (untargeted, FAME)
- batch vector assigning each sample run to a batch
- metanorm.qc vector labeling each sample run as 'QC' or other sample type
- example.RData from Metanorm package extdata folder

## Outputs

- timing_benchmark.csv with columns: method, dataset (targeted/untargeted), sample count, feature count, ncores, elapsed seconds
- Timing curves of elapsed seconds vs feature count and sample count per method, comparable to Fig. 6

## How to apply

Install R (>=4.4.0) and the Metanorm package from GitHub via pak, then load the example dataset (rawdata matrix, batch vector, metanorm.qc sample-type vector) from the package extdata folder. Run metanorm() separately with model='rLOESS', 'rGAM', 'tGAM', 'QC-RLSC', and 'QC-RSC' on both the targeted and untargeted data matrices, wrapping each call in system.time() under fixed parallelization (e.g., six cores). Run the TIGER R package and the ShinySERRF web interface on the same matrices under the same parallelization settings and record their times as well. Aggregate per-method elapsed seconds into a timing table with columns for method, dataset (targeted/untargeted), sample count, feature count, ncores, and elapsed seconds, then verify the reproduced timings fall within the same order of magnitude as the reference values (e.g., tGAM ~136 s targeted / ~15 min untargeted; rGAM ~18 s / ~11 min; fast methods QC-RSC, rLOESS, QC-RLSC <10 s / <7 min at six cores) and that timing curves scale with feature and sample count.

## Related tools

- **Metanorm R package** (Provides metanorm() implementing rLOESS, rGAM, tGAM, QC-RLSC, and QC-RSC normalization whose run times are benchmarked) — github.com/UGent-LIMET/Metanorm
- **R** (Runtime environment (>=4.4.0) in which metanorm() is executed and system.time() records wall-clock time)
- **pak** (Installs the Metanorm package from GitHub (UGent-LIMET/Metanorm))
- **rLOESS** (Fast robust normalization method benchmarked as one of the low-cost options (<10 s targeted, <7 min untargeted at six cores)) — github.com/UGent-LIMET/Metanorm
- **rGAM** (Intermediate-speed robust normalization method benchmarked (~18 s targeted, ~11 min untargeted)) — github.com/UGent-LIMET/Metanorm
- **tGAM** (Slowest Metanorm method benchmarked (~136 s targeted, ~15 min untargeted) with highest computational cost per the article) — github.com/UGent-LIMET/Metanorm
- **metanorm** (Core normalization function called with model='rLOESS'|'rGAM'|'tGAM'|'QC-RLSC'|'QC-RSC' for each timed run) — github.com/UGent-LIMET/Metanorm

## Examples

```
system.time(normdat <- metanorm(rawdata, model="tGAM", type=metanorm.qc, QCcheck=TRUE, batch=batch))
```

## Evaluation signals

- Fast methods (QC-RSC, rLOESS, QC-RLSC) normalize 53 compounds in under 10 seconds and 8913 features in under 7 minutes at six cores
- tGAM takes ~136 s on BioHEART (53 compounds) and ~15 min on FAME (8913 features); rGAM ~18 s and ~11 min; TIGER ~64 s and ~115 min
- Reproduced timings fall within the same order of magnitude as the reported Fig. 6 values
- timing_benchmark.csv has the required columns (method, dataset, sample count, feature count, ncores, elapsed seconds) and one row per method-dataset combination
- Timing curves scale monotonically with feature count and sample count as reported

## Limitations

- tGAM has a higher computational cost than rLOESS and rGAM, though normalization remains feasible within minutes for thousands of metabolites and hundreds of samples
- Timings depend on parallelization settings (e.g., six cores); results are not comparable across different core counts
- The benchmark relies on external tools (TIGER, ShinySERRF web interface) whose parallelization behavior may differ from Metanorm's

## Evidence

- [results] Fast methods runtime: [evidence span withheld — anchor status: not_found]
- [results] Slow methods runtime: [evidence span withheld — anchor status: not_found]
- [discussion] tGAM cost caveat: "A practical limitation of tGAM is its higher computational cost relative to rLOESS and rGAM; however, normalization remains computationally feasible, completing within minutes on current"
- [intro] Parallel efficiency rationale: "Our implementations further capitalize on the problem’s pleasingly parallel nature, resulting in efficient high-throughput normalization"
- [readme] Package installation: "Make sure `R` (**version >= 4.4.0**) is installed on your computer"
- [readme] Dataset loading: "load(system.file("extdata", "example.RData", package = "metanorm"))"
