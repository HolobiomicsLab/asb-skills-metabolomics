---
name: jacobian-matrix-network-inference
description: Use when raw metabolomic data or MetaboAnalyst-derived metabolite inputs
  need to be converted into a network of dynamic metabolite interactions via Jacobian
  matrix computation, with topology metrics.
when_to_use_negative:
- When the goal is generic metabolomics QC or peak annotation rather than network-level
  interaction inference — MInfer is designed to derive a dynamic MIN, not a standard
  feature table.
- When input data is already a finalized interaction network with no need for Jacobian
  re-derivation.
- When there is only a single metabolite/timepoint with insufficient variation to
  generate a meaningful covariance matrix.
edam_operation: http://edamontology.org/operation_0276
edam_topics:
- http://edamontology.org/topic_3524
- http://edamontology.org/topic_0602
tools:
- name: MInfer
  role: R package that provides data preparation, covariance matrix generation, Jacobian
    matrix computation, and visualization of the metabolite interaction network from
    metabolomic inputs.
  repo: https://github.com/cellbiomaths/MInfer
- name: MetaboAnalyst
  role: Upstream metabolomic analysis tool whose outputs MInfer integrates with Jacobian
    analysis to construct the interaction network.
- name: devtools
  role: R helper package used to install MInfer directly from GitHub.
- name: R
  role: Runtime environment in which the MInfer package and its functions (prepare_data,
    generate_covariance, calculate_jacobian, visualize_heatmap) are executed.
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.cmpb.2025.108672
    title: 'MInfer: Bridging MetaboAnalyst and Jacobian analysis for metabolomic networks'
  source_content_hash: sha256:b743e2628a8ef854a8dd16ded42aa167d5d4dcb1d1a158c201d8d6ed9cf9b73c
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
  edam_operation: http://edamontology.org/operation_0276
  edam_topics:
  - http://edamontology.org/topic_3524
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 14
---

# jacobian-matrix-network-inference

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Infers a dynamic metabolite interaction network (MIN) from metabolomics data by computing the Jacobian matrix that encodes interactions between metabolites, then deriving topology statistics (nodes, edges, average neighbors, path length, clustering, density). Implemented via the MInfer R package, which bridges MetaboAnalyst outputs to Jacobian analysis.

## When to use

Use when raw metabolomic data or MetaboAnalyst-derived metabolite inputs need to be converted into a network of dynamic metabolite interactions via Jacobian matrix computation, with topology metrics.

## When NOT to use

- When the goal is generic metabolomics QC or peak annotation rather than network-level interaction inference — MInfer is designed to derive a dynamic MIN, not a standard feature table.
- When input data is already a finalized interaction network with no need for Jacobian re-derivation.
- When there is only a single metabolite/timepoint with insufficient variation to generate a meaningful covariance matrix.

## Inputs

- Metabolomic input data (e.g. CML dataset met_input) with rows/columns of metabolite IDs (KEGG IDs)
- Condition identifier to select from the input data (e.g. 6C, 16C)
- Number of time points (num_tp)
- Interaction count parameter (icount) and finalized interaction matrix (interactions_fin)

## Outputs

- Covariance matrix for the selected condition(s)
- Jacobian matrix J encoding metabolite interactions (jacobian$J)
- Heatmap of the Jacobian matrix (e.g. jacobian_heatmap.png)
- Metabolite interaction network (MIN) with topology statistics: node count, edge count, average neighbors, characteristic path length, clustering coefficient, network density

## How to apply

Install MInfer from GitHub with devtools::install_github and load the CML metabolomic input (met_input). Select the condition(s) of interest with prepare_data(met_input, condition), then generate a covariance matrix for the prepared data using generate_covariance(data, num_tp = 1). Compute the Jacobian matrix encoding metabolite interactions via calculate_jacobian(cov[[1]], interactions_fin, icount = 15). Build the metabolite interaction network (MIN) from the Jacobian matrix and compute global topology statistics: node count, edge count, average neighbors, characteristic path length, clustering coefficient, and network density. Visualize the Jacobian matrix as a heatmap with visualize_heatmap(jacobian$J, title = ...).

## Related tools

- **MInfer** (R package that provides data preparation, covariance matrix generation, Jacobian matrix computation, and visualization of the metabolite interaction network from metabolomic inputs.) — https://github.com/cellbiomaths/MInfer
- **MetaboAnalyst** (Upstream metabolomic analysis tool whose outputs MInfer integrates with Jacobian analysis to construct the interaction network.)
- **devtools** (R helper package used to install MInfer directly from GitHub.)
- **R** (Runtime environment in which the MInfer package and its functions (prepare_data, generate_covariance, calculate_jacobian, visualize_heatmap) are executed.)

## Examples

```
devtools::install_github("https://github.com/cellbiomaths/MInfer"); library(MInfer); data_6C <- prepare_data(met_input, 6); cov_6C <- generate_covariance(data_6C, num_tp = 1); jacobian_6C <- calculate_jacobian(cov_6C[[1]], interactions_fin, icount = 15); visualize_heatmap(jacobian_6C$J, title = "Jacobian Matrix - 6C")
```

## Evaluation signals

- Verify the recomputed CML network statistics match the published values — 44 nodes, 52 edges, average neighbors 2.3, characteristic path length 3.7, clustering coefficient 0.133, and density 0.084 — within rounding tolerance.
- Check that generate_covariance returns a valid covariance matrix for each condition (cov_6C, cov_16C) before calculating the Jacobian.
- Confirm the Jacobian matrix J (jacobian$J) is symmetric and populated with the expected number of metabolite interactions (icount = 15).
- Verify that the exported heatmap of the Jacobian matrix is produced and interpretable (jacobian_heatmap.png).
- Confirm the derived MIN has nonzero clustering coefficient and network density consistent with a connected metabolite interaction graph.

## Limitations

- The method depends on user-supplied interaction count and finalized interaction matrix parameters (icount, interactions_fin), which may require manual curation.
- Topology metrics such as characteristic path length and clustering coefficient depend on the number of time points (num_tp) and selected condition, so results vary with data conditioning.
- The Jacobian heatmap and 3D visualization are exploratory; interpretation of individual metabolite interactions still requires biological context.
- MInfer is tightly coupled to MetaboAnalyst-style inputs (KEGG-style metabolite IDs in rows/columns), so data must be pre-formatted accordingly.

## Evidence

- [results] The CML network is relatively modest, comprising 44 metabolite nodes and 52 interaction edges, with each metabolite interacting on average with 2.3 other metabolites.: [evidence span withheld — anchor status: not_found]
