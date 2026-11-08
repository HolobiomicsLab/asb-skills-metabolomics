---
name: robust-gam-signal-drift-correction
description: Use when an unnormalized LC-MS metabolomics intensity matrix shows signal
  drift or batch effects over run order, contains occasional disproportionately high/low
  metabolite intensities (outliers), and QC plus replicate samples are available to
  guide and evaluate normalization.
when_to_use_negative:
- Data are already normalized or already processed with outlier detection and removal
  prior to normalization — robust drift correction targets raw unnormalized intensities
- The dataset contains no QC samples (or QCs are not representative of biological
  samples); QC-based normalization, including the QCcheck step, depends on QC runs
- Evaluating QC-RSD reduction or increased numbers of differentially abundant metabolites
  as the primary goal — the paper explicitly flags these as common benchmarking pitfalls
edam_operation: http://edamontology.org/operation_3661
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0203
tools:
- name: metanorm (Metanorm R package)
  role: Implements tGAM, rGAM, rLOESS, QC-RLSC, QC-RSC normalization plus QCcheck,
    plotdir diagnostics, and plotPCA visualization
  repo: github.com/UGent-LIMET/Metanorm
- name: R
  role: Runtime environment (version >= 4.4.0) in which Metanorm is installed and
    executed
  repo: https://cloud.r-project.org/index.html
- name: pak
  role: Used to install the Metanorm package from GitHub
  repo: github.com/UGent-LIMET/Metanorm
- name: tGAM
  role: Default robust GAM-based drift-correction model, recommended as general default
    choice
  repo: github.com/UGent-LIMET/Metanorm
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1101/2025.09.30.679445
    title: Robust metabolomics data normalization across scales and experimental designs
  source_content_hash: sha256:3224b8f3a78d404aab694373c9641e531ebd4bf6fc63650d321986b2c8fea5eb
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - fANCOVA
  - ggplot2
  - metanorm
  - metanorm (Metanorm R package)
  - mgcv
  - pak
  - pbapply
  - rGAM
  - rLOESS
  - tGAM
  edam_operation: http://edamontology.org/operation_3661
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 21
---

# robust-gam-signal-drift-correction

## Summary

Outlier-robust, QC-sample-based signal drift correction for LC-MS metabolomics intensity matrices using the Metanorm R package's tGAM (generalized additive model) method, which downweights or accommodates outliers that break classical QC-based normalizers like QC-RLSC and QC-RSC.

## When to use

Use when an unnormalized LC-MS metabolomics intensity matrix shows signal drift or batch effects over run order, contains occasional disproportionately high/low metabolite intensities (outliers), and QC plus replicate samples are available to guide and evaluate normalization.

## When NOT to use

- Data are already normalized or already processed with outlier detection and removal prior to normalization — robust drift correction targets raw unnormalized intensities
- The dataset contains no QC samples (or QCs are not representative of biological samples); QC-based normalization, including the QCcheck step, depends on QC runs
- Evaluating QC-RSD reduction or increased numbers of differentially abundant metabolites as the primary goal — the paper explicitly flags these as common benchmarking pitfalls

## Inputs

- Unnormalized LC-MS metabolomics intensity matrix (numerical, rows = compounds, columns = samples), e.g. BioHEART 1359 plasma runs x 53 metabolites
- Vector of sample types per run ('QC' vs other sample types), e.g. metanorm.qc
- Vector of batch labels per sample run, e.g. batch

## Outputs

- Normalized intensity matrix (normdat) with drift and batch effects corrected
- Pre- vs post-normalization per-compound intensity-vs-order diagnostic plots written to plotdir
- PC score plots (plotPCA) before and after normalization, labeled by batch
- QC-representativeness check result from QCcheck = TRUE

## How to apply

Load the raw intensity matrix (rows = compounds, columns = samples), a batch vector, and a QC-vs-sample type vector into R (>= 4.4.0); install Metanorm from GitHub via pak::pak("UGent-LIMET/Metanorm"). Call metanorm() with model = "tGAM" (the paper's recommended default), type = the QC/sample vector, batch = the batch vector, QCcheck = TRUE to verify QCs are representative of biological samples, and plotdir for diagnostics. Unlike QC-RLSC/QC-RSC (which use QConly = TRUE), tGAM normalizes using both QC and biological samples, and its GAM basis flexibly captures non-linear drift with observation weighting for outliers. Inspect the per-compound intensity-vs-order plots written to plotdir and compare pre/post-normalization plotPCA() score plots labeled by batch to confirm drift and batch effects were removed in biological (non-QC) samples. Quantify success via between-replicate MAD reductions (targets on BioHEART: 49-52% for robust methods vs 18-22% for QC-RL(R)SC) and joint differentially-abundant-metabolite overlaps across methods (92% among robust methods, 29-41% for SERFF).

## Related tools

- **metanorm (Metanorm R package)** (Implements tGAM, rGAM, rLOESS, QC-RLSC, QC-RSC normalization plus QCcheck, plotdir diagnostics, and plotPCA visualization) — github.com/UGent-LIMET/Metanorm
- **R** (Runtime environment (version >= 4.4.0) in which Metanorm is installed and executed) — https://cloud.r-project.org/index.html
- **pak** (Used to install the Metanorm package from GitHub) — github.com/UGent-LIMET/Metanorm
- **tGAM** (Default robust GAM-based drift-correction model, recommended as general default choice) — github.com/UGent-LIMET/Metanorm

## Examples

```
normdat <- metanorm(rawdata, model = "tGAM", type = metanorm.qc, QCcheck = TRUE, batch = batch, plotdir = "~/metanormDiagnostics/")
```

## Evaluation signals

- Between-replicate normalized-difference MAD reduction vs unnormalized data falls in the robust-method range (49-52% on BioHEART), not the QC-RLSC/QC-RSC range (18-22%)
- plotPCA() score plots labeled by batch show diminished batch effects after normalization
- Per-compound intensity-vs-order plots in plotdir show drift removed in biological (non-QC) samples — check a decent number, not just one
- Joint-DAM overlap with other robust methods reaches ~92% on method pairs (vs 29-41% for SERFF), matching Figs. 3-4 reference values
- QCcheck = TRUE reports that QCs are representative of biological samples before normalization is trusted

## Limitations

- tGAM has higher computational cost than rLOESS and rGAM, though normalization completes within minutes for thousands of metabolites and hundreds of samples
- Metanorm's GAM-based procedures may adaptively revert to sample-based normalization; feature-level outlier treatment could be improved with multivariate outlier detection
- The paper recommends inclusion of both QC and replicate samples; datasets lacking these cannot support the QC-representativeness check or replicate-concordance evaluation
- Filtering on RSD observed in QC samples can be invalidated by differences between QC and biological samples

## Evidence

- [abstract] We introduce rLOESS, rGAM and tGAM, three robust normalization methods that improve resistance to outliers by downweighting or accommodating them.: "We introduce rLOESS, rGAM and tGAM, three robust normalization methods that improve resistance to outliers by downweighting or accommodating them."
- [abstract] Overall, tGAM-based normalization suggested the best performance across scenarios and is proposed as default choice.: "Overall, tGAM-based normalization suggested the best performance across scenarios and is proposed as default choice."
- [readme] Make sure `R` (**version >= 4.4.0**) is installed on your computer: "Make sure `R` (**version >= 4.4.0**) is installed on your computer"
- [methods] pak::pak("UGent-LIMET/Metanorm"): "if (!require("pak", quietly = TRUE))
    install.packages("pak")"
- [readme] metanorm function call with QCcheck and batch arguments: "QCcheck = TRUE,      # check whether QCs are representative
                    batch = batch,       # normalize by batch"
- [readme] We further recommend using both QC as well as biological samples for normalization, and to have metanorm check for discrepancies between QC and biological samples.: "We further recommend using both QC as well as biological samples for normalization, and to have metanorm check for discrepancies between QC and biological samples."
- [abstract] the robust methods, relative to several popular existing strategies, improved replicate concordance, and reduced drift and batch effects: "the robust methods, relative to several popular existing strategies, improved replicate concordance, and reduced drift and batch effects"
- [discussion] The pre- vs. post-normalization and PCA score plots readily generated by the Metanorm package support an efficient identification of subpar data quality or aberrant normalization: "The pre- vs. post-normalization and PCA score plots readily generated by the Metanorm package support an efficient identification of subpar data quality or aberrant normalization"
