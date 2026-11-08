---
name: gam-drift-modeling
description: Use when metabolomics intensity data (runs × compounds, log(e)-transformed,
  with pooled QC injections, injection order, and batch labels) shows instrument drift
  or QC-vs-biological-sample discrepancies that must be modeled per compound batchwise
  before downstream testing.
when_to_use_negative:
- Data lack pooled QC samples, injection-order metadata, or batch labels — the drift
  model and QCcheck cannot be specified
- You need multivariate (cross-metabolite) outlier detection or batch-level treatment
  — Metanorm treats outliers at the feature level only
- You need multiple-testing-corrected downstream inference — this skill covers normalization
  only; multiple testing correction was explicitly out of scope in the source work
edam_operation: http://edamontology.org/operation_3436
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: Metanorm R package
  role: Provides the metanorm() function implementing tGAM/rGAM/rLOESS/QC-RLSC/QC-RSC
    drift models batchwise, with QCcheck, plotdir diagnostics, and plotPCA; supports
    parallelized high-throughput normalization
  repo: https://github.com/UGent-LIMET/Metanorm
- name: mgcv
  role: Fits the REML thin plate regression spline GAMs underlying tGAM and rGAM,
    and the 'type'-covariate QC-vs-sample model compared by generalized likelihood
    ratio tests
- name: R
  role: Runtime environment (>= 4.4.0 required by Metanorm) in which the batchwise
    GAM normalization is executed
- name: fANCOVA
  role: Supplies loess.as for the cross-validated robust LOESS (rLOESS) variant with
    Tukey biweight downweighting
provenance:
  source_task_ids:
  - task_002
  - task_007
  source_papers:
  - doi: 10.1021/acs.analchem.5c06841
    title: Robust Metabolomics Data Normalization across Scales and Experimental Designs
  source_content_hash: sha256:43457b8e0f9fa4c65c5c68d0abdd960cdb2110840cc661828985a337a04085f3
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: ''
  repo_url: https://github.com/UGent-LIMET/Metanorm
  tools:
  - Metanorm
  - Metanorm R package
  - R
  - SERFF
  - TIGER
  - fANCOVA
  - mgcv
  edam_operation: http://edamontology.org/operation_3436
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 18
---

# gam-drift-modeling

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Per-compound signal drift modeling for metabolomics normalization using generalized additive models: tGAM (REML-fitted thin plate splines with a scaled-t error distribution), rGAM (Gaussian GAM refit with Tukey biweight weights), and rLOESS, applied batchwise via the Metanorm R package with a QCcheck mechanism that reverts to biological samples when pooled QCs are unrepresentative.

## When to use

Use when metabolomics intensity data (runs × compounds, log(e)-transformed, with pooled QC injections, injection order, and batch labels) shows instrument drift or QC-vs-biological-sample discrepancies that must be modeled per compound batchwise before downstream testing.

## When NOT to use

- Data lack pooled QC samples, injection-order metadata, or batch labels — the drift model and QCcheck cannot be specified
- You need multivariate (cross-metabolite) outlier detection or batch-level treatment — Metanorm treats outliers at the feature level only
- You need multiple-testing-corrected downstream inference — this skill covers normalization only; multiple testing correction was explicitly out of scope in the source work

## Inputs

- Numerical intensity matrix (rows = compounds, columns = sample runs), log(e)-transformed
- Vector mapping each run to its batch
- Vector of sample types per run ('QC' vs other)
- Injection order (run index) used as the GAM covariate

## Outputs

- Normalized intensity matrix per compound
- Per-compound pre- vs post-normalization intensity-vs-order diagnostic plots (plotdir)
- PCA score plots of raw vs normalized data labeled by batch
- QCcheck GLRT p-values and refit decisions (biological-samples-only when QCs judged unrepresentative)

## How to apply

Log-transform intensities (base e), then fit a GAM of intensity on injection order separately within each batch. For tGAM, use mgcv with REML and thin plate regression splines under a scaled-t error distribution; for rGAM, fit an initial Gaussian GAM then refit with Tukey biweight observation weights computed from residuals scaled by MAD*1.4826 with constant c = 4.6821; rLOESS uses loess.as from fANCOVA with Tukey biweight downweighting. Activate QCcheck by adding a 'type' covariate so QC and biological-sample runs receive separate intercepts and smooth functions, then compare the fits with a generalized likelihood ratio test — if the p-value falls below the user-specified QCcheckp cutoff, the QCs are judged unrepresentative and the model is refit and normalization uses biological samples only. Normalize the whole matrix via metanorm() (rows = compounds, columns = runs) with the batch vector, and inspect the generated pre/post intensity-vs-order plots and PCA score plots (plotPCA) to judge batch-effect removal. On BioHEART, tGAM/rGAM/rLOESS agreed on ~92% of joint differential-abundance metabolites, whereas QConly methods like QC-RLSC and SERFF diverged, and unnormalized raw data inflated DAM counts.

## Related tools

- **Metanorm R package** (Provides the metanorm() function implementing tGAM/rGAM/rLOESS/QC-RLSC/QC-RSC drift models batchwise, with QCcheck, plotdir diagnostics, and plotPCA; supports parallelized high-throughput normalization) — https://github.com/UGent-LIMET/Metanorm
- **mgcv** (Fits the REML thin plate regression spline GAMs underlying tGAM and rGAM, and the 'type'-covariate QC-vs-sample model compared by generalized likelihood ratio tests)
- **R** (Runtime environment (>= 4.4.0 required by Metanorm) in which the batchwise GAM normalization is executed)
- **fANCOVA** (Supplies loess.as for the cross-validated robust LOESS (rLOESS) variant with Tukey biweight downweighting)

## Examples

```
normdat <- metanorm(rawdata[1:5,], model = "tGAM", type = metanorm.qc, QCcheck = TRUE, batch = batch, plotdir = "~/metanormExample/")
```

## Evaluation signals

- In in silico series with known sine drift (intensity = 20 + sin(x) + noise, sd 0.3), the median squared prediction error at QC indices should reflect recovery of the known drift
- QCcheck refit (biological-samples-only) should trigger only for unrepresentative-QC series (GLRT p < QCcheckp) and not for representative-QC series
- PCA score plots post-normalization should show diminished batch structure relative to raw data labeled by batch
- Per-compound pre/post intensity-vs-order plots should show drift removal without over-fitting to biological-sample outliers
- Differential-abundance overlap (two-sided Wilcoxon, p < 0.05) between tGAM, rGAM, and rLOESS should reach ~92% joint findings, with raw data showing the highest (inflated) DAM count

## Limitations

- Outlier treatment is feature-level only; multivariate outlier detection is not implemented
- Unexpected differences between QC and biological samples require analyst scrutiny of the diagnostic plots — the QCcheck does not fully automate interpretation
- rGAM and rLOESS are faster alternatives but tGAM is the most robust; performance on simulated data was comparable among the three, with tGAM strongest across experimental data sets
- Metanorm requires R >= 4.4.0

## Evidence

- [readme] tGAM is the robust default: [evidence span withheld — anchor status: external_doc]
