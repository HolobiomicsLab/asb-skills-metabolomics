---
name: qc-rlsc-normalization
description: Use when an unnormalized feature matrix (compounds × runs) with sample-type
  (QC vs. biological) and batch vectors needs drift correction by QC-driven robust
  regression, or when comparing QC-RLSC against robust alternatives (rLOESS, rGAM,
  tGAM).
when_to_use_negative:
- 'When outlier-robust treatment of biological-sample intensities is required: outliers
  in non-QC samples deteriorate QC-RLSC performance since it fits drift from QC samples
  only (QConly = TRUE)'
- When normalization should use both QC and biological samples — the Metanorm README
  recommends this, but it conflicts with QC-RLSC's prescribed QConly usage
- As a proxy benchmark for good normalization via increased numbers of differentially
  abundant metabolites or reduced QC RSD — the article flags these as common benchmarking
  pitfalls
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: Metanorm R package
  role: Provides the metanorm() function with model = 'QC-RLSC' and the QConly, type,
    batch, QCcheck, plotdir and cv arguments used for QC-driven normalization and
    diagnostics
  repo: github.com/UGent-LIMET/Metanorm
- name: plotPCA
  role: Generates PC score plots of rawdata vs. normalized data labeled by batch to
    verify batch-effect removal
  repo: github.com/UGent-LIMET/Metanorm
- name: R
  role: Runtime environment required at version >= 4.4.0
- name: pak
  role: Used to install the Metanorm package via pak::pak("UGent-LIMET/Metanorm")
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1101/2025.09.30.679445
    title: Robust metabolomics data normalization across scales and experimental designs
  source_content_hash: sha256:6de91daf8dd2ec95030b19b2238812c747b7026724d426b6902da5c813a31aba
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Metanorm R package
  - QC-RLSC
  - R
  - ggplot2
  - metanorm
  - pak
  - plotPCA
  - rGAM
  - rLOESS
  - tGAM
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7058823529411765
n_claims: 17
---

# qc-rlsc-normalization

## Summary

Applies QC-RLSC quality-control-sample-based normalization to LC-MS metabolomics data via the metanorm() function of the Metanorm R package, fitting signal drift from QC runs only (QConly = TRUE), batch-wise, with GCV or LOOCV span selection.

## When to use

Use when an unnormalized feature matrix (compounds × runs) with sample-type (QC vs. biological) and batch vectors needs drift correction by QC-driven robust regression, or when comparing QC-RLSC against robust alternatives (rLOESS, rGAM, tGAM).

## When NOT to use

- When outlier-robust treatment of biological-sample intensities is required: outliers in non-QC samples deteriorate QC-RLSC performance since it fits drift from QC samples only (QConly = TRUE)
- When normalization should use both QC and biological samples — the Metanorm README recommends this, but it conflicts with QC-RLSC's prescribed QConly usage
- As a proxy benchmark for good normalization via increased numbers of differentially abundant metabolites or reduced QC RSD — the article flags these as common benchmarking pitfalls

## Inputs

- rawdata: numerical matrix of unnormalized intensities (rows = compounds, columns = sample runs), log-scale values expected
- type vector: per-run sample type labels ('QC' vs. other) — e.g. metanorm.qc
- batch vector: per-run batch labels (e.g. 'batch1')
- plotdir: directory path for diagnostic plots
- cv setting: GCV or LOOCV span/smoother selection

## Outputs

- normdat: normalized data matrix
- per-compound pre- vs. post-normalization intensity vs. run-order plots in plotdir
- pre/post-normalization PCA score plots labeled by batch (plotPCA)
- downstream differential abundance (DAM) test results and cross-method overlap percentages

## How to apply

Load the rawdata matrix (rows = compounds, columns = sample runs) together with the type vector ('QC' vs other) and batch vector, e.g. from the Metanorm example.RData. Call metanorm() with model = 'QC-RLSC' and QConly = TRUE so that, as prescribed, only QC samples drive the fitted drift, passing QCcheck = TRUE, batch = batch, and a plotdir for per-compound pre/post-normalization diagnostics; span/smoother selection is controlled via the cv argument (GCV or LOOCV). Judge normalization by inspecting the generated individual-compound intensity-vs-run-order plots and by plotPCA() before/after labeling by batch — the README warns that PC score plots alone can hide drift in biological (non-QC) samples (see example compound 4). In comparative evaluations (e.g. FAME saliva, 618 runs × 8913 features), QC-RLSC showed weaker agreement with the robust methods (55–59% DAM overlap) than the robust methods among themselves (79–87%).

## Related tools

- **Metanorm R package** (Provides the metanorm() function with model = 'QC-RLSC' and the QConly, type, batch, QCcheck, plotdir and cv arguments used for QC-driven normalization and diagnostics) — github.com/UGent-LIMET/Metanorm
- **plotPCA** (Generates PC score plots of rawdata vs. normalized data labeled by batch to verify batch-effect removal) — github.com/UGent-LIMET/Metanorm
- **R** (Runtime environment required at version >= 4.4.0)
- **pak** (Used to install the Metanorm package via pak::pak("UGent-LIMET/Metanorm"))

## Examples

```
normdat2 <- metanorm(rawdata[1:5,], model = "QC-RLSC", type = metanorm.qc, QConly = TRUE, batch = batch, plotdir = "~/Documents/metanormExample2/")
```

## Evaluation signals

- Pre/post-normalization PCA score plots labeled by batch show diminished batch effects after QC-RLSC normalization
- Per-compound diagnostic plots in plotdir show fitted drift tracks non-QC sample behavior (README example compound 4 demonstrates where QC-RLSC can fail)
- QCcheck = TRUE confirms QCs are representative of biological samples before trusting the QC-driven fit
- In cross-method DAM comparisons on the FAME cohort, QC-RLSC overlap with the robust methods should fall in the reported 55–59% band (vs. 79–87% among robust methods)
- GCV vs. LOOCV span selection should yield highly comparable DAM counts and p-value orderings

## Limitations

- Outliers in biological (non-QC) samples are not downweighted, which can deteriorate QC-RLSC performance and reduce statistical power in downstream differential abundance testing
- PC score plots alone may suggest no residual batch effects while signal drift in biological samples remains (README compound 4 example)
- An alternative to robust normalization is automated outlier detection/removal prior to normalization, but this is non-trivial, and manual outlier identification is subjective and labor-intensive for large feature counts

## Evidence

- [readme] QC-RLSC prescribed QC-only fitting: "To see why this is important, you can now normalize with QC-RLSC, using, as prescribed, QC samples for normalization (*QConly* argument):"
- [readme] PC plots can miss drift: "While no apparent batch effects remain in the PC score plot, you might want to check the pre- versus post-normalization plot number 4, for example!"
- [intro] Outliers deteriorate existing methods: "the presence of occasional, disproportionately high or low metabolite intensities (outliers) can deteriorate the performance of existing normalization methods"
- [discussion] Alternative: outlier removal before normalization: "An alternative to the use of the outlier-robust normalization procedures introduced here could be outlier detection and removal prior to normalization."
- [abstract] Package scope: "Implementations of these methods are gathered in the Metanorm R package, integrating robust normalization with visualization for performance verification"
