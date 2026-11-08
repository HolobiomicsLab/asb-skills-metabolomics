---
name: differential-jacobian-ratio-analysis-between-conditions
description: Use when two metabolomic condition subsets (e.g. Cífková et al. 16C control
  vs 6C stress) are available and the goal is to rank metabolites by changes in their
  regulatory interaction strength via Jacobian ratios.
when_to_use_negative:
- When only a single condition is available — the skill requires prepare_data subsetting
  into two conditions to compute differential ratios.
- When the goal is standard MetaboAnalyst statistical pathway analysis rather than
  dynamic interaction-network/Jacobian analysis of metabolites.
- When metabolite IDs are not present in both rows and columns of the input dataset,
  as required for interaction-network construction.
edam_operation: http://edamontology.org/operation_3439
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: MInfer
  role: R package providing prepare_data, generate_covariance, calculate_jacobian
    and visualize_heatmap used to build condition-specific Jacobian matrices and differential
    ratios
  repo: https://github.com/cellbiomaths/MInfer
- name: MetaboAnalyst
  role: Upstream metabolomic data processing whose outputs MInfer integrates before
    Jacobian analysis
- name: R
  role: Execution environment for the MInfer package and its Jacobian workflow
- name: devtools
  role: Used to install MInfer from GitHub (devtools::install_github)
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.cmpb.2025.108672
    title: 'MInfer: Bridging MetaboAnalyst and Jacobian analysis for metabolomic networks'
  source_content_hash: sha256:6e1abb3d7c0f90e54dee83eb263853173b00c478e43946cc4aa9bbbad638c256
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
  edam_operation: http://edamontology.org/operation_3439
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# differential-jacobian-ratio-analysis-between-conditions

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Computes condition-specific Jacobian matrices with the MInfer R package and derives element-wise differential Jacobian ratios between two conditions (e.g. 6C cold stress vs 16C control) to identify metabolites whose regulatory roles intensify under stress.

## When to use

Use when two metabolomic condition subsets (e.g. Cífková et al. 16C control vs 6C stress) are available and the goal is to rank metabolites by changes in their regulatory interaction strength via Jacobian ratios.

## When NOT to use

- When only a single condition is available — the skill requires prepare_data subsetting into two conditions to compute differential ratios.
- When the goal is standard MetaboAnalyst statistical pathway analysis rather than dynamic interaction-network/Jacobian analysis of metabolites.
- When metabolite IDs are not present in both rows and columns of the input dataset, as required for interaction-network construction.

## Inputs

- MInfer example_data metabolomic dataset (met_input) with two conditions, e.g. 6C stress and 16C control
- Metabolite interaction network (interactions_fin) over metabolite IDs such as KEGG IDs
- List of target metabolites for ratio extraction (e.g. citrate, fumarate, glutamate, raffinose, leucine)

## Outputs

- Condition-specific Jacobian matrices (jacobian_6C$J, jacobian_16C$J)
- Heatmap visualizations of condition-specific Jacobian matrices (e.g. 'Jacobian Matrix - 6C')
- differential_jacobian_ratios.csv containing element-wise Jacobian ratios for the target metabolites

## How to apply

Load the two-condition benchmark metabolomic dataset via data(example_data) in R with the MInfer package installed from GitHub (devtools::install_github). Subset each condition with prepare_data (condition 6 for 6C stress, condition 16 for 16C control), then generate covariance matrices for each subset with generate_covariance using num_tp = 1. Compute each condition's Jacobian matrix with calculate_jacobian against the metabolite interaction network, passing icount = 15, then compare the 6C Jacobian matrix against the 16C Jacobian matrix element-wise to obtain differential Jacobian ratios for target metabolites. Visualize each condition-specific matrix with visualize_heatmap (e.g. title 'Jacobian Matrix - 6C') and export the ratio table for metabolites such as citrate, fumarate, glutamate, raffinose and leucine to differential_jacobian_ratios.csv. The rationale is that MInfer bridges MetaboAnalyst outputs with Jacobian analysis, so metabolites with increased ratios under stress are interpreted as having intensified regulatory roles.

## Related tools

- **MInfer** (R package providing prepare_data, generate_covariance, calculate_jacobian and visualize_heatmap used to build condition-specific Jacobian matrices and differential ratios) — https://github.com/cellbiomaths/MInfer
- **MetaboAnalyst** (Upstream metabolomic data processing whose outputs MInfer integrates before Jacobian analysis)
- **R** (Execution environment for the MInfer package and its Jacobian workflow)
- **devtools** (Used to install MInfer from GitHub (devtools::install_github))

## Examples

```
data(example_data); data_6C <- prepare_data(met_input, 6); cov_6C <- generate_covariance(data_6C, num_tp = 1); jacobian_6C <- calculate_jacobian(cov_6C[[1]], interactions_fin, icount = 15)
```

## Evaluation signals

- differential_jacobian_ratios.csv contains ratio values for all five target metabolites (citrate, fumarate, glutamate, raffinose, leucine).
- Each of citrate, fumarate, glutamate, raffinose and leucine shows an increased Jacobian ratio under 6C relative to 16C, matching the reported low-temperature stress behavior.
- Covariance matrices are generated with num_tp = 1 for both conditions before Jacobian computation.
- Jacobian matrices are computed with icount = 15 against the metabolite interaction network for both conditions.
- Heatmap visualizations render for each condition-specific Jacobian matrix (e.g. 'Jacobian Matrix - 6C').

## Limitations

- Validation is grounded in the Cífková et al. low-temperature benchmark (16C control vs 6C stress); other datasets require their own biological validation of ratio directions.
- The Jacobian calculation depends on the supplied metabolite interaction network (interactions_fin) and icount parameter, so results vary with network quality and icount choice.
- num_tp specifies the number of time points; the demonstrated workflow uses a single time point (num_tp = 1).

## Evidence

- [results] Increased Jacobian ratios for citrate, fumarate, glutamate, raffinose, leucine under cold stress: [evidence span withheld — anchor status: not_found]
- [abstract] MInfer bridges MetaboAnalyst and Jacobian analysis: "MInfer combines the comprehensive data processing capabilities of MetaboAnalyst with the mathematical modeling power of Jacobian analysis"
