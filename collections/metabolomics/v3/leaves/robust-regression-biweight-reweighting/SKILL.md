---
name: robust-regression-biweight-reweighting
description: Use when normalizing metabolomics intensity data exhibiting signal drift
  over injection order and occasional outlying compound intensities (outliers) that
  would deteriorate QC-based normalization and downstream differential abundance testing.
when_to_use_negative:
- When the goal is multivariate outlier detection across compounds — the article notes
  multivariate outlier detection may improve the current feature-level treatment of
  outliers, but it is not implemented here
- When the practitioner intends to remove outliers via a separate automated outlier-detection
  pipeline before normalizing — the article flags this as nontrivial and not provided
  as an alternative
- When evaluating normalization by reduction of QC-sample relative standard deviation
  (RSD) or increased numbers of differentially abundant metabolites — the article
  explicitly flags these as benchmarking pitfalls rather than appropriate goals
edam_operation: http://edamontology.org/operation_2930
edam_topics:
- http://edamontology.org/topic_3292
- http://edamontology.org/topic_0092
tools:
- name: Metanorm
  role: R package implementing rGAM and tGAM robust normalization with batch support,
    QCcheck discrepancy checking, parallel processing, and diagnostic plot generation
  repo: https://github.com/UGent-LIMET/Metanorm
- name: mgcv
  role: R package used to fit the GAMs (restricted maximum likelihood, thin plate
    regression splines) underlying rGAM and to extend the model with the 'type' covariate
    for QC vs biological samples
- name: R
  role: Runtime environment for Metanorm (requires R >= 4.4.0) and the mgcv-based
    fitting workflow
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1021/acs.analchem.5c06841
    title: Robust Metabolomics Data Normalization across Scales and Experimental Designs
  source_content_hash: sha256:3f2792670880d2419a819413b5ca68bfd31e3c1f891872c7b5e1bda62975ff41
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
  - R
  - mgcv
  edam_operation: http://edamontology.org/operation_2930
  edam_topics:
  - http://edamontology.org/topic_3292
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# robust-regression-biweight-reweighting

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Outlier-robust metabolomics signal-drift normalization by regression with reweighting: fit a GAM (rGAM) or scaled-t GAM (tGAM) of compound intensity on injection order via mgcv, then refit with Tukey biweight observation weights so occasional disproportionately high/low intensities cannot distort the fitted drift.

## When to use

Use when normalizing metabolomics intensity data exhibiting signal drift over injection order and occasional outlying compound intensities (outliers) that would deteriorate QC-based normalization and downstream differential abundance testing.

## When NOT to use

- When the goal is multivariate outlier detection across compounds — the article notes multivariate outlier detection may improve the current feature-level treatment of outliers, but it is not implemented here
- When the practitioner intends to remove outliers via a separate automated outlier-detection pipeline before normalizing — the article flags this as nontrivial and not provided as an alternative
- When evaluating normalization by reduction of QC-sample relative standard deviation (RSD) or increased numbers of differentially abundant metabolites — the article explicitly flags these as benchmarking pitfalls rather than appropriate goals

## Inputs

- Numerical intensity matrix (rows = compounds, columns = sample runs), e.g. the 'rawdata' object in the Metanorm example dataset
- Sample-type vector indicating which runs are QC vs biological samples (e.g. metanorm.qc with values 'QC'/'sample')
- Batch vector assigning each sample run to a batch (normalized batchwise)
- Injection-order index for each run (the GAM covariate)

## Outputs

- Normalized data matrix (same dimensions as input, e.g. normdat)
- Per-compound fitted drift function with robust Tukey biweight observation weights (rGAM) or scaled-t error fit (tGAM)
- Optional QCcheck refit decision per compound (biological-samples-only vs joint QC+sample fit) based on the generalized likelihood ratio test p-value vs QCcheckp
- Diagnostic plots: per-compound pre- vs post-normalization intensity-vs-order plots and PCA score plots colored by batch

## How to apply

For each compound (feature-level) and batch, fit an initial Gaussian GAM of intensity on injection order with mgcv using restricted maximum likelihood and thin plate regression splines. Compute residuals, scale them by MAD*1.4826, and derive Tukey biweight observation weights using the constant c = 4.6821, then refit the GAM with these weights (rGAM); alternatively use tGAM, which fits a scaled-t error distribution instead of two-stage reweighting. Extend either model with a 'type' covariate giving QC and biological samples separate intercepts and smooth functions, and optionally apply QCcheck: assess QC- vs sample-based fits by generalized likelihood ratio test at a user-specified cutoff (QCcheckp), refitting on biological samples only when p < cutoff. The rationale is that a single outlying injection disproportionately moves a least-squares-style fit, while robust weighting or a heavy-tailed error distribution recovers the underlying drift; the article reports tGAM as best-performing overall and proposes it as the default, with rGAM and rLOESS faster. Correct application is judged by improved replicate concordance, reduced drift and batch effects, and — in simulations with known ground truth (intensity = 20 + sin(x) + Gaussian noise, sd 0.3) — by median squared prediction error (MSPE) at the QC indices.

## Related tools

- **Metanorm** (R package implementing rGAM and tGAM robust normalization with batch support, QCcheck discrepancy checking, parallel processing, and diagnostic plot generation) — https://github.com/UGent-LIMET/Metanorm
- **mgcv** (R package used to fit the GAMs (restricted maximum likelihood, thin plate regression splines) underlying rGAM and to extend the model with the 'type' covariate for QC vs biological samples)
- **R** (Runtime environment for Metanorm (requires R >= 4.4.0) and the mgcv-based fitting workflow)

## Examples

```
normdat <- metanorm(rawdata[1:5,], model = "tGAM", type = metanorm.qc, QCcheck = TRUE, batch = batch, plotdir = "~/metanormExample/")
```

## Evaluation signals

- Refitted rGAM weights must be computed from residuals scaled by MAD*1.4826 with the Tukey biweight constant c = 4.6821 (not other constants)
- In synthetic series with known drift (20 + sin(x) + noise), the report's median squared prediction error (MSPE) at QC indices should reflect recovery of the known sine drift
- With QCcheck enabled, the biological-samples-only refit should trigger only for unrepresentative-QC series (p < QCcheckp cutoff) and not for representative-QC series
- Post-normalization PCA score plots labeled by batch should show diminished batch effects relative to pre-normalization plots
- tGAM-based normalization should exhibit the strongest performance across experimental data sets, with simulated-data performance comparable between rLOESS, rGAM, and tGAM

## Limitations

- Outlier handling is at the feature (compound) level; multivariate outlier detection across metabolites is not implemented
- Automated outlier detection and removal prior to normalization is a nontrivial alternative that the package does not provide
- Unexpected differences between QC and biological samples require due analyst scrutiny — QCcheck automates refitting but does not replace inspection of per-compound pre/post-normalization plots
- Simulation design does not fully mimic experimental data (no resampling strategies or cross-metabolite correlations), so benchmark results may not transfer to all real datasets

## Evidence

- [readme] Metanorm supports robust metabolomics data normalization across scales and experimental designs.: [evidence span withheld — anchor status: external_doc]
- [readme] We further recommend using both QC as well as biological samples for normalization, and to have metanorm check for discrepancies between QC and biological samples.: [evidence span withheld — anchor status: external_doc]
