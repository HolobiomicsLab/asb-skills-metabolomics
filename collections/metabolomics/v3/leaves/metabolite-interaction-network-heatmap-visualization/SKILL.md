---
name: metabolite-interaction-network-heatmap-visualization
description: Use when you have computed condition-specific Jacobian matrices from
  metabolomic data (e.g. via MInfer's calculate_jacobian) and need to visualize metabolite
  interaction patterns as a heatmap for interpretation or comparison across conditions.
when_to_use_negative:
- You only have raw metabolite intensity data and no computed Jacobian matrix — run
  prepare_data, generate_covariance and calculate_jacobian first
- You need quantitative differential ratios between conditions rather than a per-condition
  visual — write the differential Jacobian ratio table instead
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3314
- http://edamontology.org/topic_0203
tools:
- name: MInfer
  role: Provides visualize_heatmap (and visualize_3d) for rendering Jacobian matrices
    of metabolite interaction networks; installed from GitHub
  repo: https://github.com/cellbiomaths/MInfer
- name: R
  role: Runtime environment in which the MInfer package and its visualization functions
    are executed
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.cmpb.2025.108672
    title: 'MInfer: Bridging MetaboAnalyst and Jacobian analysis for metabolomic networks'
  source_content_hash: sha256:fa0b3688256637deef906fb256f2e38a6308fc6ddcdad43cdb13f47033593a25
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
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3314
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# metabolite-interaction-network-heatmap-visualization

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Render condition-specific metabolite interaction networks as heatmaps of Jacobian matrices using MInfer's visualize_heatmap in R, enabling visual comparison of regulatory interaction strengths between conditions (e.g. 6C cold stress vs 16C control).

## When to use

Use when you have computed condition-specific Jacobian matrices from metabolomic data (e.g. via MInfer's calculate_jacobian) and need to visualize metabolite interaction patterns as a heatmap for interpretation or comparison across conditions.

## When NOT to use

- You only have raw metabolite intensity data and no computed Jacobian matrix — run prepare_data, generate_covariance and calculate_jacobian first
- You need quantitative differential ratios between conditions rather than a per-condition visual — write the differential Jacobian ratio table instead

## Inputs

- Jacobian matrix object from MInfer calculate_jacobian (e.g. jacobian_6C$J)
- Condition label/title string (e.g. 'Jacobian Matrix - 6C')

## Outputs

- Heatmap visualization of the metabolite interaction (Jacobian) matrix for a given condition

## How to apply

After computing a Jacobian matrix with MInfer's calculate_jacobian (e.g. using a covariance matrix from generate_covariance with num_tp = 1 and icount = 15), pass the resulting J matrix (accessed as jacobian_6C$J) to visualize_heatmap with a descriptive title such as 'Jacobian Matrix - 6C'. The heatmap shows interaction strengths between metabolites in the network, making intensified regulatory roles (e.g. for citrate, fumarate, glutamate, raffinose, leucine under cold stress) visually apparent. For complementary 3D inspection of the same matrices, MInfer also provides visualize_3d. Generate one heatmap per condition (e.g. 6C and 16C) so differential Jacobian ratios can be interpreted against the condition-specific visualizations.

## Related tools

- **MInfer** (Provides visualize_heatmap (and visualize_3d) for rendering Jacobian matrices of metabolite interaction networks; installed from GitHub) — https://github.com/cellbiomaths/MInfer
- **R** (Runtime environment in which the MInfer package and its visualization functions are executed)

## Examples

```
visualize_heatmap(jacobian_6C$J, title = "Jacobian Matrix - 6C")
```

## Evaluation signals

- The heatmap renders without error from a valid calculate_jacobian output object (the $J element of the returned list)
- A heatmap is produced for each condition analyzed (e.g. 'Jacobian Matrix - 6C' and the 16C counterpart), enabling side-by-side comparison
- The visualized interaction pattern is consistent with the differential Jacobian ratio table (e.g. increased ratios for citrate, fumarate, glutamate, raffinose and leucine under 6C stress appear as intensified interactions)
- The matrix dimensions of the plotted heatmap match the metabolite IDs used in the interaction network (icount = 15 setup)

## Limitations

- Visualization operates on the Jacobian matrix element ($J) of calculate_jacobian output; it does not itself compute differential statistics between conditions
- Heatmaps are one of two provided visual modes; 3D interaction views (visualize_3d) may be needed for a different perspective of the same matrix

## Evidence
