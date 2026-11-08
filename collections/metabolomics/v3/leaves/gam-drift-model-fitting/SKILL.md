---
name: gam-drift-model-fitting
description: Use when unnormalized LC-MS metabolomics intensity matrices show signal
  drift over injection order and/or batch effects, and QC-RLSC-style normalization
  is vulnerable to occasional disproportionately high/low outlier intensities.
when_to_use_negative:
- When only QC samples should drive normalization (e.g. prescribed QC-RLSC with QConly
  = TRUE) rather than robust combined QC+biological-sample fitting
- 'When computational cost is prohibitive: tGAM has higher cost than rGAM and rLOESS,
  which are faster alternatives with comparable performance on simulated data'
- When the goal is outlier detection and removal prior to normalization rather than
  outlier-robust fitting
tools:
- name: metanorm
  role: R package providing the metanorm() function with model = "tGAM", QCcheck,
    batch, weights, QConly, and plotdir arguments for GAM-based drift normalization
  repo: github.com/UGent-LIMET/Metanorm
- name: mgcv
  role: Fits the thin-plate-spline GAM by REML with gam.k and gam.frac settings underlying
    tGAM
- name: calcWeights
  role: Computes observation weights from residuals so high-residual (outlier) runs
    are downweighted in the GAM fit
  repo: github.com/UGent-LIMET/Metanorm
- name: plotPCA
  role: Generates PC score plots before and after normalization labeled by batch to
    confirm batch effects and drift have diminished
  repo: github.com/UGent-LIMET/Metanorm
- name: rGAM
  role: Faster robust additive-model alternative to tGAM within metanorm
  repo: github.com/UGent-LIMET/Metanorm
- name: rLOESS
  role: Faster cross-validated robust LOESS alternative to tGAM within metanorm
  repo: github.com/UGent-LIMET/Metanorm
provenance:
  source_task_ids:
  - task_006
  - task_003
  source_papers:
  - doi: 10.1101/2025.09.30.679445
    title: Robust metabolomics data normalization across scales and experimental designs
  source_content_hash: sha256:8043ece17e173121d17bd8eac602ba55e43060bb5cab14a03e77128937081638
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - calcWeights
  - changepoint
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
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7586206896551724
n_claims: 29
---

# gam-drift-model-fitting

## Summary

Fits thin-plate-spline generalized additive models (tGAM, via mgcv/REML) to model signal drift in LC-MS metabolomics run-order data, with batchwise fitting, QC-representativeness testing, and residual-based observation weighting for robust normalization.

## When to use

Use when unnormalized LC-MS metabolomics intensity matrices show signal drift over injection order and/or batch effects, and QC-RLSC-style normalization is vulnerable to occasional disproportionately high/low outlier intensities.

## When NOT to use

- When only QC samples should drive normalization (e.g. prescribed QC-RLSC with QConly = TRUE) rather than robust combined QC+biological-sample fitting
- When computational cost is prohibitive: tGAM has higher cost than rGAM and rLOESS, which are faster alternatives with comparable performance on simulated data
- When the goal is outlier detection and removal prior to normalization rather than outlier-robust fitting

## Inputs

- rawdata: numerical matrix of unnormalized LC-MS intensities (rows = compounds, columns = sample runs)
- batch: character vector giving the batch membership of each sample run
- metanorm.qc / type: vector labeling each run as 'QC' or another sample type
- weights: observation weights from calcWeights(residuals) for downweighting outlier runs

## Outputs

- normdat: normalized data matrix (e.g. saved as normdat.rds)
- Per-compound pre- vs post-normalization intensity-vs-order diagnostic plots written to plotdir
- plotPCA() PC score plots before and after normalization, labeled by batch

## How to apply

Load the metanorm R package (R >= 4.4.0) and prepare a rawdata matrix (rows = compounds, columns = sample runs), a batch vector, and a sample-type vector ('QC' vs other) such as metanorm.qc from example.RData. Call metanorm() with model = "tGAM" so the mgcv thin-plate-spline GAM fitted by REML (with gam.k and gam.frac settings) models drift, passing batch = batch to run batchwise fits per batch group and type = metanorm.qc so QCcheck = TRUE can add a 'type' covariate testing QC vs biological sample representativeness; if a significant difference is detected at the user-specified cut-off, the model is re-fitted and normalization uses biological samples only (QConly = FALSE). Supply observation weights computed with calcWeights(residuals) so high-residual (outlier) runs are downweighted in the GAM fit — this robustness is the key advantage over QC-RLSC. Set plotdir so pre- vs post-normalization intensity-vs-order diagnostic plots are written per compound, and verify with plotPCA() score plots labeled by batch. tGAM is slower than rGAM and rLOESS but is the recommended default because it exhibited the strongest performance across experimental datasets.

## Related tools

- **metanorm** (R package providing the metanorm() function with model = "tGAM", QCcheck, batch, weights, QConly, and plotdir arguments for GAM-based drift normalization) — github.com/UGent-LIMET/Metanorm
- **mgcv** (Fits the thin-plate-spline GAM by REML with gam.k and gam.frac settings underlying tGAM)
- **calcWeights** (Computes observation weights from residuals so high-residual (outlier) runs are downweighted in the GAM fit) — github.com/UGent-LIMET/Metanorm
- **plotPCA** (Generates PC score plots before and after normalization labeled by batch to confirm batch effects and drift have diminished) — github.com/UGent-LIMET/Metanorm
- **rGAM** (Faster robust additive-model alternative to tGAM within metanorm) — github.com/UGent-LIMET/Metanorm
- **rLOESS** (Faster cross-validated robust LOESS alternative to tGAM within metanorm) — github.com/UGent-LIMET/Metanorm

## Examples

```
normdat <- metanorm(rawdata[1:5,], model = "tGAM", type = metanorm.qc, QCcheck = TRUE, batch = batch, plotdir = "~/Documents/metanormExample/")
```

## Evaluation signals

- Pre- vs post-normalization PC score plots labeled by batch show batch effects and drift have diminished after tGAM normalization
- Per-compound intensity-vs-order plots in plotdir show reduced signal drift, including in biological (non-QC) samples — inspect a decent number of these plots
- Robust methods improve replicate concordance relative to popular existing strategies (e.g. QC-RLSC) without inflating DAM counts as a proxy
- QCcheck decision behaves as specified: when QC samples differ significantly from biological samples at the user-specified cut-off, the model is re-fitted and normalization uses biological samples only
- High-residual (outlier) runs receive low calcWeights weights and are accommodated rather than distorting the fitted drift curve

## Limitations

- tGAM has a higher computational cost than rGAM and rLOESS, though normalization completes within minutes for thousands of metabolites and hundreds of samples
- Outlier treatment is feature-level (per compound); multivariate outlier detection may improve on this but is not currently leveraged
- Differences between QC and biological samples may invalidate QC-based filtering such as RSD filtering, hence the QCcheck test
- Metanorm's GAM-based procedures revert to sample-based normalization adaptively when the QCcheck test fails

## Evidence

- [abstract] tGAM performance as default: "Overall, tGAM-based normalization suggested the best performance across scenarios and is proposed as default choice."
- [abstract] robustness rationale: "the robust methods, relative to several popular existing strategies, improved replicate concordance, and reduced drift and batch effects"
- [methods] QCcheck argument: "QCcheck = TRUE,      # check whether QCs are representative"
- [discussion] QCcheck re-fitting mechanism: [evidence span withheld — anchor status: not_found]
- [discussion] computational cost caveat: "A practical limitation of tGAM is its higher computational cost relative to rLOESS and rGAM; however, normalization remains computationally feasible"
- [results] batchwise model description: [evidence span withheld — anchor status: not_found]
- [readme] README recommended usage: "We further recommend using both QC as well as biological samples for normalization, and to have metanorm check for discrepancies between QC and biological samples."
