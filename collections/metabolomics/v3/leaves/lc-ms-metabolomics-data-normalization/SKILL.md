---
name: lc-ms-metabolomics-data-normalization
description: Use when an unnormalized LC-MS metabolomics matrix (compounds x runs)
  shows signal drift, batch effects, or occasional disproportionately high/low outlier
  intensities that deteriorate QC-based normalization, and you need batch-aware robust
  correction with QC-sample representativeness checking.
when_to_use_negative:
- When QC samples are known to be fully representative and you specifically require
  the prescribed QC-RLSC drift fit driven by QC samples only (QConly = TRUE) rather
  than robust sample-based normalization
- When input is not a raw unnormalized feature intensity matrix (e.g., already normalized
  data or a peak table lacking per-run order and batch annotations)
- When applying downstream multiple testing correction for differential abundance
  inference — the article states this is 'not in the scope of this work'
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_2269
tools:
- name: Metanorm R package
  role: Main implementation integrating rLOESS, rGAM, tGAM, QC-RLSC and QC-RSC normalization
    with parallelization and diagnostic visualization
  repo: github.com/UGent-LIMET/Metanorm
- name: metanorm
  role: Core function called with model = 'tGAM' (default), type, batch, QCcheck,
    QConly, weights, plotdir, and cv arguments
  repo: github.com/UGent-LIMET/Metanorm
- name: tGAM
  role: 'Recommended default robust model: mgcv thin-plate-spline GAM fitted by REML
    with gam.k/gam.frac settings, run batchwise, with QCcheck re-fitting on biological
    samples'
  repo: github.com/UGent-LIMET/Metanorm
- name: rLOESS
  role: Faster cross-validated robust LOESS alternative when tGAM's computational
    cost is too high
  repo: github.com/UGent-LIMET/Metanorm
- name: rGAM
  role: Faster robust additive-model alternative to tGAM
  repo: github.com/UGent-LIMET/Metanorm
- name: calcWeights
  role: Computes observation weights from residuals so high-residual (outlier) runs
    are downweighted in the GAM fit
  repo: github.com/UGent-LIMET/Metanorm
- name: plotPCA
  role: Generates PC score plots labeled by batch to verify batch effects diminished
    post-normalization
  repo: github.com/UGent-LIMET/Metanorm
- name: mgcv
  role: Fits the thin-plate-spline GAM underlying tGAM via REML
provenance:
  source_task_ids:
  - task_006
  - task_004
  source_papers:
  - doi: 10.1101/2025.09.30.679445
    title: Robust metabolomics data normalization across scales and experimental designs
  source_content_hash: sha256:dd0a0c0e4770f24e0e1153f33706d5913594a9d0ac7799e3da3063c9d0142ad3
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Metanorm R package
  - R
  - calcWeights
  - changepoint
  - cowplot
  - fANCOVA
  - ggplot2
  - metanorm
  - mgcv
  - pbapply
  - plotPCA
  - rGAM
  - rLOESS
  - tGAM
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42105263157894735
n_claims: 19
---

# lc-ms-metabolomics-data-normalization

## Summary

Robust normalization of unnormalized LC-MS metabolomics feature matrices using the Metanorm R package's tGAM (default), rGAM, or rLOESS models, which fit cross-validated thin-plate-spline/LOESS drift curves with outlier downweighting, batchwise fitting, and a QCcheck hypothesis test that triggers re-fitting on biological samples when QC samples are unrepresentative.

## When to use

Use when an unnormalized LC-MS metabolomics matrix (compounds x runs) shows signal drift, batch effects, or occasional disproportionately high/low outlier intensities that deteriorate QC-based normalization, and you need batch-aware robust correction with QC-sample representativeness checking.

## When NOT to use

- When QC samples are known to be fully representative and you specifically require the prescribed QC-RLSC drift fit driven by QC samples only (QConly = TRUE) rather than robust sample-based normalization
- When input is not a raw unnormalized feature intensity matrix (e.g., already normalized data or a peak table lacking per-run order and batch annotations)
- When applying downstream multiple testing correction for differential abundance inference — the article states this is 'not in the scope of this work'

## Inputs

- rawdata: numerical matrix of unnormalized feature intensities (rows = compounds, columns = sample runs), e.g., the FAME saliva untargeted cohort 618 runs x 8913 features
- batch: vector giving the batch membership of each sample run
- metanorm.qc: type vector labeling each run as 'QC' or another (biological) sample type
- example.RData bundled dataset from metanorm's extdata directory (rawdata, batch, metanorm.qc)

## Outputs

- normdat: normalized data matrix with the same dimensions as rawdata (saveable as normdat.rds)
- Pre- vs post-normalization intensity-vs-order diagnostic plots per compound written to plotdir
- PCA score plots (plotPCA) of raw and normalized data labeled by batch
- Per-compound fitted drift curves (tGAM via mgcv REML, rLOESS, rGAM, or QC-RLSC) with residual-based observation weights

## How to apply

Load the metanorm package in R (>= 4.4.0) and prepare a rawdata matrix (rows = compounds, columns = samples), a batch vector, and a type vector (e.g., metanorm.qc with 'QC' vs 'sample' labels) matching the bundled example.RData structure. Call metanorm() with model = "tGAM" (the recommended default; rGAM or rLOESS are faster alternatives), passing type, batch, QCcheck = TRUE, and observation weights from calcWeights(residuals) so high-residual outlier runs are downweighted in the mgcv REML GAM fit. The batchwise option fits a separate additive model per batch subset, allowing batch-specific residual variances; the QCcheck adds a 'type' covariate comparing QC vs biological samples and, if a significant difference is detected at the user-specified cut-off, re-fits and normalizes using biological samples only (set QConly = FALSE for this automatic fallback). Provide plotdir so pre- vs post-normalization intensity-vs-order diagnostic plots are written per compound, under either GCV or LOOCV span selection via the cv argument. Verify success by inspecting a plotPCA() score plot labeled by batch (batch effects and drift should diminish) and a decent number of per-compound diagnostic plots, then save the returned matrix as normdat.rds.

## Related tools

- **Metanorm R package** (Main implementation integrating rLOESS, rGAM, tGAM, QC-RLSC and QC-RSC normalization with parallelization and diagnostic visualization) — github.com/UGent-LIMET/Metanorm
- **metanorm** (Core function called with model = 'tGAM' (default), type, batch, QCcheck, QConly, weights, plotdir, and cv arguments) — github.com/UGent-LIMET/Metanorm
- **tGAM** (Recommended default robust model: mgcv thin-plate-spline GAM fitted by REML with gam.k/gam.frac settings, run batchwise, with QCcheck re-fitting on biological samples) — github.com/UGent-LIMET/Metanorm
- **rLOESS** (Faster cross-validated robust LOESS alternative when tGAM's computational cost is too high) — github.com/UGent-LIMET/Metanorm
- **rGAM** (Faster robust additive-model alternative to tGAM) — github.com/UGent-LIMET/Metanorm
- **calcWeights** (Computes observation weights from residuals so high-residual (outlier) runs are downweighted in the GAM fit) — github.com/UGent-LIMET/Metanorm
- **plotPCA** (Generates PC score plots labeled by batch to verify batch effects diminished post-normalization) — github.com/UGent-LIMET/Metanorm
- **mgcv** (Fits the thin-plate-spline GAM underlying tGAM via REML)

## Examples

```
normdat <- metanorm(rawdata, model = "tGAM", type = metanorm.qc, QCcheck = TRUE, batch = batch, plotdir = "~/Documents/metanormExample/")
```

## Evaluation signals

- plotPCA() score plots labeled by batch show diminished batch effects and drift after normalization
- Per-compound pre- vs post-normalization intensity-vs-order plots (from plotdir) show drift removed in both QC and biological samples; check multiple compounds, e.g., plot number 4 under QC-RLSC
- In differential-abundance benchmarking, DAM overlap among robust methods (rLOESS/rGAM/tGAM) falls in the reported 79-87% band and QC-RLSC overlap with them falls in 55-59%
- GCV vs LOOCV span selection yields highly comparable DAM counts and p-value orderings within each method
- normdat matrix retains the same dimensions (compounds x runs) as the input rawdata matrix

## Limitations

- tGAM has a higher computational cost than rLOESS and rGAM, though normalization still completes within minutes for thousands of metabolites and hundreds of samples
- QCcheck fallback matters because differences between QC and biological samples may invalidate QC-only strategies such as filtering on QC RSD
- Outlier treatment is feature-level; multivariate outlier detection across features may improve upon it
- Automated outlier detection is non-trivial, and manual outlier identification is subjective and labor-intensive for large feature counts

## Evidence

- [abstract] Robust methods abstract claim: "We introduce rLOESS, rGAM and tGAM, three robust normalization methods that improve resistance to outliers by downweighting or accommodating them."
- [abstract] tGAM default recommendation: "Overall, tGAM-based normalization suggested the best performance across scenarios and is proposed as default choice."
- [readme] Package scope from README: "Metanorm supports robust metabolomics data normalization across scales and experimental designs. The R package implements three (new) robust normalization methods (tGAM, rGAM and rLOESS)"
- [methods] QCcheck usage from methods: "QCcheck = TRUE,      # check whether QCs are representative"
- [discussion] tGAM cost caveat: "A practical limitation of tGAM is its higher computational cost relative to rLOESS and rGAM"
- [discussion] QC RSD filtering caution: "differences between QC and biological samples may invalidate filtering on e.g., relative standard deviation (RSD) observed in QC samples"
