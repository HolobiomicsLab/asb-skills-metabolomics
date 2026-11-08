---
name: lc-ms-metabolomics-batch-correction
description: Use when an unnormalized LC-MS metabolomics data matrix (rows = compounds,
  columns = samples) shows signal drift over acquisition order and batch effects that
  QC-only or non-robust normalization fails to fix, especially with outlier-contaminated
  features.
when_to_use_negative:
- When QCs are not representative of biological samples and the discrepancy is genuine
  biology rather than artifacts — metanorm's QCcheck flag will flag this, and forcing
  QC-RLSC on QCs only can distort biological signals (the paper recommends including
  both QC and replicate samples).
- When filtering decisions such as RSD/CV cutoffs on QC samples are the goal — the
  article warns that differences between QC and biological samples may invalidate
  RSD-based filtering, so this skill is for normalization, not QC-RSD feature filtering.
- When downstream multiple-testing correction of differential abundance results is
  needed — the article explicitly places multiple testing correction outside the scope
  of this normalization workflow.
edam_operation: http://edamontology.org/operation_3435
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: metanorm
  role: Core function of the Metanorm R package that performs the batch- and drift-correcting
    normalization (tGAM/rGAM/rLOESS/QC-RLSC models) and writes diagnostic plots
  repo: github.com/UGent-LIMET/Metanorm
- name: Metanorm R package
  role: R package integrating the robust normalization methods with visualization
    utilities (metanorm, plotPCA) used for batch correction and performance verification
  repo: github.com/UGent-LIMET/Metanorm
- name: tGAM
  role: Default and best-performing robust GAM-based normalization model used to correct
    drift and batch effects
  repo: github.com/UGent-LIMET/Metanorm
- name: rGAM
  role: Faster robust GAM alternative used when computational cost of tGAM is a limiting
    factor
  repo: github.com/UGent-LIMET/Metanorm
- name: rLOESS
  role: Faster robust LOESS alternative used when computational cost of tGAM is a
    limiting factor
  repo: github.com/UGent-LIMET/Metanorm
- name: QC-RLSC
  role: Classical QC-only normalization model (run with QConly = TRUE) used as a comparison
    baseline; sensitive to outliers in biological samples
  repo: github.com/UGent-LIMET/Metanorm
- name: plotPCA
  role: Generates pre- vs. post-normalization PC score plots labeled by batch to verify
    that batch effects diminished
  repo: github.com/UGent-LIMET/Metanorm
- name: R
  role: Runtime environment required (version >= 4.4.0) for installing and running
    the Metanorm package
- name: pak
  role: R package installer used to install Metanorm directly from GitHub (pak::pak('UGent-LIMET/Metanorm'))
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1101/2025.09.30.679445
    title: Robust metabolomics data normalization across scales and experimental designs
  source_content_hash: sha256:3d0fd28cded25f8aa7ab7725473827aa44140ac076fdca7d13226ba52da263f0
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Metanorm R package
  - QC-RLSC
  - R
  - metanorm
  - pak
  - plotPCA
  - rGAM
  - rLOESS
  - tGAM
  edam_operation: http://edamontology.org/operation_3435
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5384615384615384
n_claims: 13
---

# lc-ms-metabolomics-batch-correction

## Summary

Corrects acquisition-order signal drift and between-batch effects in LC-MS metabolomics intensity matrices using robust, QC- and replicate-aware normalization (tGAM as default, with rGAM, rLOESS and QC-RLSC alternatives) from the Metanorm R package, chosen for outlier resistance and verified via per-compound plots and PCA.

## When to use

Use when an unnormalized LC-MS metabolomics data matrix (rows = compounds, columns = samples) shows signal drift over acquisition order and batch effects that QC-only or non-robust normalization fails to fix, especially with outlier-contaminated features.

## When NOT to use

- When QCs are not representative of biological samples and the discrepancy is genuine biology rather than artifacts — metanorm's QCcheck flag will flag this, and forcing QC-RLSC on QCs only can distort biological signals (the paper recommends including both QC and replicate samples).
- When filtering decisions such as RSD/CV cutoffs on QC samples are the goal — the article warns that differences between QC and biological samples may invalidate RSD-based filtering, so this skill is for normalization, not QC-RSD feature filtering.
- When downstream multiple-testing correction of differential abundance results is needed — the article explicitly places multiple testing correction outside the scope of this normalization workflow.

## Inputs

- Numerical intensity matrix rawdata with rows = compounds and columns = samples (e.g. the example.RData object)
- Vector batch giving the batch membership of each sample run (e.g. 'batch1', ...)
- Vector metanorm.qc giving the sample type of each run ('QC' vs. other, e.g. 'sample')

## Outputs

- Normalized intensity matrix normdat returned by metanorm()
- Pre- and post-normalization PCA score plots (plotPCA) labeled by batch
- Per-compound pre- vs. post-normalization intensity-vs-order diagnostic plots written to plotdir

## How to apply

Load the numerical rawdata matrix (rows = compounds, columns = samples), the per-sample batch vector, and the sample-type vector marking QC vs. other samples (metanorm.qc format, e.g. 'QC' and 'sample'), such as the package's example.RData. Call metanorm() with model = 'tGAM' (the default, recommended for superior robustness), type = metanorm.qc, QCcheck = TRUE so the function checks whether QCs are representative of biological samples, and batch = batch for batch-wise correction, writing diagnostic plots to a plotdir. For faster alternatives with large datasets, swap in model = 'rGAM' or 'rLOESS'; to compare against the classical approach, run model = 'QC-RLSC' with QConly = TRUE, which normalizes using QC samples only and is vulnerable to outliers in biological samples. Assess success by pre- vs. post-normalization plotPCA() score plots labeled by batch (residual batch structure should shrink) and by per-compound intensity-vs-order plots in plotdir, which reveal residual drift and outlier contamination — the authors strongly recommend inspecting many of these plots rather than relying on RSD reduction of QC samples or increased numbers of differentially abundant metabolites as proxies.

## Related tools

- **metanorm** (Core function of the Metanorm R package that performs the batch- and drift-correcting normalization (tGAM/rGAM/rLOESS/QC-RLSC models) and writes diagnostic plots) — github.com/UGent-LIMET/Metanorm
- **Metanorm R package** (R package integrating the robust normalization methods with visualization utilities (metanorm, plotPCA) used for batch correction and performance verification) — github.com/UGent-LIMET/Metanorm
- **tGAM** (Default and best-performing robust GAM-based normalization model used to correct drift and batch effects) — github.com/UGent-LIMET/Metanorm
- **rGAM** (Faster robust GAM alternative used when computational cost of tGAM is a limiting factor) — github.com/UGent-LIMET/Metanorm
- **rLOESS** (Faster robust LOESS alternative used when computational cost of tGAM is a limiting factor) — github.com/UGent-LIMET/Metanorm
- **QC-RLSC** (Classical QC-only normalization model (run with QConly = TRUE) used as a comparison baseline; sensitive to outliers in biological samples) — github.com/UGent-LIMET/Metanorm
- **plotPCA** (Generates pre- vs. post-normalization PC score plots labeled by batch to verify that batch effects diminished) — github.com/UGent-LIMET/Metanorm
- **R** (Runtime environment required (version >= 4.4.0) for installing and running the Metanorm package)
- **pak** (R package installer used to install Metanorm directly from GitHub (pak::pak('UGent-LIMET/Metanorm')))

## Examples

```
load(system.file("extdata", "example.RData", package = "metanorm")); normdat <- metanorm(rawdata[1:5,], model = "tGAM", type = metanorm.qc, QCcheck = TRUE, batch = batch, plotdir = "~/Documents/metanormExample/")
```

## Evaluation signals

- Pre- vs. post-normalization plotPCA score plots labeled by batch should show diminished batch separation after metanorm() (README demonstrates this for the example dataset).
- Per-compound intensity-vs-order plots in plotdir should show a smooth, reduced-drift trajectory after normalization; residual drift or spike-like outliers in biological samples indicate failure or need for a robust model (e.g. tGAM) instead of QC-RLSC.
- Replicate concordance should improve and drift/batch effects should decrease relative to popular existing strategies, as reported for the robust methods in the article.
- QCcheck = TRUE should not raise QC-vs-sample representativeness discrepancies for representative QCs; a flagged mismatch is a signal to inspect data quality rather than trust the normalized matrix.
- On the package's example dataset (first 5 compounds), calling metanorm(rawdata[1:5,], model='tGAM', type=metanorm.qc, QCcheck=TRUE, batch=batch, plotdir=...) must return a matrix of the same dimensions as rawdata[1:5,] with numeric entries.

## Limitations

- Outliers (occasional disproportionately high or low metabolite intensities) deteriorate existing normalization methods — hence robust models (tGAM/rGAM/rLOESS) are preferred; QC-RLSC normalizing on QCs only is vulnerable to outliers in biological samples.
- tGAM has a higher computational cost than rGAM and rLOESS, though normalization of thousands of metabolites and hundreds of samples remains feasible within minutes on current hardware.
- Reduction of RSD of QC samples or increased numbers of differentially abundant metabolites are common benchmarking pitfalls and should not be used as proxies for good normalization.
- Differences between QC and biological samples may invalidate RSD/CV-based filtering on QC samples; QC and replicate samples should both be included to support quality and normalization performance investigation.
- Machine learning-based normalization methods' reliability on small datasets has not been broadly investigated.
- Metanorm's GAM-based procedures can revert to sample-based normalization when QCs are non-representative; manual outlier identification is subjective and labor-intensive at large feature counts.

## Evidence

- [readme] Metanorm supports robust metabolomics data normalization across scales and experimental designs.: "Metanorm supports robust metabolomics data normalization across scales and experimental designs."
- [readme] tGAM is recommended due to its superior robustness, but rGAM and rLOESS are faster.: "tGAM is recommended due to its superior robustness, but rGAM and rLOESS are faster."
- [readme] We further recommend using both QC as well as biological samples for normalization, and to have metanorm check for discrepancies between QC and biological samples.: "We further recommend using both QC as well as biological samples for normalization, and to have metanorm check for discrepancies between QC and biological samples."
- [abstract] the robust methods, relative to several popular existing strategies, improved replicate concordance, and reduced drift and batch effects: "the robust methods, relative to several popular existing strategies, improved replicate concordance, and reduced drift and batch effects"
- [abstract] Overall, tGAM-based normalization suggested the best performance across scenarios and is proposed as default choice.: "Overall, tGAM-based normalization suggested the best performance across scenarios and is proposed as default choice."
- [intro] we demonstrate that the presence of occasional, disproportionately high or low metabolite intensities (outliers) can deteriorate the performance of existing normalization methods: "we demonstrate that the presence of occasional, disproportionately high or low metabolite intensities (outliers) can deteriorate the performance of existing normalization methods"
- [intro] Given common pitfalls in benchmarking normalization strategies, such as using a reduction of the relative standard deviation of QC samples: "Given common pitfalls in benchmarking normalization strategies, such as using a reduction of the relative standard deviation of QC samples or an increased number of differentially abundant"
- [methods] Make sure `R` (**version >= 4.4.0**) is installed on your computer: "Make sure `R` (**version >= 4.4.0**) is installed on your computer"
- [methods] load(system.file("extdata", "example.RData", package = "metanorm")): "load(system.file("extdata", "example.RData", package = "metanorm"))"
- [readme] The example dataset contains three objects:
- *rawdata*, a numerical matrix containing the unnormalized data (rows = compounds, columns = samples)
- *batch*, a vector containing for each sample run the batch to which it belongs
- *metanorm.qc*, a vector containing for each sample run whether it is a QC or another type of sample (not a QC): "The example dataset contains three objects:"
- [readme] While no apparent batch effects remain in the PC score plot, you might want to check the pre- versus post-normalization plot number 4, for example! What does the signal drift look like in the biological (= non-QC) samples?: "While no apparent batch effects remain in the PC score plot, you might want to check the pre- versus post-normalization plot number 4, for example! What does the signal drift look like in the"
- [discussion] Their implementation in the Metanorm R package facilitates robust, high-throughput, and efficient normalization: "their implementation in the Metanorm R package facilitates robust, high-throughput, and efficient normalization"
- [discussion] A practical limitation of tGAM is its higher computational cost relative to rLOESS and rGAM; however, normalization remains computationally feasible, completing within minutes on current hardware.: "A practical limitation of tGAM is its higher computational cost relative to rLOESS and rGAM"
- [discussion] We recommend inclusion of both QC and replicate samples to support data quality and normalization performance investigation: "We recommend inclusion of both QC and replicate samples to support data quality and normalization performance investigation"
