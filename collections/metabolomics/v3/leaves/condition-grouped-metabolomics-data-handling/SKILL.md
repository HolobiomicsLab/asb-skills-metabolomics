---
name: condition-grouped-metabolomics-data-handling
description: Use when a metabolite intensity matrix must be organized by condition
  group and converted into per-condition covariance matrices as the numeric substrate
  for Jacobian network inference in MInfer.
when_to_use_negative:
- Input is raw or unnormalized metabolomics data that has not passed MetaboAnalyst/MInfer
  data preparation — the covariance step expects a condition-prepared matrix, not
  raw intensities.
- The analysis goal is direct heatmap/3D visualization of interaction networks without
  Jacobian computation — then visualization functions on existing Jacobian output
  suffice.
- Metabolite identifiers are not KEGG-compatible IDs consistent with rows and columns
  of the dataset, so calculate_jacobian cannot consume the covariance output.
tools:
- name: MInfer
  role: Provides prepare_data, generate_covariance, and calculate_jacobian used to
    group data by condition, compute covariance matrices, and hand them to Jacobian
    inference
  repo: https://github.com/cellbiomaths/MInfer
- name: R
  role: Runtime environment in which the MInfer package workflow (prepare_data ->
    generate_covariance -> calculate_jacobian) is executed
- name: MASS
  role: Declared Imports dependency of MInfer available in the environment for the
    package's computations
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.cmpb.2025.108672
    title: 'MInfer: Bridging MetaboAnalyst and Jacobian analysis for metabolomic networks'
  source_content_hash: sha256:ce1723e06fce171273ad6a639eb536110d7cd4eee58b67d2922e59b52d0526da
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/cellbiomaths/MInfer
  repo_url: https://github.com/cellbiomaths/MInfer
  tools:
  - MASS
  - MInfer
  - R
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# condition-grouped-metabolomics-data-handling

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Reconstructs MInfer's covariance-matrix step: metabolite intensities are first split into condition groups via prepare_data (e.g., 6C vs 16C), then each group's prepared object is passed to generate_covariance with a specified num_tp, yielding per-condition covariance matrices that feed Jacobian-based network inference.

## When to use

Use when a metabolite intensity matrix must be organized by condition group and converted into per-condition covariance matrices as the numeric substrate for Jacobian network inference in MInfer.

## When NOT to use

- Input is raw or unnormalized metabolomics data that has not passed MetaboAnalyst/MInfer data preparation — the covariance step expects a condition-prepared matrix, not raw intensities.
- The analysis goal is direct heatmap/3D visualization of interaction networks without Jacobian computation — then visualization functions on existing Jacobian output suffice.
- Metabolite identifiers are not KEGG-compatible IDs consistent with rows and columns of the dataset, so calculate_jacobian cannot consume the covariance output.

## Inputs

- met_input: metabolite intensity data frame with metabolite IDs in rows/columns (e.g., MetaboAnalyst-processed data or MInfer's example_data)
- condition selector (e.g., 6 or 16 for 6C/16C) passed to prepare_data
- num_tp: number of time points (documented example uses num_tp = 1)

## Outputs

- Per-condition prepared data object (data_6C, data_16C)
- List of covariance matrices from generate_covariance; the first element cov_6C[[1]] is the condition-specific square covariance matrix
- covariance_matrix.csv: square, symmetric matrix with metabolite names as row/column identifiers

## How to apply

Load MInfer's packaged example_data (or your own MetaboAnalyst-processed metabolite table) and call prepare_data(met_input, condition) once per condition to produce a condition-structured object such as data_6C. For each prepared object, call generate_covariance(data, num_tp) — in the documented example num_tp = 1, i.e., a single time point — which returns a list of covariance matrices. Extract the desired matrix from the first list element (cov_6C[[1]]), preserving metabolite identifiers as row/column names, and write it out (e.g., covariance_matrix.csv). The rationale is that covariance between metabolites within a condition group is the statistical signal MInfer's Jacobian analysis consumes downstream via calculate_jacobian(cov, stoi, icount), so correct condition grouping and time-point counting directly determine network validity.

## Related tools

- **MInfer** (Provides prepare_data, generate_covariance, and calculate_jacobian used to group data by condition, compute covariance matrices, and hand them to Jacobian inference) — https://github.com/cellbiomaths/MInfer
- **R** (Runtime environment in which the MInfer package workflow (prepare_data -> generate_covariance -> calculate_jacobian) is executed)
- **MASS** (Declared Imports dependency of MInfer available in the environment for the package's computations)

## Examples

```
data_6C <- prepare_data(met_input, 6); cov_6C <- generate_covariance(data_6C, num_tp = 1); write.csv(cov_6C[[1]], "covariance_matrix.csv")
```

## Evaluation signals

- covariance_matrix.csv is square and symmetric, with row/column identifiers matching the metabolites of the prepared condition data
- The matrix loads cleanly as the cov argument of calculate_jacobian (e.g., calculate_jacobian(cov_6C[[1]], interactions_fin, icount = 15)) without dimension errors
- Dimensions of cov_6C[[1]] equal the number of metabolites in the prepare_data output for that condition
- num_tp matches the actual number of time points in the condition-prepared data (documented example: num_tp = 1)

## Limitations

- The documented example uses a single time point (num_tp = 1); behavior with multiple time points is not illustrated in the article or README.
- Covariance is computed per condition group, so incorrect condition selection in prepare_data propagates to a biologically invalid covariance matrix and downstream Jacobian network.
- The covariance output alone is not a final result — it must be paired with an interactions matrix (interactions_fin) and an interaction count (icount = 15 in the example) for calculate_jacobian.

## Evidence

- [methods] Generate covariance matrices for the prepared data. The parameter num_tp specifies the number of time points.: "Generate covariance matrices for the prepared data. The parameter num_tp specifies the number of time points."
- [readme] Prepare data for two conditions / prepare_data call: "data_6C <- prepare_data(met_input, 6)"
- [readme] Jacobian step consuming cov_6C[[1]]: "jacobian_6C <- calculate_jacobian(cov_6C[[1]], interactions_fin, icount = 15)"
- [intro] MInfer toolchain scope: "It provides tools for data preparation, covariance matrix generation, Jacobian matrix computation, and visualization of metabolite interaction networks."
