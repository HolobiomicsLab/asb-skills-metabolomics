---
name: scaled-t-likelihood-outlier-accommodation
description: Use when metabolomics intensities show run-order signal drift plus occasional
  extreme high/low outliers that would corrupt QC-sample-based normalization, or when
  pooled QC samples may be unrepresentative of biological samples.
when_to_use_negative:
- Runtime-constrained high-throughput normalization where rGAM or rLOESS are explicitly
  noted as faster alternatives to the recommended-but-heavier tGAM
- Data with no injection-order structure or expected signal drift (e.g. already normalized
  or randomized-acquisition data), where fitting an order GAM adds no correction
- Analyses aiming to treat outliers multivariate or detect them in a dedicated preprocessing
  pipeline — the article notes the current treatment is feature-level and automated
  outlier detection prior to normalization is nontrivial and not provided
edam_topics:
- http://edamontology.org/topic_3174
tools:
- name: Metanorm
  role: R package providing the tGAM normalization implementation with QCcheck, batchwise
    fitting, parallelization, and diagnostic visualization
  repo: https://github.com/UGent-LIMET/Metanorm
- name: mgcv
  role: R package used to fit the generalized additive models (GAMs) of intensity
    on injection order, including the scaled-t error distribution for tGAM
- name: R
  role: Runtime environment; Metanorm requires R >= 4.4.0 and is installed via pak
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1021/acs.analchem.5c06841
    title: Robust Metabolomics Data Normalization across Scales and Experimental Designs
  source_content_hash: sha256:f8be363551e93de7a40383139acb8f86b67dc51a0a79ed3880744e304d28f099
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# scaled-t-likelihood-outlier-accommodation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

The tGAM normalization approach from the Metanorm R package: per-compound generalized additive models of metabolite intensity on injection order, fitted with a scaled-t error distribution via mgcv so occasional disproportionately high or low intensity outliers are downweighted by the heavy-tailed likelihood rather than distorting the drift estimate.

## When to use

Use when metabolomics intensities show run-order signal drift plus occasional extreme high/low outliers that would corrupt QC-sample-based normalization, or when pooled QC samples may be unrepresentative of biological samples.

## When NOT to use

- Runtime-constrained high-throughput normalization where rGAM or rLOESS are explicitly noted as faster alternatives to the recommended-but-heavier tGAM
- Data with no injection-order structure or expected signal drift (e.g. already normalized or randomized-acquisition data), where fitting an order GAM adds no correction
- Analyses aiming to treat outliers multivariate or detect them in a dedicated preprocessing pipeline — the article notes the current treatment is feature-level and automated outlier detection prior to normalization is nontrivial and not provided

## Inputs

- Numerical raw intensity matrix (rows = compounds, columns = samples), e.g. the 'rawdata' object in the Metanorm example.RData
- Vector of sample types per run ('QC' vs other), e.g. 'metanorm.qc'
- Batch vector assigning each sample run to a batch (for batchwise model fitting)
- Injection/run order of samples (implicit ordering covariate for the GAM)

## Outputs

- Normalized data matrix of the same dimensions as the input ('normdat')
- Fitted tGAM drift models per compound, including the QC-vs-sample 'type' covariate extension
- QCcheck GLR-test p-values and automatic refit decisions (biological-samples-only normalization when QC samples are judged unrepresentative)
- Diagnostic pre- vs post-normalization intensity-vs-order plots per compound and PCA score plots labeled by batch

## How to apply

For each compound (batchwise), fit a Gaussian-free generalized additive model of intensity on injection order with a scaled-t error family using mgcv, letting the heavy-tailed likelihood accommodate outliers instead of the exact outlier removal needed by rLOESS/QC-RLSC. Optionally extend the model with a 'type' covariate giving QC and biological samples separate intercepts and smooth functions, then run the QCcheck: compare the QC- and sample-based fits with a generalized likelihood ratio test at a user-specified p-value cutoff (QCcheckp); a significant result indicates unrepresentative QCs and triggers a refit using biological samples only. Normalize by subtracting the fitted drift and dividing out its magnitude, processing compounds in parallel. The article reports tGAM-based normalization as the best performer across scenarios and proposes it as the default choice, so prefer it over rGAM/rLOESS unless runtime matters.

## Related tools

- **Metanorm** (R package providing the tGAM normalization implementation with QCcheck, batchwise fitting, parallelization, and diagnostic visualization) — https://github.com/UGent-LIMET/Metanorm
- **mgcv** (R package used to fit the generalized additive models (GAMs) of intensity on injection order, including the scaled-t error distribution for tGAM)
- **R** (Runtime environment; Metanorm requires R >= 4.4.0 and is installed via pak)

## Examples

```
normdat <- metanorm(rawdata[1:5,], model = "tGAM", type = metanorm.qc, QCcheck = TRUE, batch = batch, plotdir = "~/Documents/metanormExample/")
```

## Evaluation signals

- In in silico series with known sine drift (intensity = 20 + sin(x) + noise, sd 0.3), median squared prediction error (MSPE) at QC indices should reflect recovery of the known simulated drift
- QCcheck refit should trigger only for series where QC fits deviate significantly from biological-sample fits (p < QCcheckp), not for representative-QC series
- Replicate concordance should improve and drift/batch effects should shrink relative to QC-RLSC/QC-RSC and other existing strategies
- PCA score plots labeled by batch should show diminished batch effects after normalization (compare plotPCA on rawdata vs normdat)
- Pre- vs post-normalization intensity-vs-order plots for individual compounds should show the fitted drift removed in biological (non-QC) samples — README flags plot 4 of the QC-RLSC example as a case where drift persists in biological samples

## Limitations

- Outlier treatment is feature-level; multivariate outlier detection is not implemented in the current approach
- tGAM is slower than rGAM and rLOESS; README notes tGAM is recommended for robustness but the others are faster
- Unexpected differences between QC samples and biological samples still require due analyst scrutiny of diagnostic plots — normalization does not replace visual inspection
- Using QC-sample-derived statistics (e.g. RSD filtering) downstream can be invalidated when QC and biological samples differ
- Benchmarking pitfalls apply when evaluating the method: reduced QC RSD or increased numbers of differentially abundant metabolites are not reliable proxies for good normalization

## Evidence

- [readme] Metanorm implements tGAM alongside QC-RLSC/QC-RSC: [evidence span withheld — anchor status: external_doc]
- [readme] Recommendation to check QC vs sample discrepancies: [evidence span withheld — anchor status: external_doc]
