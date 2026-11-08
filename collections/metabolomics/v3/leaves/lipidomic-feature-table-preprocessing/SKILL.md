---
name: lipidomic-feature-table-preprocessing
description: Use when raw lipidomic or metabolomic peak-intensity feature tables (e.g.
  MetaboLights S-BSST1038/S-BSST1039 exports) need filtering, imputation, log-transformation,
  and EigenMS normalization before downstream limma or ClassifyR analysis.
when_to_use_negative:
- Input is proteomic data — MultiABLER currently does not provide functions to analyze
  proteomic data, which is another output of the Multi-ABLE extraction method.
- The table has already been filtered, imputed, and normalized (e.g. by TidyMass or
  a prior MultiABLER run) — re-running tbManipulate/normEigenMS would double-process
  the data.
- The required normalization mode is kNN imputation, mean, or quantile normalization
  — these were not implemented in MultiABLER.
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0153
tools:
- name: MultiABLER
  role: Supplies tbManipulate (feature filtering, imputation, log transformation)
    and normEigenMS used to preprocess the feature table
  repo: https://github.com/holab-hku/MultiABLER
- name: EigenMS
  role: Normalization method applied after tbManipulate; identified as best suited
    for MultiABLER data
- name: ProteoMM
  role: Dependency providing EigenMS normalization backend for MultiABLER
  repo: https://www.bioconductor.org/packages/release/bioc/html/ProteoMM.html
- name: limma
  role: Downstream consumer of the preprocessed feature table for differential-expression
    modeling via runLimma
  repo: https://www.bioconductor.org/packages/release/bioc/html/limma.html
- name: devtools
  role: Used to install MultiABLER from GitHub
provenance:
  source_task_ids:
  - task_006
  - task_007
  source_papers:
  - doi: 10.1016/j.isci.2023.106881
    title: Integrative processing of untargeted metabolomic and lipidomic data using
      MultiABLER
  source_content_hash: sha256:14fb1ed4893ba91c1c078388910fe0b8b04da6ee03e0db68103337f096a66b99
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/holab-hku/MultiABLER
  repo_url: https://github.com/holab-hku/MultiABLER
  tools:
  - EigenMS
  - MultiABLER
  - ProteoMM
  - R
  - devtools
  - limma
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0153
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# lipidomic-feature-table-preprocessing

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Preprocess lipidomic and metabolomic peak-intensity feature tables with MultiABLER's tbManipulate (filtering, imputation, log transformation) and EigenMS normalization, producing a clean feature table ready for differential-expression or classification analysis.

## When to use

Use when raw lipidomic or metabolomic peak-intensity feature tables (e.g. MetaboLights S-BSST1038/S-BSST1039 exports) need filtering, imputation, log-transformation, and EigenMS normalization before downstream limma or ClassifyR analysis.

## When NOT to use

- Input is proteomic data — MultiABLER currently does not provide functions to analyze proteomic data, which is another output of the Multi-ABLE extraction method.
- The table has already been filtered, imputed, and normalized (e.g. by TidyMass or a prior MultiABLER run) — re-running tbManipulate/normEigenMS would double-process the data.
- The required normalization mode is kNN imputation, mean, or quantile normalization — these were not implemented in MultiABLER.

## Inputs

- Peak-intensity feature table from untargeted lipidomics or metabolomics LC-MS/MS (e.g. MetaboLights S-BSST1038 liver lipidomic table, S-BSST1039 arterial metabolite/lipid tables from Zenodo record 8039588)
- Sample metadata / sample.info mapping samples to experimental groups (e.g. Apoe+/+ vs Apoe−/−)

## Outputs

- Filtered, imputed, log-transformed and EigenMS-normalized feature table suitable for runLimma or ClassifyR
- Per-feature differential-expression results when combined with runLimma (e.g. DE_results.csv with adjusted p<0.05 and logFC>1 features)

## How to apply

Install MultiABLER from GitHub (devtools::install_github("holab-hku/MultiABLER", dependencies = TRUE)) and, if installation fails, first install ProteoMM, limma, and Tidyverse. Load the peak-intensity feature table into an R data frame and run tbManipulate(df, filter_threshold, impute, log_base) to remove low-quality features, impute missing values, and log-transform abundances with the paper's settings. Then normalize the processed table with normEigenMS(df, sample.info); EigenMS was empirically identified as the best-suited normalization for MultiABLER, outperforming median, NOMIS, and CRMN in 5-fold ClassifyR cross-validation (median balanced accuracy ~0.68, AUC ~0.75 on liver lipidomic data). The resulting feature table feeds directly into runLimma (design/contrast matrices for group contrasts such as WT vs Apoe−/−) or ClassifyR workflows.

## Related tools

- **MultiABLER** (Supplies tbManipulate (feature filtering, imputation, log transformation) and normEigenMS used to preprocess the feature table) — https://github.com/holab-hku/MultiABLER
- **EigenMS** (Normalization method applied after tbManipulate; identified as best suited for MultiABLER data)
- **ProteoMM** (Dependency providing EigenMS normalization backend for MultiABLER) — https://www.bioconductor.org/packages/release/bioc/html/ProteoMM.html
- **limma** (Downstream consumer of the preprocessed feature table for differential-expression modeling via runLimma) — https://www.bioconductor.org/packages/release/bioc/html/limma.html
- **devtools** (Used to install MultiABLER from GitHub)

## Examples

```
devtools::install_github("holab-hku/MultiABLER", dependencies = TRUE)
```

## Evaluation signals

- On the liver lipidomic dataset (S-BSST1038, 35 samples, positive mode), ClassifyR 5-fold × 20-repetition DLDA cross-validation after EigenMS normalization should yield median balanced accuracy ≈0.68 and AUC ≈0.75 (EigenMS+median: BA ≈0.69, AUC ≈0.76), matching published benchmarks within ±0.05.
- On the arterial dataset (S-BSST1039), runLimma at adjusted p<0.05 should recover ~903/1650 lipids DE (414 with logFC>1) and ~601/1314 metabolites DE (334 with logFC>1).
- The output feature table must contain no un-imputed missing values and be on the log scale consistent with the chosen log_base parameter.
- EigenMS-based normalization should rank among the top performers versus none, median, NOMIS, and CRMN variants in the normalization comparison.

## Limitations

- MultiABLER does not implement kNN imputation, mean normalization, or quantile normalization.
- MultiABLER currently does not provide functions to analyze proteomic data; proteomic integration is planned for future development.
- Normalization performance is context-dependent: EigenMS was best for the MultiABLER extraction method, but median, NOMIS, CRMN, etc. may suit other extraction protocols — the paper recommends benchmarking per dataset (as done with ClassifyR DLDA 5-fold/20-repetition comparisons).

## Evidence
