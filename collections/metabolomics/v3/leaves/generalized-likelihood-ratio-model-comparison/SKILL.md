---
name: generalized-likelihood-ratio-model-comparison
description: Use when normalizing metabolomics run-order data with pooled QC injections
  and you must decide, per compound (and batch), whether QC-based drift fits are statistically
  consistent with biological-sample fits before trusting QC-driven normalization.
when_to_use_negative:
- When using QC-RLSC or QC-RSC normalization, which by design use QC samples only
  (QConly = TRUE) and have no type-covariate likelihood-ratio comparison
- When QC and biological samples are known a priori to differ systematically — the
  article notes unexpected differences 'should receive due analyst scrutiny' rather
  than being resolved automatically
- When the data matrix lacks pooled QC runs or sample-type labels, since the type
  covariate and GLRT cannot be constructed
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0203
tools:
- name: Metanorm
  role: R package implementing tGAM/rGAM normalization with the QCcheck option that
    adds the 'type' covariate and runs the likelihood-ratio-based refit decision
  repo: https://github.com/UGent-LIMET/Metanorm
- name: mgcv
  role: R package used to fit the Gaussian GAM (rGAM, REML, thin plate regression
    splines, Tukey biweight weights) and the scaled-t tGAM extended with the type
    covariate for the GLRT comparison
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1021/acs.analchem.5c06841
    title: Robust Metabolomics Data Normalization across Scales and Experimental Designs
  source_content_hash: sha256:4a507275946282f1a99c89bcc2cc3a0688cb9a68bd65ecd76b50e5d00d1a33d4
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
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# generalized-likelihood-ratio-model-comparison

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A hypothesis-testing device inside Metanorm's GAM-based normalization (rGAM/tGAM): a 'type' covariate gives QC and biological-sample runs separate intercepts and smooth functions, and a generalized likelihood ratio test at a user-specified p-value cutoff (QCcheckp) decides whether pooled QCs are representative of biological samples, triggering a biological-samples-only refit when they are not.

## When to use

Use when normalizing metabolomics run-order data with pooled QC injections and you must decide, per compound (and batch), whether QC-based drift fits are statistically consistent with biological-sample fits before trusting QC-driven normalization.

## When NOT to use

- When using QC-RLSC or QC-RSC normalization, which by design use QC samples only (QConly = TRUE) and have no type-covariate likelihood-ratio comparison
- When QC and biological samples are known a priori to differ systematically — the article notes unexpected differences 'should receive due analyst scrutiny' rather than being resolved automatically
- When the data matrix lacks pooled QC runs or sample-type labels, since the type covariate and GLRT cannot be constructed

## Inputs

- Numerical matrix of unnormalized metabolite intensities (rows = compounds, columns = sample runs), e.g. the metanorm 'rawdata' matrix
- Vector of sample types per run marking 'QC' vs other (biological) samples, e.g. metanorm.qc
- Vector assigning each sample run to a batch (for batchwise model specification)
- User-specified QCcheckp p-value cutoff for the generalized likelihood ratio test

## Outputs

- Per-compound (and per-batch) QCcheck p-value from the generalized likelihood ratio test
- Refit decision: whether normalization proceeds with joint QC+sample fits or a biological-samples-only refit
- Normalized intensity matrix from the selected model (tGAM or rGAM), plus optional pre/post normalization diagnostic plots

## How to apply

Extend the GAM of metabolite intensity on injection order with a 'type' covariate so QC and biological-sample runs receive separate intercepts and smooth functions (per the paper's batchwise model specification). Compare the QC- and sample-based intercepts/smooths with a generalized likelihood ratio test; when the p-value falls below the user-specified cutoff (QCcheckp), the QC samples are judged unrepresentative and the model is automatically refit and normalization uses biological samples only. In the Metanorm R package this is exposed via metanorm(model = 'tGAM', QCcheck = TRUE, ...), and the README recommends 'to have metanorm check for discrepancies between QC and biological samples' rather than defaulting to joint QC+sample normalization. Fit with mgcv (restricted maximum likelihood, thin plate regression splines); for rGAM use Tukey biweight observation weights from residuals scaled by MAD*1.4826 with c = 4.6821, and for tGAM use a scaled-t error distribution.

## Related tools

- **Metanorm** (R package implementing tGAM/rGAM normalization with the QCcheck option that adds the 'type' covariate and runs the likelihood-ratio-based refit decision) — https://github.com/UGent-LIMET/Metanorm
- **mgcv** (R package used to fit the Gaussian GAM (rGAM, REML, thin plate regression splines, Tukey biweight weights) and the scaled-t tGAM extended with the type covariate for the GLRT comparison)

## Examples

```
normdat <- metanorm(rawdata[1:5,], model = "tGAM", type = metanorm.qc, QCcheck = TRUE, batch = batch, plotdir = "~/metanormExample/")
```

## Evaluation signals

- On synthetic series with representative QCs (intensity = 20 + sin(x) + Gaussian noise, sd 0.3; 2 QC per 10 biological samples over 1000 samples), the QCcheck p-value stays above the cutoff and no biological-samples-only refit triggers
- In the unrepresentative-QC variant (QC intercepts/smooths deviating from sample smooths), the GLRT p-value falls below QCcheckp and the biological-samples-only refit fires
- Median squared prediction error (MSPE) at the QC indices, against the known sine drift, decreases after the refit, indicating recovery of the simulated drift
- Decisions and MSPE values are written to qcgam_qccheck_report.csv with per-series QCcheck p-values and refit flags
- After metanorm(QCcheck = TRUE), pre/post-normalization PCA score plots and per-compound intensity-vs-order plots should show drift removal without QC-induced distortion of biological samples

## Limitations

- The QCcheck is a feature-level (per-compound) test; multivariate outlier detection and cross-metabolite information are not leveraged
- Significant QC-vs-sample differences flag unrepresentative QCs but the article stresses such differences 'should receive due analyst scrutiny' rather than being silently corrected
- Significant differences between QC and biological samples can also invalidate downstream filtering on QC-sample RSD
- The test is batchwise; behavior depends on sufficient QC and biological runs within each batch

## Evidence

- [other] type covariate + GLRT mechanism: [evidence span withheld — anchor status: not_found]
- [other] GLRT significance triggers refit: [evidence span withheld — anchor status: not_found]
- [readme] README recommendation to use QCcheck: [evidence span withheld — anchor status: external_doc]
- [readme] README QCcheck argument: [evidence span withheld — anchor status: external_doc]
