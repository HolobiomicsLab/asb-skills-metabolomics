---
name: parallel-high-throughput-normalization
description: Reference for normalizing large-scale LC-MS metabolomics datasets (e.g.
  ~8913 untargeted features or ~53 targeted compounds across hundreds of samples)
  where single-threaded QC-based normalization would be too slow.
when_to_use_negative:
- The dataset is tiny (e.g. only a handful of features) and single-threaded runtime
  is already negligible — the overhead of parallel setup offers no benefit.
- QCs are not representative of biological samples (metanorm's QCcheck flag returns
  a discrepancy); robust QC-based normalization may mis-correct biological samples
  and a different strategy is warranted.
- The data have already been normalized or are a derived feature table rather than
  raw intensities; re-running normalization would double-correct the signal.
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: Metanorm R package
  role: Provides the parallel implementations of the robust normalization models (tGAM,
    rGAM, rLOESS, QC-RLSC, QC-RSC) and diagnostic plotting used for high-throughput
    normalization.
  repo: https://github.com/UGent-LIMET/Metanorm
- name: R
  role: Runtime environment required (version >= 4.4.0) to install and execute the
    Metanorm package.
  repo: https://cloud.r-project.org/index.html
- name: pak
  role: Package installer used to fetch Metanorm directly from GitHub.
- name: metanorm
  role: The metanorm() function call that performs the parallel normalization on the
    raw data matrix.
  repo: https://github.com/UGent-LIMET/Metanorm
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1101/2025.09.30.679445
    title: Robust metabolomics data normalization across scales and experimental designs
  source_content_hash: sha256:ac9570d79b26c13431e40c72d1dff7d5bde78ee96af1cfc3d3633c380423d478
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/UGent-LIMET/Metanorm
  tools:
  - Metanorm R package
  - R
  - metanorm
  - pak
  - rGAM
  - rLOESS
  - tGAM
  edam_operation: http://edamontology.org/operation_3646
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6428571428571429
n_claims: 14
---

# parallel-high-throughput-normalization

## Summary

Perform metabolomics normalization of large LC-MS data matrices (thousands of features, hundreds of samples) using the Metanorm R package's efficient parallel implementation of robust methods (tGAM, rGAM, rLOESS, QC-RLSC, QC-RSC), keeping runtime feasible for high-throughput datasets.

## When to use

Reference for normalizing large-scale LC-MS metabolomics datasets (e.g. ~8913 untargeted features or ~53 targeted compounds across hundreds of samples) where single-threaded QC-based normalization would be too slow.

## When NOT to use

- The dataset is tiny (e.g. only a handful of features) and single-threaded runtime is already negligible — the overhead of parallel setup offers no benefit.
- QCs are not representative of biological samples (metanorm's QCcheck flag returns a discrepancy); robust QC-based normalization may mis-correct biological samples and a different strategy is warranted.
- The data have already been normalized or are a derived feature table rather than raw intensities; re-running normalization would double-correct the signal.

## Inputs

- Numerical rawdata matrix (rows = compounds/features, columns = samples) of unnormalized LC-MS intensities
- Vector metanorm.qc indicating for each sample run whether it is a QC or another sample type
- Vector batch containing for each sample run the acquisition batch
- Model choice string: 'tGAM' (default), 'rGAM', 'rLOESS', 'QC-RLSC', or 'QC-RSC'

## Outputs

- normdat — normalized numerical matrix of the same dimensions as rawdata
- Pre- vs. post-normalization intensity vs. order plots per compound for diagnostics
- PCA score plots (via plotPCA) labeled by batch to verify batch-effect removal

## How to apply

Install R (>=4.4.0) and the Metanorm package from GitHub (UGent-LIMET/Metanorm) via pak, then load the example data matrix example.RData from the package extdata folder. Call metanorm() on the raw intensity matrix (rows = compounds, columns = samples), supplying the sample-type vector (QC vs. biological), the batch vector, and model = "tGAM" (default, most robust) or "rGAM"/"rLOESS"/"QC-RLSC"/"QC-RSC" for speed; rGAM and rLOESS complete a 8913-feature matrix within minutes on six cores, whereas tGAM may take ~15 min and TIGER up to ~115 min. The package's pleasingly parallel nature lets each feature be normalized independently, so wall-clock time scales with feature count; QCcheck = TRUE and batch = batch should be set to verify QC representativeness and reduce batch effects. Inspect the generated pre- vs. post-normalization and PCA score plots to confirm drift and batch effects were removed without introducing new artifacts.

## Related tools

- **Metanorm R package** (Provides the parallel implementations of the robust normalization models (tGAM, rGAM, rLOESS, QC-RLSC, QC-RSC) and diagnostic plotting used for high-throughput normalization.) — https://github.com/UGent-LIMET/Metanorm
- **R** (Runtime environment required (version >= 4.4.0) to install and execute the Metanorm package.) — https://cloud.r-project.org/index.html
- **pak** (Package installer used to fetch Metanorm directly from GitHub.)
- **metanorm** (The metanorm() function call that performs the parallel normalization on the raw data matrix.) — https://github.com/UGent-LIMET/Metanorm

## Examples

```
library(metanorm); normdat <- metanorm(rawdata[1:5,], model="tGAM", type=metanorm.qc, QCcheck=TRUE, batch=batch, plotdir="~/metanormExample/")
```

## Evaluation signals

- Wall-clock timing should show fast methods (QC-RSC, rLOESS, QC-RLSC) finishing 8913 features in under ~7 min on six cores, matching the benchmark of the paper.
- PCA score plots after normalization should show diminished batch separation compared with pre-normalization plots labeled by batch.
- QCcheck = TRUE should raise no warning that QCs are unrepresentative of biological samples.
- Individual compound pre- vs. post-normalization plots should show reduced signal drift while preserving biological signal.
- Normalized output matrix normdat must have identical row/column dimensions as the input rawdata matrix.

## Limitations

- tGAM has a higher computational cost than rLOESS and rGAM; for very large datasets wall-clock time may exceed minutes on limited hardware.
- Robust normalization assumes QCs are representative; discrepancies between QC and biological samples invalidate normalization and filtering based on QC RSD.
- Automated outlier detection is non-trivial; the method does not remove biological outliers, only downweights/ accommodates them via robust fitting.

## Evidence

- [intro] Pleasingly parallel implementation: "Our implementations further capitalize on the problem’s pleasingly parallel nature, resulting in efficient high-throughput normalization."
- [results] Speed benchmark on large dataset: [evidence span withheld — anchor status: not_found]
- [readme] R version requirement: "Make sure `R` (**version >= 4.4.0**) is installed on your computer"
- [methods] Package installation via pak: "if (!require("pak", quietly = TRUE))
    install.packages("pak")"
- [abstract] tGAM default choice rationale: "Overall, tGAM-based normalization suggested the best performance across scenarios and is proposed as default choice."
- [discussion] Higher cost of tGAM: "A practical limitation of tGAM is its higher computational cost relative to rLOESS and rGAM"
- [readme] Loading example data: "load(system.file("extdata", "example.RData", package = "metanorm"))"
- [readme] Diagnostic plot inspection: "It is highly recommended to look at a decent number of these plots to judge normalization performance."
