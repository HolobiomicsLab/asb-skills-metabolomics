---
name: qc-representativeness-testing
description: Use when pooled QC injections might not represent biological samples
  for a compound (e.g., suspected QC outliers or QC/sample intercept or drift differences)
  and you need a data-driven per-compound decision to normalize with QCs jointly or
  exclude them.
when_to_use_negative:
- When there are no pooled QC sample runs in the data (the 'type' covariate and QC-vs-sample
  comparison are undefined)
- When using QC-RLSC normalization, which as prescribed uses QC samples only (QConly
  = TRUE), so no joint/sample-selective decision applies
- When you only want QC-RSD-based feature filtering — the paper warns that differences
  between QC and biological samples may invalidate filtering on RSD observed in QC
  samples
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_2269
tools:
- name: Metanorm
  role: R package implementing tGAM/rGAM normalization with the QCcheck option (QCcheck
    = TRUE) that tests QC representativeness and refits on biological samples when
    the GLR-test p-value falls below the cutoff
  repo: https://github.com/UGent-LIMET/Metanorm
- name: mgcv
  role: R package used to fit the GAM models (thin plate regression splines, REML,
    scaled-t family) extended with the 'type' covariate for QC vs. sample intercepts/smooths
- name: R
  role: Runtime environment (R >= 4.4.0) for installing and running Metanorm
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1021/acs.analchem.5c06841
    title: Robust Metabolomics Data Normalization across Scales and Experimental Designs
  source_content_hash: sha256:546f89894718124c1cc2aa174f7ad34dc6c6a848eccf05daf4b5bbc4375a7fce
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
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# qc-representativeness-testing

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Metanorm's QCcheck mechanism extends the tGAM/rGAM normalization models with a 'type' covariate giving QC and biological-sample runs separate intercepts and smooths, then uses a generalized likelihood ratio test (at a user-specified p-value cutoff) to decide whether pooled QCs are representative; if not, normalization is refit on biological samples only.

## When to use

Use when pooled QC injections might not represent biological samples for a compound (e.g., suspected QC outliers or QC/sample intercept or drift differences) and you need a data-driven per-compound decision to normalize with QCs jointly or exclude them.

## When NOT to use

- When there are no pooled QC sample runs in the data (the 'type' covariate and QC-vs-sample comparison are undefined)
- When using QC-RLSC normalization, which as prescribed uses QC samples only (QConly = TRUE), so no joint/sample-selective decision applies
- When you only want QC-RSD-based feature filtering — the paper warns that differences between QC and biological samples may invalidate filtering on RSD observed in QC samples

## Inputs

- rawdata: numerical matrix of unnormalized metabolite intensities (rows = compounds, columns = sample runs), e.g. log-transformed as in the Metanorm example.RData
- metanorm.qc / type: vector with sample type per run ('QC' vs other)
- batch: vector assigning each sample run to a batch (batchwise model fit)
- User-specified QCcheckp p-value cutoff for the generalized likelihood ratio test

## Outputs

- Normalized compound intensity matrix with unrepresentative-QC compounds normalized on biological samples only
- Per-compound QCcheck GLR-test p-values and refit decisions (e.g., recorded as in qcgam_qccheck_report.csv)
- Per-compound pre- vs. post-normalization intensity-vs-order diagnostic plots written to plotdir

## How to apply

Fit the rGAM (mgcv, restricted maximum likelihood, thin plate regression splines; refit with Tukey biweight weights from residuals scaled by MAD*1.4826 with c = 4.6821) or tGAM (mgcv, scaled-t error distribution) of intensity on injection order, then extend the model with a 'type' covariate so QC and biological samples get separate intercepts and smooth functions, per the paper's batchwise specification. Compare QC- and sample-based intercepts/smooths with a generalized likelihood ratio test; a significant result (p below the user-specified QCcheckp cutoff) indicates unrepresentative QCs. When significant, automatically refit the model and normalize using biological samples only; otherwise normalize jointly on QC and biological samples as the Metanorm README recommends. Run per compound (feature-level treatment) and batchwise, and inspect the generated pre- vs. post-normalization intensity-vs-order plots to confirm the decision.

## Related tools

- **Metanorm** (R package implementing tGAM/rGAM normalization with the QCcheck option (QCcheck = TRUE) that tests QC representativeness and refits on biological samples when the GLR-test p-value falls below the cutoff) — https://github.com/UGent-LIMET/Metanorm
- **mgcv** (R package used to fit the GAM models (thin plate regression splines, REML, scaled-t family) extended with the 'type' covariate for QC vs. sample intercepts/smooths)
- **R** (Runtime environment (R >= 4.4.0) for installing and running Metanorm)

## Examples

```
normdat <- metanorm(rawdata[1:5,], model = "tGAM", type = metanorm.qc, QCcheck = TRUE, batch = batch, plotdir = "~/Documents/metanormExample/")
```

## Evaluation signals

- QCcheck GLR-test p-values fall below the user-specified QCcheckp cutoff (triggering biological-samples-only refit) only for compounds/series where QC intercepts/smooths genuinely deviate from sample fits, and stay above the cutoff for representative-QC series
- In synthetic validation with known drift (intensity = 20 + sin(x) + N(0, 0.3) noise), median squared prediction error at QC indices reflects recovery of the known sine drift after the refit decision
- Normalized output contains no NaN/NA for compounds where refit succeeded, and refit compounds show QC runs adjusted on the biological-sample-only fit
- Per-compound pre- vs. post-normalization plots in plotdir show drift removed in biological (non-QC) samples, and post-normalization PCA score plots labeled by batch show diminished batch effects
- Decisions and MSPE values are recorded per series (e.g., in qcgam_qccheck_report.csv) and are consistent across reruns

## Limitations

- Treatment is feature-level (per compound); multivariate outlier detection across metabolites is not implemented
- Unexpected differences between QC samples and biological samples should still receive due analyst scrutiny — the mechanism automates the refit but does not replace inspection of diagnostic plots
- The automated QCcheck complements but does not replace manual review: the README states individual compound pre- vs. post-normalization plots allow finegrained assessment and it is highly recommended to look at a decent number of them

## Evidence

- [readme] README statement on recommended usage: [evidence span withheld — anchor status: external_doc]
- [readme] README code comment on QCcheck argument: [evidence span withheld — anchor status: external_doc]
