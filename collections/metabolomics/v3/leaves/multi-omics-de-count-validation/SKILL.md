---
name: multi-omics-de-count-validation
description: Use when processing concurrently generated untargeted metabolomic and
  lipidomic LC-MS/MS feature tables (e.g. from the Multi-ABLE method) through filtering,
  imputation, EigenMS normalization, and limma differential expression, then verifying
  reported DE feature counts at adjusted p < 0.05.
when_to_use_negative:
- Input is already a filtered, normalized feature table — tbManipulate and EigenMS
  reprocessing would distort the data
- 'The data are proteomic: MultiABLER does not yet provide functions to analyze proteomic
  data, which is an output of the MultiABLER extraction method'
- Only a single omics layer is present and no joint metabolite/lipid downstream analysis
  is planned, so the integrative MultiABLER pipeline adds little value
edam_operation: http://edamontology.org/operation_3660
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0154
tools:
- name: MultiABLER
  role: R pipeline providing tbManipulate (filter/impute/log-transform), normEigenMS,
    and runLimma for unified metabolomic/lipidomic processing
  repo: https://github.com/holab-hku/MultiABLER
- name: EigenMS
  role: Normalization method applied via normEigenMS; identified in the article as
    best suited for MultiABLER data
- name: limma
  role: Linear-model engine inside runLimma used to test genotype contrasts (WT vs
    Apoe−/−) for DE feature counts
  repo: https://www.bioconductor.org/packages/release/bioc/html/limma.html
- name: ProteoMM
  role: Dependency installed to support MultiABLER functions
  repo: https://www.bioconductor.org/packages/release/bioc/html/ProteoMM.html
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.isci.2023.106881
    title: Integrative processing of untargeted metabolomic and lipidomic data using
      MultiABLER
  source_content_hash: sha256:882712cc7a3c57ff2d6eb131858feca8d912f7afeab093ed05e128f30bb737a1
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
  - limma
  edam_operation: http://edamontology.org/operation_3660
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# multi-omics-de-count-validation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Runs the MultiABLER pipeline (filter, impute, EigenMS-normalize, limma) on Multi-ABLE-derived metabolomic and lipidomic feature tables and validates the resulting differential-expression counts, as benchmarked on the Apoe+/+ vs Apoe−/− arterial-tissue comparison.

## When to use

Use when processing concurrently generated untargeted metabolomic and lipidomic LC-MS/MS feature tables (e.g. from the Multi-ABLE method) through filtering, imputation, EigenMS normalization, and limma differential expression, then verifying reported DE feature counts at adjusted p < 0.05.

## When NOT to use

- Input is already a filtered, normalized feature table — tbManipulate and EigenMS reprocessing would distort the data
- The data are proteomic: MultiABLER does not yet provide functions to analyze proteomic data, which is an output of the MultiABLER extraction method
- Only a single omics layer is present and no joint metabolite/lipid downstream analysis is planned, so the integrative MultiABLER pipeline adds little value

## Inputs

- Raw metabolite feature table (e.g. from S-BSST1039, Zenodo record 8039588) loaded into an R data frame
- Raw lipid feature table from the same Multi-ABLE run
- Sample metadata / sample info mapping (e.g. genotype: Apoe+/+ vs Apoe−/−) used for EigenMS normalization and the limma design matrix
- Filter threshold, imputation setting, and log base for tbManipulate

## Outputs

- Per-feature differential-expression results for the lipid layer (903/1650 DE at adjusted p < 0.05; 414 with logFC > 1)
- Per-feature differential-expression results for the metabolite layer (601/1314 DE at adjusted p < 0.05; 334 with logFC > 1)
- Combined DE results table saved as DE_results.csv

## How to apply

Load the raw metabolite and lipid feature tables (e.g. S-BSST1039 arterial tissue from 9 Apoe+/+ and 11 Apoe−/− mice on Zenodo record 8039588) into R data frames, and process each with tbManipulate(df, filter_threshold, impute, log_base) to remove low-quality features, impute missing values, and log-transform abundances. Normalize each processed table with normEigenMS(df, sample.info) using EigenMS, which the article identifies as the best suited normalization method for MultiABLER. Build a design matrix encoding the genotype contrast (WT vs Apoe−/−) together with sample metadata, and fit a limma linear model per omics layer with runLimma(df, design.matrix, cont.matrix). Extract DE features at adjusted p < 0.05 (and optionally logFC > 1) and validate against the benchmark: 903 of 1650 lipids and 601 of 1314 metabolites DE (414 lipids and 334 metabolites with logFC > 1). Save the combined per-feature results as DE_results.csv.

## Related tools

- **MultiABLER** (R pipeline providing tbManipulate (filter/impute/log-transform), normEigenMS, and runLimma for unified metabolomic/lipidomic processing) — https://github.com/holab-hku/MultiABLER
- **EigenMS** (Normalization method applied via normEigenMS; identified in the article as best suited for MultiABLER data)
- **limma** (Linear-model engine inside runLimma used to test genotype contrasts (WT vs Apoe−/−) for DE feature counts) — https://www.bioconductor.org/packages/release/bioc/html/limma.html
- **ProteoMM** (Dependency installed to support MultiABLER functions) — https://www.bioconductor.org/packages/release/bioc/html/ProteoMM.html

## Examples

```
devtools::install_github("holab-hku/MultiABLER", dependencies = TRUE)
```

## Evaluation signals

- Lipid layer yields 903 DE features of 1650 total at adjusted p < 0.05 after limma
- Metabolite layer yields 601 DE features of 1314 total at adjusted p < 0.05 after limma
- Applying an additional logFC > 1 cutoff reproduces 414 DE lipids and 334 DE metabolites
- Normalized feature tables remain suitable for downstream differential expression, sPLS, and rCCA
- The combined results file DE_results.csv is produced and contains per-feature statistics for both omics layers

## Limitations

- MultiABLER does not implement kNN imputation, mean normalization, or quantile normalization — only its built-in manipulation and EigenMS normalization options are available
- MultiABLER does not provide functions to analyze proteomic data; proteomic integration is planned for future development
- DE counts are benchmark-specific and will vary with filter threshold, imputation setting, log base, and the chosen design/contrast matrix

## Evidence
