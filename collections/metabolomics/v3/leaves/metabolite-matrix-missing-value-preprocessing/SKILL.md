---
name: metabolite-matrix-missing-value-preprocessing
description: Use when a metabolomics featuredata matrix (metabolite peak intensities/concentrations,
  unique sample names as row names) contains missing values or zeros that must be
  repaired before LogTransform or any normalization step.
when_to_use_negative:
- The featuredata matrix is already complete (no missing intensities and no zeros
  needing conversion) — proceed directly to LogTransform and normalization instead.
- The goal is normalization of unwanted variation rather than missingness repair —
  use NormScaling, NormQcmets, or NormQcsamples instead.
- The data are not in NormalizeMets' three-part format (featuredata/metabolitedata/sampledata)
  and cannot be restructured into it with unique sample and metabolite row/column
  names.
tools:
- name: NormalizeMets
  role: Supplies the MissingValues() function (feature.cutoff/sample.cutoff filtering,
    knn k=10 or replace imputation) applied to the featuredata matrix in this skill
- name: R
  role: Execution environment (version 3.4.3 or higher) in which the NormalizeMets
    functions are called
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1007/s11306-018-1347-7
    title: 'NormalizeMets: assessing, selecting and implementing statistical methods
      for normalizing metabolomics data'
  source_content_hash: sha256:58ad37c21f4305e37f0cc779250033d047f00b078a09f3a3aba8e8c280985f08
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - NormalizeMets
  - R
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# metabolite-matrix-missing-value-preprocessing

## Summary

Detects and imputes missing metabolite peak intensities in a NormalizeMets 'featuredata' matrix (metabolites in columns, samples in rows) using the MissingValues() function with feature/sample 80% completeness cutoffs and KNN (k=10) or simple replacement, so the matrix is complete before log-transformation and normalization.

## When to use

Use when a metabolomics featuredata matrix (metabolite peak intensities/concentrations, unique sample names as row names) contains missing values or zeros that must be repaired before LogTransform or any normalization step.

## When NOT to use

- The featuredata matrix is already complete (no missing intensities and no zeros needing conversion) — proceed directly to LogTransform and normalization instead.
- The goal is normalization of unwanted variation rather than missingness repair — use NormScaling, NormQcmets, or NormQcsamples instead.
- The data are not in NormalizeMets' three-part format (featuredata/metabolitedata/sampledata) and cannot be restructured into it with unique sample and metabolite row/column names.

## Inputs

- featuredata matrix: metabolite peak intensities or concentrations, metabolites in columns, samples in rows with unique sample names as row names
- sampledata dataframe: sample-specific information (sample type, analysis order, factors of interest) keyed by unique sample names
- metabolitedata dataframe: metabolite-specific annotations (e.g., internal/external standards, positive/negative controls) with metabolite names as row names

## Outputs

- Imputed featuredata matrix with no remaining missing values in retained rows/columns, ready for LogTransform and downstream normalization or visualization

## How to apply

In R (version 3.4.3 or higher), load NormalizeMets and assemble its three-part input: featuredata (metabolites in columns, samples in rows), sampledata, and metabolitedata (a dataframe of metabolite-specific annotations such as internal/external standards, with metabolite names as row names). Call MissingValues(featuredata, sampledata, as.dataframe(metabolitedata), ...) with the default feature.cutoff = 0.8 and sample.cutoff = 0.8, which filter metabolites and samples exceeding the missingness thresholds, and choose method = 'knn' (default k = 10) for similarity-based imputation or method = 'replace' for simple replacement depending on the nature of the missing data. This step precedes LogTransform(featuredata, base = exp(1), zerotona = TRUE), which itself converts residual zeros to missing values before taking logs, so a complete matrix is required for meaningful downstream normalization (NormQcmets, NormQcsamples, NormScaling, NormCombined).

## Related tools

- **NormalizeMets** (Supplies the MissingValues() function (feature.cutoff/sample.cutoff filtering, knn k=10 or replace imputation) applied to the featuredata matrix in this skill)
- **R** (Execution environment (version 3.4.3 or higher) in which the NormalizeMets functions are called)

## Examples

```
imputed <- MissingValues(featuredata, sampledata, as.dataframe(metabolitedata), feature.cutoff = 0.8, sample.cutoff = 0.8, method = c("knn", "replace"), k = 10)
```

## Evaluation signals

- After MissingValues(), the returned featuredata matrix contains no NA entries (given method='knn' imputation) and retains only metabolites and samples meeting the 0.8 completeness cutoffs.
- Row names of the imputed matrix remain the unique sample names of the original featuredata and match sampledata row names.
- Column count reflects removal of metabolites failing feature.cutoff = 0.8; row count reflects removal of samples failing sample.cutoff = 0.8.
- A subsequent LogTransform(zerotona = TRUE) call yields finite log-transformed values across the imputed matrix with no errors from residual NAs.
- Imputed values appear only at previously missing positions; observed peak intensities should be unchanged relative to the raw matrix.

## Limitations

- Default cutoffs (feature.cutoff = 0.8, sample.cutoff = 0.8) may discard many metabolites or samples in sparse datasets, shrinking the matrix substantially.
- KNN imputation uses a default neighborhood of k = 10, which may be inappropriate for very small cohorts or highly structured missingness; the 'replace' method is the simpler alternative.
- Requires R version 3.4.3 or higher; the Excel GUI alternative additionally requires Microsoft Excel (2016 or later) on Windows 7 or higher.

## Evidence

- [readme] Input format definition: [evidence span withheld — anchor status: external_doc]
