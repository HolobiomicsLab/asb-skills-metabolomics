---
name: covariance-matrix-io-for-network-inference
description: Use when metabolite data has already been condition-prepared with MInfer's
  prepare_data and a per-condition covariance matrix must be generated and exported
  as the cov argument for calculate_jacobian.
when_to_use_negative:
- The input is raw metabolite data (met_input) that has not yet been condition-grouped
  by prepare_data — generate_covariance consumes the prepared object, not raw intensities.
- The goal is the interaction network itself — once the covariance matrix exists,
  proceed to calculate_jacobian rather than re-running covariance generation.
- Data spans multiple conditions (e.g. 6C and 16C) — each condition requires its own
  generate_covariance call (cov_6C and cov_16C); do not pool conditions into a single
  call.
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_0602
tools:
- name: MInfer
  role: R package providing generate_covariance(), which computes the per-condition
    covariance matrix from prepared metabolite data, and calculate_jacobian(), which
    consumes the exported matrix as its cov argument
  repo: https://github.com/cellbiomaths/MInfer
- name: R
  role: Runtime language in which MInfer's generate_covariance is invoked and from
    which the matrix is written to covariance_matrix.csv
- name: MASS
  role: Declared dependency of MInfer (Imports) that must be available when running
    generate_covariance
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.cmpb.2025.108672
    title: 'MInfer: Bridging MetaboAnalyst and Jacobian analysis for metabolomic networks'
  source_content_hash: sha256:ff628ce2a0b85a2e3950c560b9374249f5cdbed7cb21a636e67c5b27ca6e4483
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
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# covariance-matrix-io-for-network-inference

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Computes and exports the per-condition covariance matrix in MInfer's workflow: generate_covariance turns a condition-prepared metabolite matrix (e.g. data_6C with num_tp = 1) into the square, symmetric covariance matrix that serves as the cov input to calculate_jacobian for Jacobian-based metabolite network inference.

## When to use

Use when metabolite data has already been condition-prepared with MInfer's prepare_data and a per-condition covariance matrix must be generated and exported as the cov argument for calculate_jacobian.

## When NOT to use

- The input is raw metabolite data (met_input) that has not yet been condition-grouped by prepare_data — generate_covariance consumes the prepared object, not raw intensities.
- The goal is the interaction network itself — once the covariance matrix exists, proceed to calculate_jacobian rather than re-running covariance generation.
- Data spans multiple conditions (e.g. 6C and 16C) — each condition requires its own generate_covariance call (cov_6C and cov_16C); do not pool conditions into a single call.

## Inputs

- Condition-prepared metabolite data object from MInfer's prepare_data (e.g. data_6C <- prepare_data(met_input, 6))
- num_tp parameter: number of time points in the experiment (example value: 1)
- R environment with MInfer installed from github.com/cellbiomaths/MInfer (Imports: MASS)

## Outputs

- List returned by generate_covariance whose first element (cov[[1]]) is the square, symmetric covariance matrix for the condition
- covariance_matrix.csv containing the covariance matrix with metabolite names preserved as row/column identifiers
- Matrix validated to load cleanly as the cov argument of MInfer's calculate_jacobian(cov, stoi, icount)

## How to apply

Run this inside the MInfer R package (dependency: MASS) after data preparation: pass the condition-grouped metabolite object produced by prepare_data (e.g. data_6C <- prepare_data(met_input, 6) for condition 6C) into cov_6C <- generate_covariance(data_6C, num_tp = 1), setting num_tp to the number of time points in the experiment (the example uses 1). generate_covariance returns a list, so extract the covariance matrix from its first element (cov_6C[[1]]). Persist the matrix to covariance_matrix.csv, preserving metabolite names as row and column identifiers, because this file is the cov argument consumed downstream by calculate_jacobian(cov, stoi, icount) (icount = 15 in the example, with interactions_fin supplying the interactions). Before export, verify the matrix is square and symmetric and its identifiers match the metabolites of the prepared condition data — a mismatch breaks the Jacobian computation. The rationale is that the covariance matrix, not the raw metabolite intensities, encodes the inter-metabolite relationships that Jacobian analysis exploits to infer the dynamic interaction network.

## Related tools

- **MInfer** (R package providing generate_covariance(), which computes the per-condition covariance matrix from prepared metabolite data, and calculate_jacobian(), which consumes the exported matrix as its cov argument) — https://github.com/cellbiomaths/MInfer
- **R** (Runtime language in which MInfer's generate_covariance is invoked and from which the matrix is written to covariance_matrix.csv)
- **MASS** (Declared dependency of MInfer (Imports) that must be available when running generate_covariance)

## Examples

```
cov_6C <- generate_covariance(data_6C, num_tp = 1); write.csv(cov_6C[[1]], "covariance_matrix.csv")
```

## Evaluation signals

- The emitted covariance_matrix.csv is square and symmetric (equal to its transpose).
- Row and column identifiers of the matrix match the metabolites of the prepared condition data (e.g. data_6C).
- Matrix dimensions equal the number of metabolites in the prepared condition input (n × n).
- The matrix was extracted from the first element of generate_covariance's returned list (cov_6C[[1]]), matching the documented usage.
- The exported matrix loads and runs cleanly as the cov argument of calculate_jacobian(cov, stoi, icount) with icount = 15 without dimension or identifier errors.

## Limitations

- The covariance matrix is condition-specific: 6C and 16C each require a separate generate_covariance call, as shown in the README example.
- num_tp must match the experiment's number of time points; the documented example uses num_tp = 1.
- The covariance matrix alone does not infer interactions — it is only the upstream numeric input to calculate_jacobian, which also needs interactions_fin and icount.

## Evidence

- [intro] MInfer toolchain overview: "It provides tools for data preparation, covariance matrix generation, Jacobian matrix computation, and visualization of metabolite interaction networks."
- [methods] generate_covariance invocation: "cov_6C <- generate_covariance(data_6C, num_tp = 1)"
- [methods] covariance consumed by calculate_jacobian: "jacobian_6C <- calculate_jacobian(cov_6C[[1]], interactions_fin, icount = 15)"
- [readme] num_tp parameter semantics: "Generate covariance matrices for the prepared data. The parameter num_tp specifies the number of time points."
