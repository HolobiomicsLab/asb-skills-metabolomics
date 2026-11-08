---
name: median-normalization-of-omics-feature-tables
description: Use when a MultiABLER-derived metabolomic or lipidomic feature table
  needs sample-level intensity scaling so that all samples share the same median intensity
  before joint downstream analyses.
when_to_use_negative:
- The pipeline is designed for MultiABLER-extracted metabolomic and lipidomic LC-MS/MS
  feature tables, not for proteomic data — MultiABLER currently does not provide functions
  to analyze proteomic data
- When kNN imputation, mean normalization, or quantile normalization are desired —
  these were not implemented in MultiABLER
- The input is not a feature table (e.g. raw spectral files), since the skill operates
  on an already-extracted feature table
edam_operation: http://edamontology.org/operation_3660
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: MultiABLER
  role: R pipeline providing normMedian and normEigenMS normalization functions applied
    to the feature table
  repo: https://github.com/holab-hku/MultiABLER
- name: ProteoMM
  role: Provides the EigenMS normalization routine (normEigenMS) that is chained before
    median normalization in MultiABLER
  repo: https://www.bioconductor.org/packages/release/bioc/html/ProteoMM.html
- name: Tidyverse
  role: readr/tibble/tidyverse conventions used to load and coerce the feature table
    before normalization
  repo: https://www.tidyverse.org
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.isci.2023.106881
    title: Integrative processing of untargeted metabolomic and lipidomic data using
      MultiABLER
  source_content_hash: sha256:868ea793f9d8a6c8643ae619c20ab2ed925cfefe9571190341b98cf113c466f6
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
  edam_operation: http://edamontology.org/operation_3660
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# Median normalization of omics feature tables

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Equalize per-sample medians of an LC-MS/MS feature table by scaling each sample column to a common median intensity, as implemented in MultiABLER for metabolomic and lipidomic data. It is applied after EigenMS normalization to produce a uniformly comparable feature table for downstream joint analyses.

## When to use

Use when a MultiABLER-derived metabolomic or lipidomic feature table needs sample-level intensity scaling so that all samples share the same median intensity before joint downstream analyses.

## When NOT to use

- The pipeline is designed for MultiABLER-extracted metabolomic and lipidomic LC-MS/MS feature tables, not for proteomic data — MultiABLER currently does not provide functions to analyze proteomic data
- When kNN imputation, mean normalization, or quantile normalization are desired — these were not implemented in MultiABLER
- The input is not a feature table (e.g. raw spectral files), since the skill operates on an already-extracted feature table

## Inputs

- MultiABLER metabolomic or lipidomic feature table (sample-by-feature intensity matrix)
- Sample metadata / sample information accompanying the feature table

## Outputs

- Normalized feature table with equalized per-sample median intensities (same features and samples, adjusted intensities)
- normalized_feature_table.csv written via readr

## How to apply

Load the feature table (and sample metadata) with readr and coerce it to a tibble using tidyverse conventions. In MultiABLER, first apply EigenMS normalization — the authors identified EigenMS as the best suited normalization method — by wrapping the normEigenMS routine from the ProteoMM dependency, taking the feature table and sample information as arguments. Then implement median normalization by dividing or scaling each sample (column) by its median intensity so that sample medians are equalized across all samples and across the different omics modalities. Chain the two normalization functions in sequence and confirm the returned object is a normalized feature table with the same features and samples but adjusted intensities.

## Related tools

- **MultiABLER** (R pipeline providing normMedian and normEigenMS normalization functions applied to the feature table) — https://github.com/holab-hku/MultiABLER
- **ProteoMM** (Provides the EigenMS normalization routine (normEigenMS) that is chained before median normalization in MultiABLER) — https://www.bioconductor.org/packages/release/bioc/html/ProteoMM.html
- **Tidyverse** (readr/tibble/tidyverse conventions used to load and coerce the feature table before normalization) — https://www.tidyverse.org

## Examples

```
devtools::install_github("holab-hku/MultiABLER", dependencies = TRUE); normEigenMS(feature_table, sample_info); normMedian(feature_table); write_csv(normalized_feature_table, "normalized_feature_table.csv")
```

## Evaluation signals

- The returned object is a normalized feature table with the same features and samples as the input, only intensities adjusted
- Per-sample (column) medians of the normalized feature table are equalized across all samples and across metabolomic and lipidomic modalities
- EigenMS normalization (via ProteoMM) has been applied first, and median normalization is run on top of it, matching MultiABLER's documented order
- The normalized table can be written to normalized_feature_table.csv with readr without schema errors
- The feature table remains suitable for differential expression analysis, sPLS, and rCCA downstream

## Limitations

- MultiABLER currently does not provide functions to analyze proteomic data
- Other normalization methods such as kNN imputation, mean normalization, and quantile normalization were not implemented in MultiABLER
- EigenMS normalization is applied before median normalization, so skipping EigenMS may change the intended output; the two are documented as a chain, not alternatives

## Evidence
