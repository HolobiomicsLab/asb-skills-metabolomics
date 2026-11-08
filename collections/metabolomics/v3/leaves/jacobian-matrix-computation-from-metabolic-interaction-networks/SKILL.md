---
name: jacobian-matrix-computation-from-metabolic-interaction-networks
description: Use when a metabolomics study has preprocessed concentration data, a
  defined interaction network, and the goal is to quantify metabolite interactions
  via Jacobian matrices for specific conditions (e.g., 6C vs 16C).
when_to_use_negative:
- When only raw metabolite abundance data is available without an interaction network
  or MetaboAnalyst-derived MIN (Jacobian computation requires `interactions_fin`)
- When the study objective is simple statistical differential analysis rather than
  dynamic interaction modeling (MInfer focuses on Jacobian-based interaction inference)
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3315
- http://edamontology.org/topic_3170
tools:
- name: MInfer
  role: R package providing `calculate_jacobian` to compute Jacobian matrices from
    covariance matrices and interaction networks
  repo: https://github.com/cellbiomaths/MInfer
- name: MetaboAnalyst
  role: Source of metabolomic data processing outputs and interaction network inputs
    (MetaboAnalystMINs) consumed by Jacobian analysis
- name: devtools
  role: Used to install MInfer from GitHub via `devtools::install_github`
  repo: https://github.com/cellbiomaths/MInfer
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.cmpb.2025.108672
    title: 'MInfer: Bridging MetaboAnalyst and Jacobian analysis for metabolomic networks'
  source_content_hash: sha256:0f897eb8a39d3743c0f59544c283d72a50697801c7b488dc918bc0d68bdad8b4
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
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3315
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 16
---

# jacobian-matrix-computation-from-metabolic-interaction-networks

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Computes condition-specific Jacobian matrices from covariance matrices and a metabolite interaction network (MIN) using the MInfer R package, enabling dynamic interaction analysis of metabolomics data.

## When to use

Use when a metabolomics study has preprocessed concentration data, a defined interaction network, and the goal is to quantify metabolite interactions via Jacobian matrices for specific conditions (e.g., 6C vs 16C).

## When NOT to use

- When only raw metabolite abundance data is available without an interaction network or MetaboAnalyst-derived MIN (Jacobian computation requires `interactions_fin`)
- When the study objective is simple statistical differential analysis rather than dynamic interaction modeling (MInfer focuses on Jacobian-based interaction inference)

## Inputs

- Prepared metabolite concentration data for a condition (from `prepare_data`, e.g., `data_6C`)
- Covariance matrix generated via `generate_covariance(data, num_tp = 1)`
- Finalized metabolite interaction network (e.g., `interactions_fin`) derived from MetaboAnalyst or KEGG metabolite IDs

## Outputs

- Jacobian matrix object (e.g., `jacobian_6C` with element `$J`) representing metabolite interactions for that condition
- Condition-specific Jacobian matrices suitable for differential comparison (e.g., 6C vs 16C)
- Visualizations: heatmap via `visualize_heatmap(jacobian$J)` or 3D plot via `visualize_3d(jacobian$J)`

## How to apply

First, prepare data for each condition using MInfer's `prepare_data` (e.g., `prepare_data(met_input, 6)` for 6C stress). Next, generate condition-specific covariance matrices with `generate_covariance(data, num_tp = 1)` where `num_tp` sets the number of time points. Then compute the Jacobian matrix by calling `calculate_jacobian(cov, interactions_fin, icount = 15)`, where `interactions_fin` is the metabolite interaction network (derived from MetaboAnalyst outputs, e.g., KEGG IDs like 'C00042') and `icount = 15` controls interaction count. Finally, visualize the Jacobian (e.g., `visualize_heatmap(jacobian_6C$J)`) or use 3D plots. The output is a list containing the Jacobian matrix `$J` that can be compared across conditions for differential analysis.

## Related tools

- **MInfer** (R package providing `calculate_jacobian` to compute Jacobian matrices from covariance matrices and interaction networks) — https://github.com/cellbiomaths/MInfer
- **MetaboAnalyst** (Source of metabolomic data processing outputs and interaction network inputs (MetaboAnalystMINs) consumed by Jacobian analysis)
- **devtools** (Used to install MInfer from GitHub via `devtools::install_github`) — https://github.com/cellbiomaths/MInfer

## Examples

```
jacobian_6C <- calculate_jacobian(cov_6C[[1]], interactions_fin, icount = 15)
```

## Evaluation signals

- The returned object must contain a matrix accessible as `$J` (e.g., `jacobian_6C$J`) matching the README API
- The Jacobian matrix dimensions must match the number of metabolites in the interaction network (rows/columns correspond to metabolites)
- Cross-condition comparison should yield distinct Jacobian matrices for 6C vs 16C when using the same `icount = 15` parameter
- Heatmap/3D visualization must render without error, confirming the matrix is valid and finite
- Re-running `calculate_jacobian` with identical inputs should reproduce the same Jacobian values (deterministic inference)

## Limitations

- Requires a priori definition of the metabolite interaction network (`interactions_fin`); without it Jacobian computation cannot proceed
- `num_tp = 1` implies single time point; multi-time-point dynamics require adjustment of the `num_tp` parameter
- The `icount = 15` parameter controls interaction count and may need tuning for different network sizes
- Reliance on MetaboAnalyst preprocessing outputs means errors upstream propagate into covariance and Jacobian estimates

## Evidence
