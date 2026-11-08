---
name: lc-ms-qc-based-normalization
description: Use when an unnormalized LC-MS metabolomics intensity matrix (e.g., runs
  x metabolites with QC annotations and batch labels) shows signal drift, batch effects,
  or outliers that degrade replicate concordance and downstream differential abundance
  testing.
when_to_use_negative:
- Input is already normalized (the workflow starts from a raw intensity matrix such
  as example.RData rawdata)
- Data contain no QC samples or QCs are not representative of biological samples —
  metanorm() with QCcheck = TRUE is designed to flag such QC-vs-sample discrepancies
  before normalization
- The goal is automated outlier detection/removal prior to normalization rather than
  outlier-robust fitting; the paper treats outlier removal as a separate, non-trivial
  alternative
edam_topics:
- http://edamontology.org/topic_3315
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0157
tools:
- name: metanorm
  role: R package providing the metanorm() normalization function (tGAM, rGAM, rLOESS,
    QC-RLSC, QC-RSC), QCcheck, batchwise normalization and diagnostic plotting
  repo: github.com/UGent-LIMET/Metanorm
- name: Metanorm R package
  role: Integrated implementation of robust and legacy QC-based normalization with
    visualization for performance verification
  repo: github.com/UGent-LIMET/Metanorm
- name: tGAM
  role: Default, best-performing robust GAM-based normalization model; higher computational
    cost than rLOESS/rGAM
  repo: github.com/UGent-LIMET/Metanorm
- name: rGAM
  role: Faster robust additive-model alternative to tGAM
  repo: github.com/UGent-LIMET/Metanorm
- name: rLOESS
  role: Faster cross-validated robust LOESS alternative to tGAM
  repo: github.com/UGent-LIMET/Metanorm
- name: QC-RLSC
  role: Legacy QC-based strategy run with QConly = TRUE as prescribed, used for comparison
  repo: github.com/UGent-LIMET/Metanorm
- name: QC-RSC
  role: Legacy QC-based strategy supported by the package for benchmarking
  repo: github.com/UGent-LIMET/Metanorm
- name: pak
  role: Installer for the Metanorm package from GitHub
  repo: github.com/UGent-LIMET/Metanorm
- name: ggplot2
  role: Rendering PC score plots and comparison heatmaps
- name: plotPCA
  role: Pre- vs. post-normalization PC score plots labeled by batch to verify batch-effect
    reduction
  repo: github.com/UGent-LIMET/Metanorm
provenance:
  source_task_ids:
  - task_002
  - task_003
  source_papers:
  - doi: 10.1101/2025.09.30.679445
    title: Robust metabolomics data normalization across scales and experimental designs
  source_content_hash: sha256:1f1608d1a1d237629998c17320d37723d94519e4f48a5d68acf7f52c4a09b063
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Metanorm R package
  - QC-RLSC
  - QC-RSC
  - R
  - cowplot
  - fANCOVA
  - ggplot2
  - metanorm
  - mgcv
  - pak
  - pbapply
  - plotPCA
  - rGAM
  - rLOESS
  - tGAM
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3315
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0157
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7575757575757576
n_claims: 33
---

# lc-ms-qc-based-normalization

## Summary

Quality-control (QC) sample-based normalization of LC-MS metabolomics intensity matrices, extended with three outlier-robust methods (rLOESS, rGAM, tGAM) implemented in the Metanorm R package, with QC-representativeness checking, batchwise correction, and diagnostic visualization.

## When to use

Use when an unnormalized LC-MS metabolomics intensity matrix (e.g., runs x metabolites with QC annotations and batch labels) shows signal drift, batch effects, or outliers that degrade replicate concordance and downstream differential abundance testing.

## When NOT to use

- Input is already normalized (the workflow starts from a raw intensity matrix such as example.RData rawdata)
- Data contain no QC samples or QCs are not representative of biological samples — metanorm() with QCcheck = TRUE is designed to flag such QC-vs-sample discrepancies before normalization
- The goal is automated outlier detection/removal prior to normalization rather than outlier-robust fitting; the paper treats outlier removal as a separate, non-trivial alternative

## Inputs

- Unnormalized LC-MS intensity matrix (e.g., 1359 plasma runs x 53 metabolites BioHEART; 393 runs x 269 metabolites ENVIRONAGE urine), rows = compounds, columns = sample runs
- Sample-type vector labeling each run as 'QC' or other (metanorm.qc)
- Batch annotation vector (e.g., 'batch1', ...) per run
- Optional replicate-sample annotation for concordance evaluation

## Outputs

- Normalized intensity matrix (same dimensions as input)
- Per-compound pre- vs. post-normalization intensity-vs-order diagnostic plots written to plotdir
- PC score plots before and after normalization labeled by batch
- Between-replicate MAD reduction percentages and differential abundance testing (DAM) counts/overlaps for method comparison

## How to apply

Load the unnormalized matrix (rows = compounds, columns = sample runs), a type vector labeling QC vs. non-QC runs, and a batch vector into R (>= 4.4.0) after installing Metanorm via pak::pak("UGent-LIMET/Metanorm"). Call metanorm() per feature with model = "tGAM" (the recommended default; rGAM and rLOESS are faster alternatives, QC-RLSC/QC-RSC as legacy options with QConly = TRUE as prescribed), QCcheck = TRUE to verify QCs are representative of biological samples, and batch = batch for batchwise normalization, writing per-compound drift plots to plotdir. Inspect pre- vs. post-normalization PC score plots labeled by batch via plotPCA(), and manually review a decent number of per-compound intensity-vs-order plots to confirm drift is removed in biological samples — the README warns that batch effects can look resolved in PCA while individual compounds remain misnormalized (e.g., plot number 4 in the QC-RLSC example). Assess success via between-replicate MAD reduction relative to unnormalized data (paper reference values: 49-52% for robust methods, 41% SERFF, 40% TIGER, 18-22% QC-RLSC/QC-RSC) rather than QC-RSD reduction or increased DAM counts, which the authors flag as benchmarking pitfalls.

## Related tools

- **metanorm** (R package providing the metanorm() normalization function (tGAM, rGAM, rLOESS, QC-RLSC, QC-RSC), QCcheck, batchwise normalization and diagnostic plotting) — github.com/UGent-LIMET/Metanorm
- **Metanorm R package** (Integrated implementation of robust and legacy QC-based normalization with visualization for performance verification) — github.com/UGent-LIMET/Metanorm
- **tGAM** (Default, best-performing robust GAM-based normalization model; higher computational cost than rLOESS/rGAM) — github.com/UGent-LIMET/Metanorm
- **rGAM** (Faster robust additive-model alternative to tGAM) — github.com/UGent-LIMET/Metanorm
- **rLOESS** (Faster cross-validated robust LOESS alternative to tGAM) — github.com/UGent-LIMET/Metanorm
- **QC-RLSC** (Legacy QC-based strategy run with QConly = TRUE as prescribed, used for comparison) — github.com/UGent-LIMET/Metanorm
- **QC-RSC** (Legacy QC-based strategy supported by the package for benchmarking) — github.com/UGent-LIMET/Metanorm
- **pak** (Installer for the Metanorm package from GitHub) — github.com/UGent-LIMET/Metanorm
- **ggplot2** (Rendering PC score plots and comparison heatmaps)
- **plotPCA** (Pre- vs. post-normalization PC score plots labeled by batch to verify batch-effect reduction) — github.com/UGent-LIMET/Metanorm

## Examples

```
library(metanorm); normdat <- metanorm(rawdata[1:5,], model = "tGAM", type = metanorm.qc, QCcheck = TRUE, batch = batch, plotdir = "~/metanormExample/")
```

## Evaluation signals

- Between-replicate MAD reduction vs. unnormalized data falls in the reported ranges: 49-52% for robust methods, 41% SERFF, 40% TIGER, 18-22% QC-RLSC/QC-RSC (Figs. 3-4)
- Joint-DAM overlap percentages match reported values (e.g., 92% among robust methods, 29-41% SERFF; on ENVIRONAGE robust 74-88%, QC-RLSC 47-51%, SERFF 68-70%, TIGER 64-67%)
- PC score plots labeled by batch show diminished batch effects after normalization, matching the paper's pre/post figures
- Per-compound intensity-vs-order plots in plotdir show drift corrected in biological (non-QC) samples — the README's QC-RLSC example deliberately hides residual drift visible in plot number 4
- QCcheck = TRUE reports no QC-vs-biological-sample discrepancy; downstream differential abundance testing uses multiple testing correction before interpreting DAM sets

## Limitations

- tGAM has a higher computational cost than rLOESS and rGAM, though it completes within minutes on current hardware for thousands of metabolites and hundreds of samples
- Occasional disproportionately high or low metabolite intensities (outliers) deteriorate the performance of existing QC-based methods (QC-RLSC, QC-RSC), which is why robust fitting downweighting/accommodating outliers is recommended
- Package requires R >= 4.4.0; an exact released Metanorm version is not pinned in the paper's discussion
- Reduced QC relative standard deviation (RSD/CV filtering on QCs) or increased DAM counts must not be used as success proxies; filtering on QC RSD can be invalidated by QC-vs-biological-sample differences
- Performance on simulated data was comparable between rLOESS, rGAM and tGAM; tGAM's superiority was established on experimental datasets
