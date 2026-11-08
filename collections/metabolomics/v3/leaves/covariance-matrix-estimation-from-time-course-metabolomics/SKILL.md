---
name: covariance-matrix-estimation-from-time-course-metabolomics
description: Use when metabolomic abundance data have been split into condition subsets
  (e.g. 6C stress vs 16C control) and a covariance matrix is needed as input to calculate_jacobian
  for inferring metabolite interaction networks.
when_to_use_negative:
- Input is raw unprocessed metabolomic data that has not been subset by condition
  via prepare_data
- The analysis goal is statistical testing or pathway enrichment rather than Jacobian-based
  network inference, where a covariance matrix is not the required input
- Multi-time-point data where the num_tp parameter semantics have not been matched
  to the experimental design
edam_operation: http://edamontology.org/operation_3465
edam_topics:
- http://edamontology.org/topic_3314
- http://edamontology.org/topic_3520
tools:
- name: MInfer
  role: Provides generate_covariance (and prepare_data) to compute covariance matrices
    from prepared metabolomic data as input to Jacobian calculation
  repo: https://github.com/cellbiomaths/MInfer
- name: R
  role: Runtime environment in which the MInfer package functions are executed
provenance:
  source_task_ids:
  - task_004
  - task_001
  source_papers:
  - doi: 10.1016/j.cmpb.2025.108672
    title: 'MInfer: Bridging MetaboAnalyst and Jacobian analysis for metabolomic networks'
  source_content_hash: sha256:81dfd68b17c02bfeac01ff9dbafb3c79ea5b30cd52ffa618d1eb1c4b207afc14
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
  - MInfer
  - MetaboAnalyst
  - R
  - devtools
  edam_operation: http://edamontology.org/operation_3465
  edam_topics:
  - http://edamontology.org/topic_3314
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# covariance-matrix-estimation-from-time-course-metabolomics

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Generate condition-specific covariance matrices from prepared metabolomic data using MInfer's generate_covariance, a prerequisite step for Jacobian-based metabolite interaction network (MIN) inference in R.

## When to use

Use when metabolomic abundance data have been split into condition subsets (e.g. 6C stress vs 16C control) and a covariance matrix is needed as input to calculate_jacobian for inferring metabolite interaction networks.

## When NOT to use

- Input is raw unprocessed metabolomic data that has not been subset by condition via prepare_data
- The analysis goal is statistical testing or pathway enrichment rather than Jacobian-based network inference, where a covariance matrix is not the required input
- Multi-time-point data where the num_tp parameter semantics have not been matched to the experimental design

## Inputs

- Prepared condition-specific metabolomic data object returned by MInfer prepare_data (e.g. data_6C, data_16C from met_input)
- num_tp parameter value specifying the number of time points (e.g. num_tp = 1)

## Outputs

- Condition-specific covariance matrix (list element cov_6C[[1]], cov_16C[[1]]) ready for calculate_jacobian

## How to apply

First prepare the data with MInfer's prepare_data(met_input, condition) to select the condition of interest, e.g. condition 6 (6C stress) or condition 16 (16C control) from the Cífková low-temperature benchmark or the CML dataset. Then call generate_covariance(data, num_tp = 1), where num_tp specifies the number of time points; the returned object is a list whose first element (cov[[1]]) is the covariance matrix passed directly to calculate_jacobian(cov, interactions_fin, icount = 15). Generate one covariance matrix per condition so that condition-specific Jacobian matrices can later be compared element-wise to compute differential Jacobian ratios. The rationale is that covariance structure across metabolites is the empirical anchor from which MInfer derives the dynamic interaction (Jacobian) matrix bridging MetaboAnalyst-style processed data to network modeling.

## Related tools

- **MInfer** (Provides generate_covariance (and prepare_data) to compute covariance matrices from prepared metabolomic data as input to Jacobian calculation) — https://github.com/cellbiomaths/MInfer
- **R** (Runtime environment in which the MInfer package functions are executed)

## Examples

```
cov_6C <- generate_covariance(data_6C, num_tp = 1); cov_16C <- generate_covariance(data_16C, num_tp = 1)
```

## Evaluation signals

- The covariance matrix is a square symmetric matrix whose dimensions match the metabolite count of the prepared condition subset
- One covariance matrix is produced per condition (e.g. cov_6C and cov_16C both generated with identical num_tp settings) to enable downstream differential comparison
- The returned object is a list whose first element [[1]] is accepted directly by calculate_jacobian without re-shaping
- Downstream calculate_jacobian(cov[[1]], interactions_fin, icount = 15) executes without error and yields a Jacobian whose heatmap visualizes condition-specific structure

## Limitations

- num_tp must correctly encode the number of time points; the README only demonstrates num_tp = 1, and behavior for multi-time-point designs is not illustrated
- Covariance matrices are only meaningful per condition subset; mixing conditions before estimation would confound the differential Jacobian analysis
- The covariance alone does not encode interactions — calculate_jacobian additionally requires the metabolite interaction network (interactions_fin) and icount parameter

## Evidence

- [methods] Generate covariance matrices for the prepared data. The parameter num_tp specifies the number of time points.: "Generate covariance matrices for the prepared data. The parameter num_tp specifies the number of time points."
- [readme] Generate covariance matrices: "# Generate covariance matrices
cov_6C <- generate_covariance(data_6C, num_tp = 1)
cov_16C <- generate_covariance(data_16C, num_tp = 1)"
- [intro] It provides tools for data preparation, covariance matrix generation, Jacobian matrix computation, and visualization of metabolite interaction networks.: "It provides tools for data preparation, covariance matrix generation, Jacobian matrix computation, and visualization of metabolite interaction networks."
- [methods] Prepare the data by selecting specific conditions: "Prepare the data by selecting specific conditions (e.g., 6C and 16C)"
- [readme] Calculate Jacobian matrices: "jacobian_6C <- calculate_jacobian(cov_6C[[1]], interactions_fin, icount = 15)"
