---
name: synthetic-drift-simulation-with-known-ground-truth
description: Use when a normalization method (rGAM, tGAM, rLOESS, QC-RLSC) must be
  benchmarked against a known reference drift, e.g. verifying recovery of simulated
  drift or testing QCcheck refit behavior on unrepresentative QCs.
when_to_use_negative:
- You intend to judge normalization quality by proxies the paper explicitly warns
  against, such as reduction of the relative standard deviation of QC samples or an
  increased number of differentially abundant metabolites
- Real experimental data with both QC and replicate samples are available for direct
  pre/post-normalization and PCA-based performance verification, and no ground-truth
  drift comparison is needed
- Automated outlier detection and removal prior to normalization is the intended alternative
  strategy (the article notes automated outlier detection is nontrivial and is a separate
  approach)
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3318
- http://edamontology.org/topic_0091
tools:
- name: R
  role: Language used to generate the synthetic drift series and run the simulation/evaluation
    loop
- name: mgcv
  role: Fits the rGAM (Gaussian GAM + Tukey biweight refit) and tGAM (scaled-t error
    distribution) models to the synthetic series so MSPE against the known drift can
    be computed
- name: Metanorm
  role: R package whose robust normalization methods (rGAM, tGAM, rLOESS) and QCcheck
    mechanism are the benchmarking targets being evaluated on the simulated series
  repo: https://github.com/UGent-LIMET/Metanorm
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1021/acs.analchem.5c06841
    title: Robust Metabolomics Data Normalization across Scales and Experimental Designs
  source_content_hash: sha256:20a5af9252ffadd41c0af69f176a7df0040d81c1a7eda9805ae167920437f5bd
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
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3318
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# synthetic-drift-simulation-with-known-ground-truth

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Generates in silico metabolomics intensity series with a known drift function (intensity = 20 + sin(x) + Gaussian noise, sd 0.3) so normalization performance can be verified against ground truth rather than proxies like QC-sample RSD reduction.

## When to use

Use when a normalization method (rGAM, tGAM, rLOESS, QC-RLSC) must be benchmarked against a known reference drift, e.g. verifying recovery of simulated drift or testing QCcheck refit behavior on unrepresentative QCs.

## When NOT to use

- You intend to judge normalization quality by proxies the paper explicitly warns against, such as reduction of the relative standard deviation of QC samples or an increased number of differentially abundant metabolites
- Real experimental data with both QC and replicate samples are available for direct pre/post-normalization and PCA-based performance verification, and no ground-truth drift comparison is needed
- Automated outlier detection and removal prior to normalization is the intended alternative strategy (the article notes automated outlier detection is nontrivial and is a separate approach)

## Inputs

- Simulation parameters: drift function (20 + sin(x)), Gaussian noise sd 0.3, 1000 samples with 2 QC injections per 10 biological samples (84 QC indices)
- Outlier perturbation schedule (0-4 outliers per series from the four perturbation types)
- Unrepresentative-QC variant specification (QC intercepts/smooths deviating from biological-sample smooths)
- Optionally, a real rawdata matrix (rows = compounds, columns = samples) with batch and QC/sample-type vectors for complementary experimental benchmarking

## Outputs

- Synthetic drift series with known sine drift at known QC indices
- qcgam_qccheck_report.csv containing per-series QCcheck p-value, refit decision, and median squared prediction error (MSPE) against the known drift at QC indices

## How to apply

Generate synthetic drift series as intensity = 20 + sin(x) + Gaussian noise (sd 0.3), with two QC injections per 10 biological samples over 1000 samples (84 QC indices). Inject 0-4 outliers per series drawn from the paper's four perturbation types: subtract 20-30%, subtract 20-100%, subtract 100%, or add 200% of intensity; and create a second variant in which QC intercepts/smooths deviate from biological-sample smooths to emulate unrepresentative QCs. Fit the target normalization models (e.g. rGAM via mgcv with thin plate regression splines and Tukey biweight weights with c = 4.6821, or tGAM with a scaled-t error distribution) to each synthetic series, optionally extended with a 'type' covariate so QC and biological samples get separate intercepts and smooths. Record per series the QCcheck p-value, refit decision, and median squared prediction error (MSPE) against the known sine drift at the QC indices, writing results to qcgam_qccheck_report.csv. The rationale, per the article, is that benchmarking pitfalls such as using reduced QC-sample RSD or more differentially abundant metabolites as quality proxies are avoided when the ground truth is known.

## Related tools

- **R** (Language used to generate the synthetic drift series and run the simulation/evaluation loop)
- **mgcv** (Fits the rGAM (Gaussian GAM + Tukey biweight refit) and tGAM (scaled-t error distribution) models to the synthetic series so MSPE against the known drift can be computed)
- **Metanorm** (R package whose robust normalization methods (rGAM, tGAM, rLOESS) and QCcheck mechanism are the benchmarking targets being evaluated on the simulated series) — https://github.com/UGent-LIMET/Metanorm

## Examples

```
intensity <- 20 + sin(x) + rnorm(n, sd = 0.3)  # known sine drift, two QC injections per 10 biological samples
```

## Evaluation signals

- qcgam_qccheck_report.csv shows the biological-samples-only refit triggers only for unrepresentative-QC series (p < user-specified QCcheckp cutoff) and not for representative-QC series
- MSPE at the 84 QC indices reflects recovery of the known simulated sine drift (approaching the noise floor of sd 0.3 Gaussian noise)
- Synthetic series match the paper's in silico design: 1000 samples, two QC injections per 10 biological samples, 84 QC indices
- Injected outliers reproduce the four perturbation types (subtract 20-30%, subtract 20-100%, subtract 100%, add 200% of intensity)
- Simulation outcome is interpretable because the ground truth drift function is exactly known

## Limitations

- The simulation design does not mimic experimental metabolomics data comprehensively (no resampling strategies or cross-metabolite correlations), as flagged by the authors as future work
- Outlier treatment in Metanorm is feature-level; the simulation therefore cannot assess multivariate outlier behavior
- Unexpected differences between QC samples and biological samples, even when detected, require analyst scrutiny rather than fully automated resolution

## Evidence

- [readme] Metanorm supports robust metabolomics data normalization across scales and experimental designs.: [evidence span withheld — anchor status: external_doc]
