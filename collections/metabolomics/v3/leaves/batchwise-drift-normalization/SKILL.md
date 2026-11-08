---
name: batchwise-drift-normalization
description: Use when unnormalized metabolomics intensity matrices (raw compounds
  × sample runs) show instrument drift or batch effects across multiple acquisition
  batches and must be normalized batchwise before differential abundance testing.
when_to_use_negative:
- When a downstream workflow needs a fair benchmark across very different normalization
  philosophies (e.g. multivariate methods such as SERFF or TIGER) — the article explicitly
  notes these were not benchmarked with Metanorm
- When the data contain unexpected differences between QC and biological samples —
  these require analyst scrutiny rather than blind normalization, per the article's
  recommendation
- When the goal is statistical inference on DAMs requiring controlled false discovery
  — multiple testing correction is essential but was out of scope for this work
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: Metanorm R package
  role: Implements the batchwise normalization call (metanorm() with batch and type
    arguments) plus diagnostic plotPCA() visualization
  repo: https://github.com/UGent-LIMET/Metanorm
- name: R
  role: Runtime environment (>= 4.4.0) in which the Metanorm package is installed
    and executed
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1021/acs.analchem.5c06841
    title: Robust Metabolomics Data Normalization across Scales and Experimental Designs
  source_content_hash: sha256:4bf075f7694fec8d5a6dc9def1ecd03df1957aa9c8820675bd9a81e9767ea9f0
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
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# batchwise-drift-normalization

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Applies Metanorm's drift-correcting metabolomics normalization (robust tGAM/rGAM/rLOESS or QC-sample-based QC-RLSC/QC-RSC) separately per batch, using a QC-type vector and batch labels so that within-batch signal drift is removed before downstream testing.

## When to use

Use when unnormalized metabolomics intensity matrices (raw compounds × sample runs) show instrument drift or batch effects across multiple acquisition batches and must be normalized batchwise before differential abundance testing.

## When NOT to use

- When a downstream workflow needs a fair benchmark across very different normalization philosophies (e.g. multivariate methods such as SERFF or TIGER) — the article explicitly notes these were not benchmarked with Metanorm
- When the data contain unexpected differences between QC and biological samples — these require analyst scrutiny rather than blind normalization, per the article's recommendation
- When the goal is statistical inference on DAMs requiring controlled false discovery — multiple testing correction is essential but was out of scope for this work

## Inputs

- Unnormalized metabolomics intensity matrix (rows = compounds, columns = sample runs), e.g. ENVIRONAGE urine: 393 runs × 269 metabolites
- Vector of sample types per run ('QC' vs. other) for the metanorm.qc / type argument
- Vector of batch labels per sample run (batch argument, e.g. 6 batches)
- Choice of model: 'tGAM', 'rGAM', 'rLOESS', 'QC-RLSC', or 'QC-RSC'

## Outputs

- Normalized data matrix (same compounds × samples layout), suitable for log-transformed Wilcoxon DAM testing
- Pre- vs. post-normalization per-compound intensity-vs-order diagnostic plots written to plotdir
- PCA score plots of data before and after normalization, labeled by batch

## How to apply

Load the unnormalized numerical matrix (rows = compounds, columns = sample runs), a vector of sample types ('QC' vs. non-QC), and a vector of batch labels. Call metanorm() with the batch argument so drift correction is performed batchwise, choosing model='tGAM' (recommended default for robustness), 'rGAM' or 'rLOESS' (faster alternatives), or 'QC-RLSC'/'QC-RSC' with QConly=TRUE when only QC samples should drive the fit. For QC-based methods, exclude sample runs occurring before the first QC in each batch (47 such runs in the ENVIRONAGE urine dataset). Log-transform intensities (base e) before normalization, and use QCcheck=TRUE plus the package's plotPCA() and per-compound pre/post plots to verify that batch effects and drift have diminished. Rationale: the article shows robust methods improve replicate concordance and reduce drift and batch effects relative to existing strategies, with tGAM exhibiting the strongest performance across experimental datasets.

## Related tools

- **Metanorm R package** (Implements the batchwise normalization call (metanorm() with batch and type arguments) plus diagnostic plotPCA() visualization) — https://github.com/UGent-LIMET/Metanorm
- **R** (Runtime environment (>= 4.4.0) in which the Metanorm package is installed and executed)

## Examples

```
normdat <- metanorm(rawdata, model = "tGAM", type = metanorm.qc, QCcheck = TRUE, batch = batch, plotdir = "~/metanormExample/")
```

## Evaluation signals

- PCA score plots before and after normalization, labeled by batch, should show diminished batch separation post-normalization
- Per-compound pre- vs. post-normalization intensity-vs-order plots should show flattened signal drift (README flags these as the way to catch subpar normalization, e.g. drift in biological samples under QC-RLSC)
- QCcheck=TRUE must not flag discrepancies between QC and biological samples, or those must be scrutinized by the analyst
- For QC-RLSC/QC-RSC, runs preceding the first QC in each batch must have been excluded (47 in ENVIRONAGE)
- Downstream DAM overlap sanity check: robust batchwise methods (rLOESS, rGAM, tGAM) should agree at 74–88% on ENVIRONAGE sex DAMs, while QC-RLSC falls at 47–51%

## Limitations

- QC-sample-based methods (QC-RLSC) can deteriorate in the presence of outliers; the robust methods were introduced to address this
- Automated outlier detection prior to normalization is nontrivial and is not provided by the package
- Outlier treatment is at feature level; multivariate outlier detection is not implemented
- Normalization performance relies on inclusion of QC and replicate samples; without them, quality verification is limited

## Evidence

- [readme] Metanorm supports robust metabolomics data normalization across scales and experimental designs.: [evidence span withheld — anchor status: external_doc]
- [intro] we integrate both existing and the here introduced robust normalization methods into the Metanorm R package.: "we integrate both existing and the here introduced robust normalization methods into the Metanorm R package."
- [abstract] Implementations of these methods are gathered in the Metanorm R package, integrating robust normalization with visualization for performance verification: "Implementations of these methods are gathered in the Metanorm R package, integrating robust normalization with visualization for performance verification"
- [discussion] The pre- vs. postnormalization and PCA score plots readily generated by the Metanorm package support an efficient identification of subpar data quality: "The pre- vs. postnormalization and PCA score plots readily generated by the Metanorm package support an efficient identification of subpar data quality"
- [readme] normalize the first 5 compounds in the example dataset using the (default) tGAM method: [evidence span withheld — anchor status: external_doc]
