---
name: limma-differential-abundance-modeling
description: Use when a normalized MultiABLER feature table plus sample metadata are
  available and the goal is group-level differential expression of lipids or metabolites,
  such as genotype contrasts in mouse arterial tissue.
when_to_use_negative:
- The input has not been processed by tbManipulate and normalized with normEigenMS
  (EigenMS) — MultiABLER does not implement kNN imputation, mean normalization, or
  quantile normalization, so running limma on un-normalized tables violates the reference
  workflow.
- The data are proteomic — MultiABLER currently does not provide functions to analyze
  proteomic data, and proteomic integration is planned only for future development.
- The analysis goal is multivariate integration (sPLS or rCCA) rather than univariate
  group-contrast differential expression testing.
edam_operation: http://edamontology.org/operation_3661
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0153
tools:
- name: limma
  role: Fits the linear model for differential-expression testing via runLimma(df,
    design.matrix, cont.matrix)
  repo: https://www.bioconductor.org/packages/release/bioc/html/limma.html
- name: MultiABLER
  role: Provides the runLimma, tbManipulate, and normEigenMS functions that prepare
    feature tables and run the limma model
  repo: https://github.com/holab-hku/MultiABLER
- name: EigenMS
  role: Normalization method applied before limma fitting via normEigenMS(df, sample.info)
- name: ProteoMM
  role: Dependency of MultiABLER; supports the imputation/normalization layer feeding
    into runLimma
  repo: https://www.bioconductor.org/packages/release/bioc/html/ProteoMM.html
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.isci.2023.106881
    title: Integrative processing of untargeted metabolomic and lipidomic data using
      MultiABLER
  source_content_hash: sha256:bd80f4a4735e96ffaa1d1e4e7c778695418db47d07e9e58867d0417e6b4e278d
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/holab-hku/MultiABLER
  tools:
  - EigenMS
  - MultiABLER
  - ProteoMM
  - R
  - limma
  edam_operation: http://edamontology.org/operation_3661
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0153
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# limma-differential-abundance-modeling

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Fit a limma linear model on MultiABLER-processed metabolite/lipid feature tables to call differentially expressed features between biological groups (e.g. Apoe+/+ vs Apoe−/− mice) at adjusted p < 0.05.

## When to use

Use when a normalized MultiABLER feature table plus sample metadata are available and the goal is group-level differential expression of lipids or metabolites, such as genotype contrasts in mouse arterial tissue.

## When NOT to use

- The input has not been processed by tbManipulate and normalized with normEigenMS (EigenMS) — MultiABLER does not implement kNN imputation, mean normalization, or quantile normalization, so running limma on un-normalized tables violates the reference workflow.
- The data are proteomic — MultiABLER currently does not provide functions to analyze proteomic data, and proteomic integration is planned only for future development.
- The analysis goal is multivariate integration (sPLS or rCCA) rather than univariate group-contrast differential expression testing.

## Inputs

- Metabolite feature table from Multi-ABLE LC-MS/MS acquisition (Zenodo S-BSST1039, record 8039588)
- Lipid feature table from Multi-ABLE LC-MS/MS acquisition
- Sample metadata with genotype group labels (Apoe+/+ vs Apoe−/−)
- Design matrix with group contrasts
- Contrast matrix for limma

## Outputs

- Per-feature differential-expression statistics table (p-values, adjusted p < 0.05, logFC)
- Combined DE results file DE_results.csv
- Counts of differentially expressed features per omics layer (903 lipids DE / 601 metabolites DE in the reference study)

## How to apply

Load the S-BSST1039 arterial metabolite and lipid feature tables from Zenodo (https://zenodo.org/records/8039588), process each with tbManipulate(df, filter_threshold, impute, log_base), and normalize with normEigenMS(df, sample.info) since EigenMS was identified as the best-suited normalization for MultiABLER. Construct a design matrix encoding the genotype group contrast (WT Apoe+/+ vs Apoe−/−) with the sample metadata, then fit the limma linear model per omics layer via runLimma(df, design.matrix, cont.matrix). Extract results at adjusted p < 0.05 and, where required, apply a log-fold-change threshold of logFC > 1. In the reference study this yielded 903/1650 differentially expressed lipids (414 with logFC > 1) and 601/1314 metabolites (334 with logFC > 1); save the combined per-feature results as DE_results.csv.

## Related tools

- **limma** (Fits the linear model for differential-expression testing via runLimma(df, design.matrix, cont.matrix)) — https://www.bioconductor.org/packages/release/bioc/html/limma.html
- **MultiABLER** (Provides the runLimma, tbManipulate, and normEigenMS functions that prepare feature tables and run the limma model) — https://github.com/holab-hku/MultiABLER
- **EigenMS** (Normalization method applied before limma fitting via normEigenMS(df, sample.info))
- **ProteoMM** (Dependency of MultiABLER; supports the imputation/normalization layer feeding into runLimma) — https://www.bioconductor.org/packages/release/bioc/html/ProteoMM.html

## Examples

```
runLimma(df, design.matrix, cont.matrix)
```

## Evaluation signals

- Differential-expression counts at adjusted p < 0.05 match the reported values: 903 of 1650 lipids and 601 of 1314 metabolites
- With the additional logFC > 1 threshold, 414 lipids and 334 metabolites pass
- The combined per-feature output file DE_results.csv is produced
- Feature tables were log-transformed via tbManipulate and normalized with normEigenMS before model fitting
- Adjusted p-values come from a limma linear model with a genotype group contrast design matrix

## Limitations

- MultiABLER does not implement kNN imputation, mean normalization, or quantile normalization, so only EigenMS normalization is validated for this workflow
- Proteomic data cannot be analyzed with MultiABLER; proteomic integration is planned for future development
- The reference case study used only limma for differential expression; other methods were not validated in this workflow

## Evidence

- [results] Applying the MultiABLER pipeline to arterial tissue from Apoe+/+ and Apoe−/− mice, a limma model identified 903 lipids and 601 metabolites as differentially expressed at adjusted p < 0.05.: [evidence span withheld — anchor status: not_found]
