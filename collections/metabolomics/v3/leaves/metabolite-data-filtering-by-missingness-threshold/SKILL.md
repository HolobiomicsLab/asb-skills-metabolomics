---
name: metabolite-data-filtering-by-missingness-threshold
description: Use when preparing large-scale targeted metabolomics data (e.g. BioHEART-CT
  plasma, LC-MS/MS peaks integrated from MultiQuant) before hRUV/RUV-III normalization,
  which requires a complete (no-missing-value) metabolite-by-sample matrix.
when_to_use_negative:
- The matrix already contains no missing values — imputation is unnecessary and filtering
  at 50% could discard usable features.
- Analysis targets feature discovery (e.g. metabolites present in only one batch)
  rather than cross-batch integration; the 50% per-batch cutoff and cross-batch intersection
  will remove such features.
- Input is a feature table intended for a method that tolerates missingness (not hRUV),
  so the completeness constraint does not apply.
edam_operation: http://edamontology.org/operation_3661
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0219
tools:
- name: hRUV
  role: Downstream normalization workflow that motivates the missing-value filter,
    cross-batch intersection, and kNN imputation (rawImpute assay)
  repo: https://github.com/SydneyBioX/hRUV
- name: DMwR2
  role: Provides the kNN imputation used on residual missing values after filtering
    and intersection
- name: R
  role: Implementation language for the clean()/filtering/imputation pipeline
- name: MultiQuant (SCIEX)
  role: Upstream peak integration producing the calibrated peak areas that form the
    raw input matrix
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1101/2020.12.21.423723
    title: 'hRUV: Hierarchical approach to removal of unwanted variation for large-scale
      metabolomics data'
  source_content_hash: sha256:0abfbeb72edafb9f8b58634f83207f5d9a6d3322912e2f8cfe323860a5720751
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/SydneyBioX/hRUV
  tools:
  - DMwR2
  - MultiQuant (SCIEX)
  - R
  - hRUV
  edam_operation: http://edamontology.org/operation_3661
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0219
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# metabolite-data-filtering-by-missingness-threshold

## Summary

Filter metabolites measured across batches by dropping features missing in more than 50% of samples, then restrict to the intersection of metabolites quantified in every batch, and finally impute the residual missing values with k-nearest-neighbour imputation (DMwR2) to yield the complete rawImpute assay required by hRUV normalization.

## When to use

Use when preparing large-scale targeted metabolomics data (e.g. BioHEART-CT plasma, LC-MS/MS peaks integrated from MultiQuant) before hRUV/RUV-III normalization, which requires a complete (no-missing-value) metabolite-by-sample matrix.

## When NOT to use

- The matrix already contains no missing values — imputation is unnecessary and filtering at 50% could discard usable features.
- Analysis targets feature discovery (e.g. metabolites present in only one batch) rather than cross-batch integration; the 50% per-batch cutoff and cross-batch intersection will remove such features.
- Input is a feature table intended for a method that tolerates missingness (not hRUV), so the completeness constraint does not apply.

## Inputs

- Raw metabolite relative-abundance matrix (metabolite × sample) from LC-MS/MS peak integration (MultiQuant, SCIEX)
- Batch/run metadata mapping each sample to its acquisition batch (e.g. 96-well plates run over 44 days)

## Outputs

- rawImpute assay: imputed metabolite-by-sample matrix with no missing values, ready for hRUV normalization

## How to apply

Load the raw relative-abundance matrix and split it by acquisition batch. Apply the clean() step to remove any metabolite missing in more than 50% of samples within each batch, then compute the intersection of retained metabolites across all batches so analysis uses features quantified in every batch. For remaining missing values, perform k-nearest-neighbour imputation (kNN from the DMwR2 package) and store the result in a new assay called rawImpute. This is necessary because the hRUV algorithm expects data without missing values, which is often not possible in large-scale metabolomics data.

## Related tools

- **hRUV** (Downstream normalization workflow that motivates the missing-value filter, cross-batch intersection, and kNN imputation (rawImpute assay)) — https://github.com/SydneyBioX/hRUV
- **DMwR2** (Provides the kNN imputation used on residual missing values after filtering and intersection)
- **R** (Implementation language for the clean()/filtering/imputation pipeline)
- **MultiQuant (SCIEX)** (Upstream peak integration producing the calibrated peak areas that form the raw input matrix)

## Examples

```
library(hRUV); filtered <- hRUV::clean(raw_se, missing_threshold = 0.5); imputed <- hRUV::impute(filtered, method = "kNN"); se <- hRUV::hruv(imputed, replicates = batch_replicates)
```

## Evaluation signals

- After imputation, the rawImpute assay must contain no missing values, consistent with hRUV's requirement of complete data.
- Every metabolite retained in the final matrix must be present in ≥50% of samples within each batch (otherwise it should have been filtered by clean()).
- Every retained metabolite must be quantified in all batches — the matrix is the intersect of per-batch retained features.
- Row/column dimension of rawImpute must match the filtered intersect feature set and original sample order.
- Imputed values should be finite numeric (no NaN/Inf) and preserve the assay structure (SummarizedExperiment-style) expected by hRUV.

## Limitations

- The 50%-missing per-batch cutoff and cross-batch intersection can be aggressive: metabolites absent from an entire batch are permanently discarded.
- kNN imputation is a workaround, not a principled treatment of missingness; imputed values are fabricated data and downstream biological interpretation of those values should be cautious.
- Batch-replicate sampling can cluster by chance in one half of a plate, limiting coverage of positional unwanted variability, which affects features whose missingness is position-dependent (documented between batches 6–7 and 13–14).
- The hRUV algorithm expects data without missing values, which is often not possible in large-scale metabolomics data — forcing completeness via imputation is a known compromise.

## Evidence

- [discussion] Metabolites not present in at least 50% of samples were filtered out: "metabolites that were not present in at least 50% of the samples were filtered out"
- [discussion] k-nearest neighbour imputation produces rawImpute assay: [evidence span withheld — anchor status: not_found]
- [discussion] hRUV expects complete data: "While the proposed hRUV algorithm expects data without missing values, this is often not possible in large-scale metabolomics data"
- [readme] DMwR2 is a dependency (provides kNN): "### CRAN

dplyr
tidyr
rlang
ggplot2
plotly
S4Vectors
tibble
DMwR2
MASS"
- [full_text] rawImpute contains no missing values after imputation: [evidence span withheld — anchor status: not_found]
- [discussion] Metabolite peaks integrated from MultiQuant: "Metabolite abundance peaks were integrated using the area under the curve for calibrated peaks from MultiQuant (SCIEX)"
