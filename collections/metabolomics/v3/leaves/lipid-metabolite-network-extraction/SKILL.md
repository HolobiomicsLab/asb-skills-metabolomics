---
name: lipid-metabolite-network-extraction
description: Use when you have a MultiABLER-normalized feature table from concurrently
  acquired LC-MS/MS metabolomics and lipidomics data and need to identify correlated
  lipid-metabolite pairs, network structure, or the effect of normalization on cross-omics
  correlation via rCCA/sPLS.
when_to_use_negative:
- MultiABLER currently does not provide functions to analyze proteomic data, so do
  not extend this lipid-metabolite network workflow to proteomics blocks; proteomic
  integration is only planned for future development
- Input is not a jointly acquired metabolomics/lipidomics LC-MS/MS dataset processed
  through MultiABLER — the paper only validates the feature table for sPLS and rCCA
  in that setting
edam_operation: http://edamontology.org/operation_3744
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0153
tools:
- name: MultiABLER
  role: R pipeline that produces the normalized feature table (using EigenMS normalization
    via tbManipulate/normEigenMS) on which sPLS and rCCA are run; installed via devtools::install_github
  repo: https://github.com/holab-hku/MultiABLER
- name: mixOmics
  role: R package used to run regularized Canonical Correlation Analysis (rCCA) and
    sparse Partial Least Squares (sPLS) on the metabolomics and lipidomics blocks
- name: R
  role: Runtime environment in which MultiABLER and mixOmics analyses are executed
- name: EigenMS
  role: Normalization method identified as best suited for MultiABLER, applied via
    tbManipulate/normEigenMS before network extraction
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1016/j.isci.2023.106881
    title: Integrative processing of untargeted metabolomic and lipidomic data using
      MultiABLER
  source_content_hash: sha256:995d78a55c02feecfaba4127cf2edccbd430b627bed360520f939ba0701a1ff1
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
  - R
  - mixOmics
  edam_operation: http://edamontology.org/operation_3744
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0153
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# lipid-metabolite-network-extraction

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Extracts correlated lipid-metabolite networks by running sparse Partial Least Squares (sPLS) and regularized Canonical Correlation Analysis (rCCA) from mixOmics on a MultiABLER-normalized joint feature table, and quantifies how normalization (e.g. via tbManipulate/normEigenMS) changes the lipidomics-metabolomics correlation.

## When to use

Use when you have a MultiABLER-normalized feature table from concurrently acquired LC-MS/MS metabolomics and lipidomics data and need to identify correlated lipid-metabolite pairs, network structure, or the effect of normalization on cross-omics correlation via rCCA/sPLS.

## When NOT to use

- MultiABLER currently does not provide functions to analyze proteomic data, so do not extend this lipid-metabolite network workflow to proteomics blocks; proteomic integration is only planned for future development
- Input is not a jointly acquired metabolomics/lipidomics LC-MS/MS dataset processed through MultiABLER — the paper only validates the feature table for sPLS and rCCA in that setting

## Inputs

- MultiABLER-normalized joint feature table from LC-MS/MS metabolomics and lipidomics (post tbManipulate/normEigenMS processing)
- Pre-normalization (raw/alternative normalization) feature table for the pre/post-normalization rCCA comparison
- Block definitions assigning features to metabolomics vs lipidomics data blocks

## Outputs

- correlation_results.csv containing correlation coefficients, sPLS network memberships, and pre/post-normalization rCCA comparison
- Two distinct lipid-metabolite networks identified by sPLS
- Correlated lipid-metabolite feature pairs with their correlation coefficients

## How to apply

Load the normalized arterial (or analogous) feature table produced by the MultiABLER pipeline after tbManipulate/normEigenMS processing, and split it into metabolomics and lipidomics blocks for multi-omics integration. Run regularized Canonical Correlation Analysis (rCCA) via mixOmics to estimate the correlation between the two blocks, running it both before and after MultiABLER normalization so the increase in correlation attributable to normalization can be quantified. Then run sparse Partial Least Squares (sPLS) via mixOmics on the normalized feature table to identify correlated lipid-metabolite pairs and derive the network structure; the MultiABLER paper reports that its feature table is suitable for differential expression analysis, sPLS, and rCCA. Extract the two distinct lipid-metabolite networks identified by sPLS, record the correlated feature pairs with their correlation coefficients, and assemble correlation coefficients, network memberships, and the pre/post-normalization comparison into a single results table written as correlation_results.csv.

## Related tools

- **MultiABLER** (R pipeline that produces the normalized feature table (using EigenMS normalization via tbManipulate/normEigenMS) on which sPLS and rCCA are run; installed via devtools::install_github) — https://github.com/holab-hku/MultiABLER
- **mixOmics** (R package used to run regularized Canonical Correlation Analysis (rCCA) and sparse Partial Least Squares (sPLS) on the metabolomics and lipidomics blocks)
- **R** (Runtime environment in which MultiABLER and mixOmics analyses are executed)
- **EigenMS** (Normalization method identified as best suited for MultiABLER, applied via tbManipulate/normEigenMS before network extraction)

## Examples

```
devtools::install_github("holab-hku/MultiABLER", dependencies = TRUE)
```

## Evaluation signals

- rCCA run on post-MultiABLER-normalization blocks yields a higher lipidomics-metabolomics correlation than the pre-normalization run; the increase is the recorded finding
- sPLS on the normalized feature table resolves two distinct lipid-metabolite networks, and every extracted pair carries a correlation coefficient
- correlation_results.csv contains correlation coefficients, network memberships, and the pre/post-normalization comparison in a single table
- The input feature table must be the MultiABLER output after tbManipulate/normEigenMS processing; running sPLS/rCCA on the raw table should reproduce the (lower) pre-normalization baseline

## Limitations

- MultiABLER currently does not provide functions to analyze proteomic data, which is another important output from the MultiABLER extraction method; integration for proteomic analysis is planned for future development
- Other data manipulation and normalization methods, such as k-nearest neighbours (kNN) imputation, mean normalization, and quantile normalization, were not implemented in MultiABLER, limiting alternative normalization baselines for the pre/post comparison

## Evidence

- [results] rCCA analysis demonstrated that running the MultiABLER normalization increased the correlation between the lipidomics and metabolomics data from mouse arterial tissue.: [evidence span withheld — anchor status: not_found]
