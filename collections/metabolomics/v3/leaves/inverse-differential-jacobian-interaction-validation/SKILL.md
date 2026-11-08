---
name: inverse-differential-jacobian-interaction-validation
description: Use when two metabolomic conditions are to be compared for differential
  metabolite-metabolite interaction identification, especially when validating against
  a curated model (e.g. BioModels BIOM0000000070) and scoring precision/sensitivity/specificity.
when_to_use_negative:
- Only one experimental condition is available — the inverse differential Jacobian
  requires two condition Jacobians to differ element-wise.
- No prior metabolite interaction stoichiometry (interactions_fin) is available, since
  calculate_jacobian requires it as input.
- The goal is MetaboAnalyst statistical preprocessing or pathway enrichment only,
  not dynamic interaction network inference.
edam_operation: http://edamontology.org/operation_0279
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: MInfer
  role: R package providing prepare_data, generate_covariance, calculate_jacobian,
    and visualization functions used to build the Jacobians and the inverse differential
    Jacobian
  repo: https://github.com/cellbiomaths/MInfer
- name: MetaboAnalyst
  role: upstream metabolomic data processing whose outputs feed the Jacobian analysis
    step
- name: R
  role: runtime environment in which the MInfer pipeline and validation scoring are
    executed
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.cmpb.2025.108672
    title: 'MInfer: Bridging MetaboAnalyst and Jacobian analysis for metabolomic networks'
  source_content_hash: sha256:d46aae2bb4e66d695c5818c52f33d24d60cb0c4d00ed5a1123629fdff27f280a
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
  edam_operation: http://edamontology.org/operation_0279
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# inverse-differential-jacobian-interaction-validation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Applies the MInfer Jacobian pipeline (prepare_data → generate_covariance → calculate_jacobian) to two metabolic conditions and computes the inverse differential Jacobian per Sun and Weckwerth to identify differential metabolite-metabolite interactions, scoring them with precision, sensitivity, and specificity against a curated reference such as the RBC model BIOM0000000070.

## When to use

Use when two metabolomic conditions are to be compared for differential metabolite-metabolite interaction identification, especially when validating against a curated model (e.g. BioModels BIOM0000000070) and scoring precision/sensitivity/specificity.

## When NOT to use

- Only one experimental condition is available — the inverse differential Jacobian requires two condition Jacobians to differ element-wise.
- No prior metabolite interaction stoichiometry (interactions_fin) is available, since calculate_jacobian requires it as input.
- The goal is MetaboAnalyst statistical preprocessing or pathway enrichment only, not dynamic interaction network inference.

## Inputs

- metabolite input data (met_input, e.g. the MInfer example_data dataset)
- condition labels for the two states to compare (e.g. 6C and 16C)
- metabolite interaction stoichiometry matrix (interactions_fin)
- reference interaction list from a curated model (e.g. BioModels BIOM0000000070 red blood cell metabolism model)

## Outputs

- per-condition covariance matrices (cov_6C, cov_16C)
- per-condition Jacobian matrices (jacobian_6C$J, jacobian_16C$J)
- inverse differential Jacobian (element-wise difference of the two condition Jacobians)
- classification of metabolite-metabolite interactions versus the reference model
- metrics table of precision, sensitivity, and specificity (validation_metrics.csv)

## How to apply

Load metabolite input data (e.g. data(example_data) in R) and prepare each condition with prepare_data(met_input, <condition>), such as conditions 6 and 16. Generate a covariance matrix per condition with generate_covariance(data, num_tp = 1) and estimate a Jacobian per condition with calculate_jacobian(cov[[1]], interactions_fin, icount = 15), supplying the metabolite interaction stoichiometry (interactions_fin). Compute the inverse differential Jacobian as the element-wise difference between the two condition Jacobians, following Sun and Weckwerth. Classify the identified interactions against the reference interactions of the curated RBC model and score identification quality with precision, sensitivity, and specificity. Inspect Jacobian structure with visualize_heatmap(jacobian$J) before or after differencing.

## Related tools

- **MInfer** (R package providing prepare_data, generate_covariance, calculate_jacobian, and visualization functions used to build the Jacobians and the inverse differential Jacobian) — https://github.com/cellbiomaths/MInfer
- **MetaboAnalyst** (upstream metabolomic data processing whose outputs feed the Jacobian analysis step)
- **R** (runtime environment in which the MInfer pipeline and validation scoring are executed)

## Examples

```
cov_6C <- generate_covariance(prepare_data(met_input, 6), num_tp = 1); jacobian_6C <- calculate_jacobian(cov_6C[[1]], interactions_fin, icount = 15)
```

## Evaluation signals

- A validation_metrics.csv containing precision, sensitivity, and specificity computed against the curated reference interactions is produced.
- Two separate Jacobian matrices (e.g. jacobian_6C$J and jacobian_16C$J) are generated with identical dimensions matching the metabolite set before differencing.
- The inverse differential Jacobian equals the element-wise difference between the two condition Jacobians, consistent with the Sun and Weckwerth formulation.
- generate_covariance is called with num_tp = 1 and calculate_jacobian with icount = 15, matching the documented example parameters.
- Interactions classified as identified overlap the reference interactions of the curated RBC model (BIOM0000000070).

## Limitations

- The calculate_jacobian step depends on a supplied metabolite interaction stoichiometry (interactions_fin); without a prior interaction network the Jacobian cannot be estimated.
- Validation metrics depend on the quality/completeness of the curated reference model's interaction list; incomplete references bias precision, sensitivity, and specificity.
- The demonstrated workflow uses num_tp = 1 (single time point); behavior with multi-time-point data is not covered by the documented example.

## Evidence
