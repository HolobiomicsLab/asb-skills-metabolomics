---
name: cross-batch-metabolite-intersection
description: Use when preparing multi-batch metabolomics intensity matrices (e.g.
  >1,000 plasma samples run over many batches/plates) for hRUV or other normalization
  that requires complete, missing-value-free data.
when_to_use_negative:
- Input is a single-batch dataset where batch-wise filtering and cross-batch intersection
  are unnecessary
- Downstream method tolerates missing values (hRUV itself requires complete data,
  but other normalizers may not need this imputation workaround)
- Metabolites are already filtered and the matrix is complete — re-running the 50%
  filter and intersection would discard valid features
edam_operation: http://edamontology.org/operation_3660
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: hRUV
  role: R package whose clean() function performs the per-batch 50% missing-value
    filter, cross-batch metabolite intersection, and kNN imputation into the rawImpute
    assay
  repo: github.com/SydneyBioX/hRUV
- name: DMwR2
  role: CRAN dependency supplying the kNN imputation function used by clean() to fill
    residual missing values
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1101/2020.12.21.423723
    title: 'hRUV: Hierarchical approach to removal of unwanted variation for large-scale
      metabolomics data'
  source_content_hash: sha256:ed3d2c0f6f7f4217c7c5a02698073c8c734febe3af6193d40cd4f7c7f4173a04
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - DMwR2
  - R
  - hRUV
  edam_operation: http://edamontology.org/operation_3660
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# Cross-batch metabolite intersection

## Summary

A data-cleaning step in the hRUV metabolomics workflow that removes metabolites missing in more than 50% of samples within each batch, retains only the metabolites quantified across all batches (intersect), and imputes residual missing values with kNN to produce the analysis-ready rawImpute assay.

## When to use

Use when preparing multi-batch metabolomics intensity matrices (e.g. >1,000 plasma samples run over many batches/plates) for hRUV or other normalization that requires complete, missing-value-free data.

## When NOT to use

- Input is a single-batch dataset where batch-wise filtering and cross-batch intersection are unnecessary
- Downstream method tolerates missing values (hRUV itself requires complete data, but other normalizers may not need this imputation workaround)
- Metabolites are already filtered and the matrix is complete — re-running the 50% filter and intersection would discard valid features

## Inputs

- Raw multi-batch metabolomics abundance matrix (metabolite x sample peak areas, e.g. from MultiQuant peak integration)
- Batch/plate sample annotation defining which samples belong to each batch

## Outputs

- A filtered, intersected metabolite x sample matrix with residual missing values kNN-imputed, stored as the rawImpute assay in a SummarizedExperiment object

## How to apply

Load the raw metabolomics abundance matrix (e.g. peak areas integrated with MultiQuant from an Agilent 1260 Infinity HPLC / Qtrap5500 acquisition) and split it into batches. Within each batch, filter out metabolites not present in at least 50% of the samples, then compute the intersection of retained metabolites across all batches so the analysis uses only metabolites measured in every batch. Impute the residual missing values in the filtered matrix using k-nearest-neighbour imputation (DMwR2 kNN in the hRUV R package), storing the result in a new SummarizedExperiment assay named rawImpute. This imputation is required because the hRUV algorithm expects data without missing values, which is often not possible in large-scale metabolomics.

## Related tools

- **hRUV** (R package whose clean() function performs the per-batch 50% missing-value filter, cross-batch metabolite intersection, and kNN imputation into the rawImpute assay) — github.com/SydneyBioX/hRUV
- **DMwR2** (CRAN dependency supplying the kNN imputation function used by clean() to fill residual missing values)

## Examples

```
library(hRUV); se_clean <- clean(se_raw, missing_threshold = 0.5); assay(se_clean, "rawImpute")  # per-batch 50% filter, cross-batch intersect, DMwR2::kNN imputation
```

## Evaluation signals

- The rawImpute assay contains no missing values after imputation, consistent with hRUV's requirement of complete data
- Every metabolite retained in the final matrix passes the ≥50% presence threshold in each individual batch, not just globally
- The metabolite set equals the intersection across all batches — no metabolite present in some but not all batches remains
- The rawImpute assay is stored as a distinct assay alongside the raw data so the pre-imputation filtered matrix remains inspectable

## Limitations

- The 50% per-batch presence cutoff can be stringent for large-scale studies, potentially discarding low-abundance metabolites
- kNN imputation introduces values that were never measured; imputed intensities should not be interpreted as observed peaks
- Filtering and intersection must happen before hRUV because the algorithm expects data without missing values, which is often not possible in large-scale metabolomics

## Evidence

- [methods] After the filtering process, the clean function performs k-nearest neighbour imputation on the remaining missing values, storing the resulting imputed data in a new assay named rawImpute.: [evidence span withheld — anchor status: not_found]
- [discussion] While the proposed hRUV algorithm expects data without missing values, this is often not possible in large-scale metabolomics data: "While the proposed hRUV algorithm expects data without missing values, this is often not possible in large-scale metabolomics data"
- [discussion] metabolites that were not present in at least 50% of the samples were filtered out: "metabolites that were not present in at least 50% of the samples were filtered out"
