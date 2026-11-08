---
name: batchwise-metabolomics-normalization
description: Use when LC-MS metabolomics runs are split into multiple acquisition
  batches and signal drift plus batch effects must be corrected together, especially
  when QC samples may not represent biological samples.
when_to_use_negative:
- When only a single acquisition batch exists and batch subsets would be too small
  to support a separate additive model per batch
- When QC samples are demonstrably representative and simple QC-based normalization
  (e.g. QC-RLSC with QConly = TRUE) suffices without batch subsetting
- When the goal is generic feature filtering or downstream differential-abundance
  testing rather than drift/batch correction of raw intensities
edam_operation: http://edamontology.org/operation_3578
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: metanorm
  role: Core R function performing the batchwise robust normalization; accepts model
    = "tGAM", type, batch, QCcheck and plotdir arguments and returns the normalized
    matrix
  repo: github.com/UGent-LIMET/Metanorm
- name: Metanorm R package
  role: Package distributing the metanorm(), plotPCA() and calcWeights() implementations
    plus the example.RData dataset; installed from GitHub
  repo: github.com/UGent-LIMET/Metanorm
- name: mgcv
  role: Fits the thin-plate-spline GAM used by the tGAM model, with REML estimation
    and gam.k / gam.frac settings, allowing the 'type' covariate for the QCcheck hypothesis
    test
- name: calcWeights
  role: Computes observation weights from model residuals so high-residual (outlier)
    sample runs are downweighted in the GAM fit
- name: plotPCA
  role: Generates PC score plots before and after normalization labeled by batch to
    verify that batch effects have diminished
  repo: github.com/UGent-LIMET/Metanorm
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1101/2025.09.30.679445
    title: Robust metabolomics data normalization across scales and experimental designs
  source_content_hash: sha256:1e1a389f987e346c6202379b7478bf5c4036e18a73319f88d70ecc985b6117d1
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
  - rLOESS
  edam_operation: http://edamontology.org/operation_3578
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6206896551724138
n_claims: 29
---

# batchwise-metabolomics-normalization

## Summary

A batch-aware metabolomics normalization procedure that fits a separate robust additive model (tGAM, rGAM or rLOESS) per batch subset and optionally checks whether QC samples are representative of biological samples before choosing which sample type drives the correction.

## When to use

Use when LC-MS metabolomics runs are split into multiple acquisition batches and signal drift plus batch effects must be corrected together, especially when QC samples may not represent biological samples.

## When NOT to use

- When only a single acquisition batch exists and batch subsets would be too small to support a separate additive model per batch
- When QC samples are demonstrably representative and simple QC-based normalization (e.g. QC-RLSC with QConly = TRUE) suffices without batch subsetting
- When the goal is generic feature filtering or downstream differential-abundance testing rather than drift/batch correction of raw intensities

## Inputs

- rawdata: numerical matrix of unnormalized metabolite intensities (rows = compounds, columns = sample runs), e.g. from example.RData
- batch: character/factor vector labeling the batch each sample run belongs to (e.g. "batch1")
- type vector (metanorm.qc): vector marking each run as "QC" or another (biological) sample type

## Outputs

- normdat: normalized numerical matrix of the same shape as rawdata (saveable as normdat.rds)
- Pre- vs post-normalization intensity-vs-order diagnostic plots per compound written to plotdir
- PC score plots before and after normalization (plotPCA), labeled by batch

## How to apply

Load the metanorm R package (requires R >= 4.4.0, install via pak::pak("UGent-LIMET/Metanorm")) and prepare a rawdata intensity matrix (rows = compounds, columns = sample runs), a batch vector labeling each run, and a type vector distinguishing 'QC' from other samples. Call metanorm() with model = "tGAM" (default; alternatives are "rGAM", "rLOESS" or "QC-RLSC"), passing type = metanorm.qc and batch = batch so the mgcv thin-plate-spline GAM — fitted by REML with gam.k and gam.frac settings and observation weights from calcWeights(residuals) to downweight high-residual outlier runs — is applied batchwise, fitting a separate additive model per batch subset with batch-specific residual variances rather than a single global fit. Set QCcheck = TRUE so the function runs the hypothesis test for whether QC samples are representative of the biological samples by adding a 'type' covariate comparing QC vs non-QC samples; if a significant difference is detected at a user-specified cut-off, the model is re-fitted and normalization uses biological samples only (with QConly = FALSE). Supply plotdir so pre- vs post-normalization intensity-vs-order diagnostic plots are written per compound, and save the returned normalized matrix as an RDS file. Verify by inspecting the diagnostic plots and a plotPCA() score plot labeled by batch to confirm that drift and batch effects have diminished; the README shows that a naive QC-RLSC run with QConly = TRUE can look fine in the PC plot yet leave drift in biological samples, so individual compound plots are essential.

## Related tools

- **metanorm** (Core R function performing the batchwise robust normalization; accepts model = "tGAM", type, batch, QCcheck and plotdir arguments and returns the normalized matrix) — github.com/UGent-LIMET/Metanorm
- **Metanorm R package** (Package distributing the metanorm(), plotPCA() and calcWeights() implementations plus the example.RData dataset; installed from GitHub) — github.com/UGent-LIMET/Metanorm
- **mgcv** (Fits the thin-plate-spline GAM used by the tGAM model, with REML estimation and gam.k / gam.frac settings, allowing the 'type' covariate for the QCcheck hypothesis test)
- **calcWeights** (Computes observation weights from model residuals so high-residual (outlier) sample runs are downweighted in the GAM fit)
- **plotPCA** (Generates PC score plots before and after normalization labeled by batch to verify that batch effects have diminished) — github.com/UGent-LIMET/Metanorm

## Examples

```
normdat <- metanorm(rawdata[1:5,], model = "tGAM", type = metanorm.qc, QCcheck = TRUE, batch = batch, plotdir = "~/Documents/metanormExample/")
```

## Evaluation signals

- Inspect a decent number of individual compound pre- vs post-normalization intensity-vs-order plots in plotdir: residual drift in biological (non-QC) samples should be visibly flattened; the README warns that a batch-effect-free PC plot can still hide drift in biological samples under QC-RLSC with QConly = TRUE.
- Compare plotPCA(normdat, type = batch) against the pre-normalization PC score plot: batch clustering should be reduced after normalization.
- Check the QCcheck hypothesis test result: when a significant QC-vs-biological difference is detected at the specified cut-off, the model is re-fitted and normalization should switch to biological samples rather than QCs.
- Confirm the returned normdat matrix has the same dimensions and row/column ordering as the input rawdata matrix.
- Verify the fit downweights outlier runs: sample runs with high residuals should receive low weights from calcWeights so they do not dominate the GAM/LOESS fit.

## Limitations

- tGAM has higher computational cost than rGAM and rLOESS, though normalization of thousands of metabolites and hundreds of samples completes within minutes.
- QC-based filtering on relative standard deviation (RSD) may be invalidated when differences exist between QC and biological samples, which is why QCcheck re-fitting is offered.
- Manually inspecting pre/post plots is essential; batch effects can appear absent in PC score plots while drift persists in biological samples (demonstrated with QC-RLSC, QConly = TRUE, compound #4).

## Evidence

- [results] batchwise option fits a separate additive model per batch subset: [evidence span withheld — anchor status: not_found]
- [results] QCcheck triggers re-fitting with biological samples: [evidence span withheld — anchor status: not_found]
- [abstract] tGAM recommended as default normalization model: "Overall, tGAM-based normalization suggested the best performance across scenarios and is proposed as default choice."
- [readme] package installation command: "pak::pak("UGent-LIMET/Metanorm")"
- [readme] recommended metanorm call with batch and QCcheck: [evidence span withheld — anchor status: not_found]
- [readme] diagnostic plot inspection caveat: "It is highly recommended to look at a decent number of these plots to judge normalization performance."
