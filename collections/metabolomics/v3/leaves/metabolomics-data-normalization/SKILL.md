---
name: metabolomics-data-normalization
description: Use when a raw or peak-picked feature table from Multi-ABLE LC-MS/MS
  acquisition contains sample-to-sample intensity bias that must be removed before
  differential expression analysis, sPLS, or rCCA on jointly generated metabolomic
  and lipidomic data.
when_to_use_negative:
- 'Proteomic data analysis: MultiABLER currently does not provide functions to analyze
  proteomic data, so this normalization workflow is not the right entry point for
  proteomics outputs.'
- When kNN imputation, mean normalization, or quantile normalization is specifically
  required — these methods were not implemented in MultiABLER and are outside this
  skill's scope.
edam_operation: http://edamontology.org/operation_3637
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: MultiABLER
  role: R pipeline providing the normEigenMS and normMedian functions that implement
    the EigenMS-then-median normalization sequence on the feature table
  repo: https://github.com/holab-hku/MultiABLER
- name: ProteoMM
  role: Bioconductor package supplying the EigenMS normalization routine that normEigenMS
    wraps
  repo: https://www.bioconductor.org/packages/release/bioc/html/ProteoMM.html
- name: Tidyverse
  role: Used with readr to load the feature table and coerce it to a tibble before
    normalization
  repo: https://www.tidyverse.org
- name: R
  role: Language in which the normEigenMS and normMedian functions are implemented
    and chained
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.isci.2023.106881
    title: Integrative processing of untargeted metabolomic and lipidomic data using
      MultiABLER
  source_content_hash: sha256:4f7eac5f1bfe1eb9caa1bd62d1c500d559f5384b4c00581aa905e19f910e7867
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
  - MultiABLER
  - ProteoMM
  - R
  - Tidyverse
  edam_operation: http://edamontology.org/operation_3637
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 12
---

# metabolomics-data-normalization

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A two-stage normalization workflow from the MultiABLER R pipeline that applies EigenMS normalization (via the ProteoMM package) followed by median normalization to an LC-MS/MS feature table, equalizing sample medians so metabolomic and lipidomic features can be compared jointly.

## When to use

Use when a raw or peak-picked feature table from Multi-ABLE LC-MS/MS acquisition contains sample-to-sample intensity bias that must be removed before differential expression analysis, sPLS, or rCCA on jointly generated metabolomic and lipidomic data.

## When NOT to use

- Proteomic data analysis: MultiABLER currently does not provide functions to analyze proteomic data, so this normalization workflow is not the right entry point for proteomics outputs.
- When kNN imputation, mean normalization, or quantile normalization is specifically required — these methods were not implemented in MultiABLER and are outside this skill's scope.

## Inputs

- Feature table from MultiABLER extraction (rows = features, columns = samples) read from file with readr
- Sample metadata / sample information accompanying the feature table

## Outputs

- Normalized feature table written to normalized_feature_table.csv via readr, with the same features and samples but adjusted intensities

## How to apply

Load the feature table (and sample metadata) with readr and coerce it to a tibble using tidyverse conventions. Implement normEigenMS as a wrapper around the EigenMS routine from the ProteoMM package, passing the feature table and sample information; EigenMS was identified in the article as the best suited normalization method for MultiABLER. Then implement normMedian to scale each sample (column) by its median intensity so that all samples share the same median. Chain normEigenMS followed by normMedian in sequence on the loaded feature table, confirming the returned object is a normalized feature table with the same features and samples but adjusted intensities. Note that mean normalization, quantile normalization, and kNN imputation were deliberately not implemented in MultiABLER, so EigenMS + median is the pipeline's prescribed normalization order.

## Related tools

- **MultiABLER** (R pipeline providing the normEigenMS and normMedian functions that implement the EigenMS-then-median normalization sequence on the feature table) — https://github.com/holab-hku/MultiABLER
- **ProteoMM** (Bioconductor package supplying the EigenMS normalization routine that normEigenMS wraps) — https://www.bioconductor.org/packages/release/bioc/html/ProteoMM.html
- **Tidyverse** (Used with readr to load the feature table and coerce it to a tibble before normalization) — https://www.tidyverse.org
- **R** (Language in which the normEigenMS and normMedian functions are implemented and chained)

## Examples

```
devtools::install_github("holab-hku/MultiABLER", dependencies = TRUE)
```

## Evaluation signals

- The normalized output retains the same features and samples as the input feature table, with only intensities adjusted.
- After normMedian, the median intensity is equalized across all samples in the table.
- The output is a valid normalized feature table suitable for downstream differential expression analysis, sPLS, or rCCA.
- The workflow runs as normEigenMS followed by normMedian (EigenMS applied first, median normalization second).
- The normalized table is written successfully to normalized_feature_table.csv via readr without row or column loss.

## Limitations

- MultiABLER currently does not provide functions to analyze proteomic data, which is another important output from the MultiABLER extraction method; proteomic integration is planned for future development.
- Other normalization and imputation methods — k-nearest neighbours (kNN) imputation, mean normalization, and quantile normalization — were not implemented in MultiABLER.
- Installation issues can arise; in that case install the ProteoMM, limma, and Tidyverse dependencies first and rerun devtools::install_github("holab-hku/MultiABLER", dependencies = TRUE).

## Evidence
