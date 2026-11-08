---
name: metabolomics-intensity-log-transformation
description: Use when raw (untransformed) metabolomics feature intensities (e.g.,
  a compounds x samples matrix such as the 393-run, 269-metabolite ENVIRONAGE urine
  dataset) feed into Metanorm normalization (tGAM, rGAM, rLOESS, QC-RLSC, QC-RSC)
  or downstream DAM testing.
when_to_use_negative:
- The input matrix is already on a log scale (e.g., the Metanorm example rawdata values
  are ~16.1–16.8, consistent with log-transformed intensities) — re-transforming would
  double-transform and distort the drift fits
- The matrix contains zero or negative intensities, for which the base-e logarithm
  is undefined and requires prior imputation or a different handling step
- The data are count-based (e.g., sequencing counts) rather than continuous LC-MS/MS
  intensity measurements, where other variance-stabilizing transforms are appropriate
edam_operation: http://edamontology.org/operation_2935
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: Metanorm R package
  role: Consumes the log-transformed intensity matrix (rawdata) together with batch
    and sample-type vectors as input to metanorm() normalization
  repo: https://github.com/UGent-LIMET/Metanorm
- name: R
  role: Environment in which the element-wise base-e log transform of the intensity
    matrix is performed (log()) before normalization
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1021/acs.analchem.5c06841
    title: Robust Metabolomics Data Normalization across Scales and Experimental Designs
  source_content_hash: sha256:959e6f66b3f42bb5d589fefcf75383f66312ec0c6ed39bdd1d2f1ab3c4e488b3
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
  edam_operation: http://edamontology.org/operation_2935
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# metabolomics-intensity-log-transformation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Log-transforms (base e) raw metabolomics intensity matrices before normalization and differential abundance testing, as done in the Metanorm/ENVIRONAGE workflow so that intensity values are on an additive scale suitable for drift/batch normalization and Wilcoxon DAM testing.

## When to use

Use when raw (untransformed) metabolomics feature intensities (e.g., a compounds x samples matrix such as the 393-run, 269-metabolite ENVIRONAGE urine dataset) feed into Metanorm normalization (tGAM, rGAM, rLOESS, QC-RLSC, QC-RSC) or downstream DAM testing.

## When NOT to use

- The input matrix is already on a log scale (e.g., the Metanorm example rawdata values are ~16.1–16.8, consistent with log-transformed intensities) — re-transforming would double-transform and distort the drift fits
- The matrix contains zero or negative intensities, for which the base-e logarithm is undefined and requires prior imputation or a different handling step
- The data are count-based (e.g., sequencing counts) rather than continuous LC-MS/MS intensity measurements, where other variance-stabilizing transforms are appropriate

## Inputs

- Raw metabolomics intensity matrix (rows = compounds/metabolites, columns = sample runs), e.g., ENVIRONAGE urine: 393 runs x 269 metabolites across 6 batches
- Vector of sample run order / batch labels
- Vector of sample types (QC vs biological samples)

## Outputs

- Natural-log-transformed intensity matrix of the same dimensions as the input, ready for metanorm() normalization
- Downstream: normalized matrices per method and DAM lists (two-sided Wilcoxon rank sum tests at p < 0.05) computed on the transformed scale

## How to apply

Load the numerical intensity matrix (rows = compounds, columns = samples), confirm all intensities are strictly positive, and apply the natural logarithm (base e) element-wise before any Metanorm normalization step. In the ENVIRONAGE contrast analysis this transform is applied jointly with excluding the 47 pre-first-QC samples per batch (relevant only for QC-RLSC/QC-RSC) and retaining batch and QC/sample-type vectors. Log-transformed intensities (example README values ~16.1–16.8) are then passed to metanorm() with the chosen model ('tGAM' default, or 'rGAM', 'rLOESS', 'QC-RLSC', 'QC-RSC'), the sample-type vector, and the batch argument for batchwise normalization. The rationale is that normalization models (robust LOESS/GAM fits of signal drift) and log-ratio-style comparisons assume additive, approximately symmetric errors on the transformed scale; skipping the transform leaves multiplicative drift that the models fit poorly.

## Related tools

- **Metanorm R package** (Consumes the log-transformed intensity matrix (rawdata) together with batch and sample-type vectors as input to metanorm() normalization) — https://github.com/UGent-LIMET/Metanorm
- **R** (Environment in which the element-wise base-e log transform of the intensity matrix is performed (log()) before normalization)

## Examples

```
normdat <- metanorm(log(rawdata), model = "tGAM", type = metanorm.qc, batch = batch)
```

## Evaluation signals

- All input intensities are strictly positive before applying log(); the transformed matrix has the same rows x columns dimensions as the input
- Transformed values fall in a compact positive range comparable to the Metanorm example (rawdata ~16.1–16.8), and intensity distributions become approximately symmetric rather than right-skewed
- Downstream DAM detection on the transformed data reproduces the ENVIRONAGE benchmark: robust-method DAM overlap 74–88%, QC-RLSC 47–51%, SERFF 68–70%, TIGER 64–67%
- Pairwise Spearman correlations of method p-values and the complete-linkage heatmap (Euclidean distances of one-minus-correlation vectors) can be computed without errors on results derived from the transformed matrix

## Limitations

- The article does not apply multiple testing correction downstream ('While not in the scope of this work, multiple testing correction is essential for downstream statistical inference'), so p < 0.05 DAM lists on the log-transformed data are unadjusted
- QC-RLSC and QC-RSC additionally require excluding samples occurring before the first QC in each batch (47 excluded ENVIRONAGE runs), so the transform alone does not resolve batch-structure handling
- Unexpected differences between QC and biological samples on the transformed scale should receive due analyst scrutiny rather than automated handling

## Evidence

- [other] Load the ENVIRONAGE urine dataset (393 runs, 269 metabolites, 6 batches), log-transform intensities (base e), and exclude the 47 samples occurring before the first QC in each batch: [evidence span withheld — anchor status: not_found]
- [readme] rawdata, a numerical matrix containing the unnormalized data (rows = compounds, columns = samples): [evidence span withheld — anchor status: not_found]
- [other] For each normalized dataset, run two-sided Wilcoxon rank sum tests at p < 0.05 to determine DAMs for biological sex.: [evidence span withheld — anchor status: not_found]
- [readme] normalize the first 5 compounds in the example dataset using the (default) tGAM method: [evidence span withheld — anchor status: external_doc]
