---
name: metabolomic-network-inference
description: Use when metabolomic intensity data (rows/columns labelled with KEGG
  compound IDs such as 'C00042') must be converted into an inferred metabolite interaction
  network with topology statistics (nodes, edges, neighbours, path length, clustering
  coefficient, density).
when_to_use_negative:
- When the input is already a finished network edge list or adjacency matrix rather
  than raw metabolomic intensity data requiring Jacobian-based inference.
- When the goal is standard MetaboAnalyst pathway-enrichment or statistical testing
  without dynamic interaction modelling between metabolites.
- When the metabolite identifiers are not KEGG compound IDs present in both rows and
  columns of the dataset, as minfer() expects.
edam_operation: http://edamontology.org/operation_3791
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: MInfer
  role: Core R package providing prepare_data(), generate_covariance(), calculate_jacobian(),
    visualize_heatmap(), visualize_3d(), and the high-level minfer() wrapper used
    to build the interaction network
  repo: https://github.com/cellbiomaths/MInfer
- name: MetaboAnalyst
  role: Upstream source of metabolomic data processing and KEGG/SMP pathway identifiers
    that MInfer bridges into Jacobian analysis
- name: devtools
  role: Used to install MInfer from GitHub before running the pipeline
- name: R
  role: Runtime environment for all MInfer commands
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.cmpb.2025.108672
    title: 'MInfer: Bridging MetaboAnalyst and Jacobian analysis for metabolomic networks'
  source_content_hash: sha256:38f88d7263c19a546483d4c3936abe012c30f2f126b51cf3660c9b855b5225fb
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
  edam_operation: http://edamontology.org/operation_3791
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# metabolomic-network-inference

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

An R-based skill that bridges MetaboAnalyst outputs and Jacobian analysis to infer dynamic metabolite interaction networks, using the MInfer package to prepare data, generate covariance matrices, compute Jacobian matrices, and visualise metabolite interaction networks.

## When to use

Use when metabolomic intensity data (rows/columns labelled with KEGG compound IDs such as 'C00042') must be converted into an inferred metabolite interaction network with topology statistics (nodes, edges, neighbours, path length, clustering coefficient, density).

## When NOT to use

- When the input is already a finished network edge list or adjacency matrix rather than raw metabolomic intensity data requiring Jacobian-based inference.
- When the goal is standard MetaboAnalyst pathway-enrichment or statistical testing without dynamic interaction modelling between metabolites.
- When the metabolite identifiers are not KEGG compound IDs present in both rows and columns of the dataset, as minfer() expects.

## Inputs

- Metabolomic dataset with KEGG compound IDs (e.g. C00042, C00149, C00036) in both rows and columns
- Condition-specific prepared data (e.g. 6C or 16C) produced by prepare_data()
- List of metabolite IDs (KEGG IG) drawn from MetaboAnalyst KEGG/SMP pathway resources
- Interaction/stoichiometry list (interactions_fin) for Jacobian computation

## Outputs

- Jacobian matrix object (jacobian_6C$J) describing metabolite interaction strengths
- Metabolite interaction network (MIN) results from minfer()
- Network topology statistics: node count, edge count, average neighbours, characteristic path length, clustering coefficient, density
- Heatmap or 3D visualisation of the Jacobian matrix

## How to apply

Install MInfer from GitHub via devtools::install_github("https://github.com/cellbiomaths/MInfer") and load it with library(MInfer). Define the metabolite IDs (KEGG IG) that appear in the rows and columns of the dataset, then either run the high-level minfer(input_ids) wrapper for a MIN analysis or follow the staged pipeline: prepare_data() to select a condition (e.g. 6C or 16C), generate_covariance() with num_tp = 1 for a single time point, and calculate_jacobian() supplying the interaction/stoichiometry list with icount = 15. Inspect the resulting Jacobian matrix (jacobian_6C$J) with visualize_heatmap() or visualize_3d() to identify inferred metabolite interactions, and derive topology metrics (node count, edge count, average neighbours, characteristic path length, clustering coefficient, density) from the network. The rationale is that MInfer combines MetaboAnalyst's data-processing capabilities with Jacobian analysis's mathematical modelling to capture dynamic interactions between metabolites.

## Related tools

- **MInfer** (Core R package providing prepare_data(), generate_covariance(), calculate_jacobian(), visualize_heatmap(), visualize_3d(), and the high-level minfer() wrapper used to build the interaction network) — https://github.com/cellbiomaths/MInfer
- **MetaboAnalyst** (Upstream source of metabolomic data processing and KEGG/SMP pathway identifiers that MInfer bridges into Jacobian analysis)
- **devtools** (Used to install MInfer from GitHub before running the pipeline)
- **R** (Runtime environment for all MInfer commands)

## Examples

```
library(MInfer); data_6C <- prepare_data(met_input, 6); cov_6C <- generate_covariance(data_6C, num_tp = 1); jacobian_6C <- calculate_jacobian(cov_6C[[1]], interactions_fin, icount = 15)
```

## Evaluation signals

- Verify that every metabolite ID supplied to minfer() is present in both the rows and columns of the input dataset.
- Check that generate_covariance() is called with num_tp = 1 for a single time point and that the resulting covariance matrix is passed as cov_6C[[1]] to calculate_jacobian().
- Confirm calculate_jacobian() is invoked with the interaction list and icount = 15, matching the documented example.
- Compare recomputed topology statistics against the reported lung-cancer benchmark: 121 nodes, 187 edges, 4.7 average neighbours, 3.2 characteristic path length, 0.616 clustering coefficient, 0.103 density.
- Ensure visualize_heatmap() produces a plot titled from the Jacobian matrix (e.g. 'Jacobian Matrix - 6C') without errors.

## Limitations

- The documented example uses a single time point (num_tp = 1); multi-time-point behaviour is not demonstrated in the provided context.
- Metabolite IDs must be KEGG IG identifiers that appear in both rows and columns; other identifier schemes are not covered by the shown workflow.
- The interaction/stoichiometry list (interactions_fin) and icount parameter must be supplied manually, so incorrect or incomplete interaction lists will propagate into the Jacobian matrix.

## Evidence

- [other] The lung cancer metabolomic interaction network inferred with MInfer comprises 121 nodes and 187 edges, with an average of 4.7 neighbors per node.: [evidence span withheld — anchor status: not_found]
