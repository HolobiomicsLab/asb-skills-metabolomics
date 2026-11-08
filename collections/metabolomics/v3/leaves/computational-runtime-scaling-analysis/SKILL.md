---
name: computational-runtime-scaling-analysis
description: Use when comparing computational cost of normalization methods (rLOESS,
  rGAM, tGAM, QC-RLSC, QC-RSC) across targeted (53-compound) and untargeted (8913-feature)
  metabolomics datasets, or when deciding whether parallelization over cores is needed
  to keep normalization tractable.
when_to_use_negative:
- The goal is benchmarking TIGER or SERFF alone without also running the Metanorm
  methods under identical parallelization settings — the comparison becomes uninterpretable.
- Data are already normalized, or the data matrix is not a raw unnormalized compound-by-sample
  matrix such as Metanorm's rawdata.
- Only a very small dataset is available where run-time differences between methods
  are negligible and parallelization is irrelevant.
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: Metanorm R package
  role: Provides the parallelized implementations of rLOESS, rGAM, tGAM, QC-RLSC and
    QC-RSC used for timing benchmarks.
  repo: github.com/UGent-LIMET/Metanorm
- name: pak
  role: Used to install the Metanorm R package directly from GitHub for reproducible
    benchmark setup.
  repo: github.com/UGent-LIMET/Metanorm
- name: R
  role: Runtime environment (version >=4.4.0) in which the normalization methods and
    system.time() timing are executed.
- name: rLOESS
  role: Fast robust normalization method included in the runtime comparison as a low-cost
    baseline against tGAM.
  repo: github.com/UGent-LIMET/Metanorm
- name: rGAM
  role: Intermediate-cost robust GAM normalization method benchmarked alongside tGAM
    to contrast scaling behavior.
  repo: github.com/UGent-LIMET/Metanorm
- name: tGAM
  role: Highest-cost method in the benchmark; timing its run on large untargeted datasets
    motivates parallelization and default-choice trade-offs.
  repo: github.com/UGent-LIMET/Metanorm
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1101/2025.09.30.679445
    title: Robust metabolomics data normalization across scales and experimental designs
  source_content_hash: sha256:a692cecb5c312a181bde10af3306898663a45167046d101ca2408940c14949bd
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
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 15
---

# computational-runtime-scaling-analysis

## Summary

Benchmark wall-clock run times of metabolomics normalization methods (rLOESS, rGAM, tGAM, QC-RLSC, QC-RSC) across dataset sizes and core counts, using Metanorm's parallelized implementations to quantify scaling behavior.

## When to use

Use when comparing computational cost of normalization methods (rLOESS, rGAM, tGAM, QC-RLSC, QC-RSC) across targeted (53-compound) and untargeted (8913-feature) metabolomics datasets, or when deciding whether parallelization over cores is needed to keep normalization tractable.

## When NOT to use

- The goal is benchmarking TIGER or SERFF alone without also running the Metanorm methods under identical parallelization settings — the comparison becomes uninterpretable.
- Data are already normalized, or the data matrix is not a raw unnormalized compound-by-sample matrix such as Metanorm's rawdata.
- Only a very small dataset is available where run-time differences between methods are negligible and parallelization is irrelevant.

## Inputs

- rawdata numerical matrix (rows = compounds, columns = samples), e.g. Metanorm's example.RData
- metanorm.qc vector indicating which sample runs are QC
- batch vector of batch labels per sample run
- Number of parallel workers (cores) available for the run

## Outputs

- Per-method elapsed run-time values (seconds) for each normalization model
- Aggregated timing table (timing_benchmark.csv) with method, dataset type, sample count, feature count, ncores and elapsed seconds
- Timing-vs-feature-count scaling comparison across methods

## How to apply

Install R (>=4.4.0) and the Metanorm R package from GitHub via pak, then run the metanorm() function on both a targeted (53-compound) and an untargeted (8913-feature) data matrix for each model — "rLOESS", "rGAM", "tGAM", "QC-RLSC" and "QC-RSC" — recording elapsed seconds with system.time() under a fixed core count (e.g. six cores). Benchmark TIGER and the ShinySERRF/SERFF web interface on the same matrices under the same parallelization settings to obtain a like-for-like comparison. Aggregate per-method run times into a table with columns for method, dataset type (targeted/untargeted), sample count, feature count, ncores and elapsed seconds, then inspect how run time scales with feature count and sample count and how the higher-cost tGAM compares with rLOESS and rGAM. Validate that reproduced timings fall within the same order of magnitude as the reported values (e.g. tGAM ~136 s / ~15 min, rGAM ~18 s / ~11 min, QC-RSC <10 s / <7 min at six cores) and that the timing curves increase with dataset size as reported.

## Related tools

- **Metanorm R package** (Provides the parallelized implementations of rLOESS, rGAM, tGAM, QC-RLSC and QC-RSC used for timing benchmarks.) — github.com/UGent-LIMET/Metanorm
- **pak** (Used to install the Metanorm R package directly from GitHub for reproducible benchmark setup.) — github.com/UGent-LIMET/Metanorm
- **R** (Runtime environment (version >=4.4.0) in which the normalization methods and system.time() timing are executed.)
- **rLOESS** (Fast robust normalization method included in the runtime comparison as a low-cost baseline against tGAM.) — github.com/UGent-LIMET/Metanorm
- **rGAM** (Intermediate-cost robust GAM normalization method benchmarked alongside tGAM to contrast scaling behavior.) — github.com/UGent-LIMET/Metanorm
- **tGAM** (Highest-cost method in the benchmark; timing its run on large untargeted datasets motivates parallelization and default-choice trade-offs.) — github.com/UGent-LIMET/Metanorm

## Examples

```
pak::pak("UGent-LIMET/Metanorm"); load(system.file("extdata","example.RData",package="metanorm")); system.time(normdat <- metanorm(rawdata, model="tGAM", type=metanorm.qc, QCcheck=TRUE, batch=batch))
```

## Evaluation signals

- Reproduced run times fall within the same order of magnitude as the reported values (tGAM ~136 s on BioHEART / ~15 min on FAME; rGAM ~18 s / ~11 min; QC-RSC <10 s / <7 min at six cores).
- Fast methods (QC-RSC, rLOESS, QC-RLSC) normalize 53 compounds in under 10 seconds and 8913 features in under 7 minutes at six cores.
- Timing curves increase monotonically with feature count and sample count across all benchmarked methods.
- timing_benchmark.csv contains the required columns: method, dataset (targeted/untargeted), sample count, feature count, ncores, elapsed seconds.
- Every method is benchmarked on both the targeted and untargeted datasets under the same ncores setting before comparison.

## Limitations

- tGAM has a higher computational cost relative to rLOESS and rGAM, so timing benchmarks on very large datasets can be slow.
- Run-time comparisons must hold the core count fixed (e.g. six cores) to be fair, since Metanorm's implementations capitalize on the problem's pleasingly parallel nature.
- Simulated timing benchmarks must maintain neutrality and not favor any benchmarked method, e.g. by simulating from a fitted model of one of them.

## Evidence

- [intro] Parallelization and high-throughput normalization of Metanorm implementations.: "Our implementations further capitalize on the problem’s pleasingly parallel nature, resulting in efficient high-throughput normalization."
- [discussion] tGAM computational cost limitation.: "A practical limitation of tGAM is its higher computational cost relative to rLOESS and rGAM; however, normalization remains computationally feasible, completing within minutes on current"
- [methods] Method installation from R.: "pak::pak("UGent-LIMET/Metanorm")"
- [readme] README: recommended use and performance context for tGAM and rGAM/rLOESS.: "tGAM is recommended due to its superior robustness, but rGAM and rLOESS are faster."
- [readme] README: concrete R installation prerequisite and version.: "Make sure `R` (**version >= 4.4.0**) is installed on your computer"
- [readme] README: concrete metanorm invocation with QC/batch parameters.: [evidence span withheld — anchor status: not_found]
