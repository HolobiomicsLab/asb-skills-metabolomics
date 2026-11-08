---
name: r-metabolomics-package-workflow-execution
description: Use when metabolomics data must be processed and analyzed in R via the
  MInfer package, e.g. preparing conditions, generating covariance matrices (num_tp
  = 1), and calculating Jacobians of metabolite interactions.
when_to_use_negative:
- The input metabolite IDs are not in KEGG IG format or are absent from the rows and
  columns of the dataset, as required by the minfer function.
- The analysis goal is raw metabolomic data processing (peak detection, normalization)
  rather than Jacobian-based network analysis — MInfer is designed to consume outputs
  downstream of MetaboAnalyst-style processing.
- Multi-time-point data without knowing the correct num_tp setting is required, since
  the example workflow demonstrates only num_tp = 1.
edam_operation: http://edamontology.org/operation_3439
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0091
tools:
- name: MInfer
  role: R package whose workflow (prepare_data, generate_covariance, calculate_jacobian,
    visualize_heatmap/visualize_3d, minfer) is executed to analyze metabolomic interaction
    networks
  repo: https://github.com/cellbiomaths/MInfer
- name: R
  role: runtime environment in which the MInfer package and devtools installation
    are executed
- name: devtools
  role: used to install MInfer from GitHub (devtools::install_github)
- name: MetaboAnalyst
  role: upstream metabolomic analysis tool whose outputs MInfer integrates with Jacobian
    analysis
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.cmpb.2025.108672
    title: 'MInfer: Bridging MetaboAnalyst and Jacobian analysis for metabolomic networks'
  source_content_hash: sha256:41af41bc793a02dedcc2266fab3b39e7b29d8e78b4b165c89bb5743bff70d417
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
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# r-metabolomics-package-workflow-execution

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Executes the MInfer R package pipeline—loading metabolomic input, preparing condition-specific data, generating covariance matrices, and computing Jacobian matrices—to bridge MetaboAnalyst-style data processing with Jacobian-based metabolomic network analysis.

## When to use

Use when metabolomics data must be processed and analyzed in R via the MInfer package, e.g. preparing conditions, generating covariance matrices (num_tp = 1), and calculating Jacobians of metabolite interactions.

## When NOT to use

- The input metabolite IDs are not in KEGG IG format or are absent from the rows and columns of the dataset, as required by the minfer function.
- The analysis goal is raw metabolomic data processing (peak detection, normalization) rather than Jacobian-based network analysis — MInfer is designed to consume outputs downstream of MetaboAnalyst-style processing.
- Multi-time-point data without knowing the correct num_tp setting is required, since the example workflow demonstrates only num_tp = 1.

## Inputs

- metabolite input data frame (met_input) with rows/columns keyed by metabolite identifiers, loaded via data(example_data)
- condition selector (e.g. 6 or 16 for the 6C/16C example conditions) passed to prepare_data
- metabolite interaction stoichiometry matrix (interactions_fin) supplied to calculate_jacobian
- list of KEGG IG metabolite IDs (e.g. 'C00042', 'C00149', 'C00036') for the minfer function

## Outputs

- per-condition covariance matrices from generate_covariance (e.g. cov_6C[[1]])
- per-condition Jacobian matrices from calculate_jacobian (accessed as jacobian$J)
- heatmap and 3D visualizations of Jacobian matrices via visualize_heatmap and visualize_3d
- minfer results object printed for the metabolite interaction network (MIN)

## How to apply

Install MInfer from GitHub with devtools::install_github('https://github.com/cellbiomaths/MInfer') and load it with library(MInfer). Load the bundled example data with data(example_data), then prepare one dataset per condition using prepare_data(met_input, <condition>) — e.g. prepare_data(met_input, 6) for 6C and prepare_data(met_input, 16) for 16C. For each prepared condition, generate a covariance matrix with generate_covariance(data, num_tp = 1), where num_tp specifies the number of time points (single time-point data uses num_tp = 1). Compute the Jacobian matrix with calculate_jacobian(cov[[1]], interactions_fin, icount = 15), supplying the metabolite interaction stoichiometry (interactions_fin) and icount = 15. Visualize the resulting Jacobian matrices with visualize_heatmap(jacobian$J, title = ...) or visualize_3d(jacobian$J); a higher-level minfer(input_ids) function accepts a list of KEGG IG metabolite IDs (e.g. 'C00042', 'C00149', 'C00036') that must be present in the rows and columns of the dataset.

## Related tools

- **MInfer** (R package whose workflow (prepare_data, generate_covariance, calculate_jacobian, visualize_heatmap/visualize_3d, minfer) is executed to analyze metabolomic interaction networks) — https://github.com/cellbiomaths/MInfer
- **R** (runtime environment in which the MInfer package and devtools installation are executed)
- **devtools** (used to install MInfer from GitHub (devtools::install_github))
- **MetaboAnalyst** (upstream metabolomic analysis tool whose outputs MInfer integrates with Jacobian analysis)

## Examples

```
library(MInfer); data(example_data); data_6C <- prepare_data(met_input, 6); cov_6C <- generate_covariance(data_6C, num_tp = 1); jacobian_6C <- calculate_jacobian(cov_6C[[1]], interactions_fin, icount = 15); visualize_heatmap(jacobian_6C$J, title = "Jacobian Matrix - 6C")
```

## Evaluation signals

- library(MInfer) loads without error after devtools::install_github succeeds, and MASS (Imports) dependency is satisfied.
- prepare_data returns a condition-specific dataset for each requested condition (e.g. data_6C and data_16C) from the same met_input.
- generate_covariance with num_tp = 1 returns a covariance matrix accessible as cov[[1]] with dimensions matching the metabolite set.
- calculate_jacobian(cov[[1]], interactions_fin, icount = 15) returns a Jacobian retrievable via $J that can be passed directly to visualize_heatmap and visualize_3d.
- minfer(input_ids) returns a results object whose metabolite IDs (e.g. C00042, C00149, C00036) match the input KEGG IG list.

## Limitations

- Metabolite IDs for minfer must be KEGG IG identifiers and 'should be present in the rows and columns of the dataset'.
- The documented example is limited to single time-point data (num_tp = 1); behavior with multiple time points is not demonstrated in the README.
- The workflow depends on a supplied interaction/stoichiometry matrix (interactions_fin), which must come from outside the shown example commands.

## Evidence
