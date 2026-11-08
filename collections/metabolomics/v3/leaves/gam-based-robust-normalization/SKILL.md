---
name: gam-based-robust-normalization
description: Use when an unnormalized metabolomics feature matrix (rows = compounds,
  columns = sample runs) contains outliers or QC/sample drift, and robust per-metabolite
  drift and batch-effect correction is needed before downstream DA testing.
when_to_use_negative:
- When the input is not a feature-by-sample intensity matrix but already normalized
  data, metadata tables, or raw instrument files (mzML/NetCDF) that have not been
  peak-picked into a matrix
- When QC samples are not representative of biological samples — run metanorm with
  QCcheck = TRUE first, and if QC/sample discrepancies persist, QC-based drift models
  may be inappropriate
- When cross-metabolite (multivariate) normalization is required — Metanorm's methods
  are feature-level and do not leverage cross-metabolite information
edam_operation: http://edamontology.org/operation_3660
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: Metanorm R package
  role: Implements the robust normalization methods (tGAM, rGAM, rLOESS) via the metanorm()
    function with batch correction, QCcheck and diagnostic plotting
  repo: github.com/UGent-LIMET/Metanorm
- name: metanorm
  role: Main normalization function called with model = 'tGAM'/'rGAM'/'rLOESS', type,
    QCcheck = TRUE, batch and plotdir arguments
  repo: github.com/UGent-LIMET/Metanorm
- name: plotPCA
  role: Generates PC score plots before and after normalization, labelled by batch,
    to verify batch-effect reduction
  repo: github.com/UGent-LIMET/Metanorm
- name: R
  role: Runtime environment; Metanorm requires R >= 4.4.0
  repo: https://cloud.r-project.org/index.html
- name: pak
  role: Used to install Metanorm directly from GitHub via pak::pak("UGent-LIMET/Metanorm")
provenance:
  source_task_ids:
  - task_004
  - task_007
  source_papers:
  - doi: 10.1101/2025.09.30.679445
    title: Robust metabolomics data normalization across scales and experimental designs
  source_content_hash: sha256:82cbbb19673ea6898f8c8a924d234d2ff0f9b63f058a735a778e87cb1e670343
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Metanorm R package
  - R
  - ggplot2
  - metanorm
  - pak
  - plotPCA
  - rGAM
  - rLOESS
  - tGAM
  edam_operation: http://edamontology.org/operation_3660
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 22
---

# gam-based-robust-normalization

## Summary

Normalize LC-MS metabolomics intensity matrices with the GAM-based robust methods (tGAM default, rGAM, rLOESS) implemented in the Metanorm R package, which downweight or accommodate outliers that degrade QC-based drift correction such as QC-RLSC.

## When to use

Use when an unnormalized metabolomics feature matrix (rows = compounds, columns = sample runs) contains outliers or QC/sample drift, and robust per-metabolite drift and batch-effect correction is needed before downstream DA testing.

## When NOT to use

- When the input is not a feature-by-sample intensity matrix but already normalized data, metadata tables, or raw instrument files (mzML/NetCDF) that have not been peak-picked into a matrix
- When QC samples are not representative of biological samples — run metanorm with QCcheck = TRUE first, and if QC/sample discrepancies persist, QC-based drift models may be inappropriate
- When cross-metabolite (multivariate) normalization is required — Metanorm's methods are feature-level and do not leverage cross-metabolite information

## Inputs

- rawdata: numerical intensity matrix (rows = compounds, columns = sample runs), e.g. from the metanorm example.RData extdata
- type/metadata vector marking each run as "QC" or other sample type (metanorm.qc)
- batch vector assigning each sample run to a batch

## Outputs

- Normalized intensity matrix (normdat) of the same dimensions as rawdata
- Per-compound pre- vs post-normalization intensity-vs-order diagnostic plots written to plotdir
- Pre- and post-normalization PC score plots labelled by batch via plotPCA

## How to apply

Install the Metanorm R package with pak::pak("UGent-LIMET/Metanorm") under R >= 4.4.0 and load rawdata (numerical matrix, rows = compounds, columns = samples), batch, and the sample-type vector (e.g. metanorm.qc with values "QC"/"sample"). Call metanorm() with the default model = "tGAM" (or rGAM/rLOESS for speed), QCcheck = TRUE to verify QCs are representative of biological samples, batch = batch for batch-wise correction, and a plotdir to write per-compound pre- vs post-normalization intensity-vs-order diagnostic plots. The GAM/LOESS fits are cross-validated (selectSpan exposes GCV vs LOOCV span selection), and the robust formulations downweight or accommodate disproportionate high/low intensities, so QC samples and biological samples can both drive the fit, unlike QC-RLSC's QConly mode. Inspect a decent number of diagnostic plots and run plotPCA on pre- and post-normalization data labelled by batch to confirm batch effects and drift have diminished. Across benchmarks, tGAM showed the strongest performance and is the recommended default, at higher computational cost than rLOESS/rGAM (still minutes for thousands of metabolites and hundreds of samples).

## Related tools

- **Metanorm R package** (Implements the robust normalization methods (tGAM, rGAM, rLOESS) via the metanorm() function with batch correction, QCcheck and diagnostic plotting) — github.com/UGent-LIMET/Metanorm
- **metanorm** (Main normalization function called with model = 'tGAM'/'rGAM'/'rLOESS', type, QCcheck = TRUE, batch and plotdir arguments) — github.com/UGent-LIMET/Metanorm
- **plotPCA** (Generates PC score plots before and after normalization, labelled by batch, to verify batch-effect reduction) — github.com/UGent-LIMET/Metanorm
- **R** (Runtime environment; Metanorm requires R >= 4.4.0) — https://cloud.r-project.org/index.html
- **pak** (Used to install Metanorm directly from GitHub via pak::pak("UGent-LIMET/Metanorm"))

## Examples

```
normdat <- metanorm(rawdata, model = "tGAM", type = metanorm.qc, QCcheck = TRUE, batch = batch, plotdir = "~/metanormOut/")
```

## Evaluation signals

- QCcheck = TRUE passes: QCs are confirmed representative of biological samples before fitting drift
- Post-normalization plotPCA shows diminished batch effects relative to the pre-normalization plot labelled by batch
- Per-compound diagnostic plots in plotdir show drift curves that accommodate rather than chase outliers, with reduced variance near previously outlying intensities
- tGAM (default) reproduces the paper's best performance; on benchmarks such as the FAME cohort (618 runs x 8913 features), robust methods agree strongly (79-87% DAM overlap) versus QC-RLSC (55-59%)
- GCV vs LOOCV span selection yields highly comparable DAM counts and p-value orderings

## Limitations

- tGAM has a higher computational cost than rLOESS and rGAM, though it still completes within minutes for thousands of metabolites and hundreds of samples
- Outlier treatment is feature-level; multivariate outlier detection may improve over per-metabolite robust fitting
- Cross-metabolite information is not leveraged by these metabolite-level normalization approaches
- If QC samples differ from biological samples, QC-based strategies (e.g. QC-RLSC with QConly = TRUE) degrade in biological samples under outlier contamination; robust methods avoid this by using both QC and sample information

## Evidence

- [abstract] We introduce rLOESS, rGAM and tGAM, three robust normalization methods that improve resistance to outliers by downweighting or accommodating them.: "We introduce rLOESS, rGAM and tGAM, three robust normalization methods that improve resistance to outliers by downweighting or accommodating them."
- [abstract] tGAM default recommendation: "Overall, tGAM-based normalization suggested the best performance across scenarios and is proposed as default choice."
- [readme] metanorm call arguments: "QCcheck = TRUE,      # check whether QCs are representative"
- [readme] plot PCA usage: "We can generate a PC score plot to see whether batch effects have diminished"
- [readme] diagnostic plot recommendation: "It is highly recommended to look at a decent number of these plots to judge normalization performance."
- [methods] install command: "pak::pak("UGent-LIMET/Metanorm")"
