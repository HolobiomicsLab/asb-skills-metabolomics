---
name: grouped-metabolite-data-serialization
description: Use when a metabolomics matrix contains samples from multiple experimental
  conditions (e.g., 6C and 16C) and separate per-condition covariance/Jacobian analysis
  is planned with MInfer.
when_to_use_negative:
- The dataset contains only a single experimental condition — no grouping is needed
  and prepare_data's condition selection adds no value.
- The input is already a covariance matrix, Jacobian matrix, or MetaboAnalyst MIN
  output rather than a raw/processed metabolite sample matrix.
- The goal is the KEGG-ID-based minfer(input_ids) pathway, which skips example-data
  preparation entirely and starts from metabolite IDs (e.g., C00042, C00149, C00036).
edam_topics:
- http://edamontology.org/topic_3315
tools:
- name: MInfer
  role: Provides prepare_data(met_input, condition) used to select and group the metabolite
    matrix into condition-specific subsets before covariance/Jacobian analysis
  repo: https://github.com/cellbiomaths/MInfer
- name: R
  role: Execution environment for running prepare_data, loading example_data, and
    serializing grouped objects (e.g., saveRDS to .rds)
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.cmpb.2025.108672
    title: 'MInfer: Bridging MetaboAnalyst and Jacobian analysis for metabolomic networks'
  source_content_hash: sha256:5f9ea1a19161cff0c9ce43e8dcbea3e844a27ebef8fd140ba64357da1df66b73
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
  - R
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3315
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# grouped-metabolite-data-serialization

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Splits a metabolite input matrix (met_input) into condition-specific grouped subsets using MInfer's prepare_data (e.g., conditions 6C and 16C) and persists the resulting grouped objects so downstream covariance and Jacobian computations operate on condition-isolated data. This isolates per-condition dynamics before network inference in the MInfer pipeline.

## When to use

Use when a metabolomics matrix contains samples from multiple experimental conditions (e.g., 6C and 16C) and separate per-condition covariance/Jacobian analysis is planned with MInfer.

## When NOT to use

- The dataset contains only a single experimental condition — no grouping is needed and prepare_data's condition selection adds no value.
- The input is already a covariance matrix, Jacobian matrix, or MetaboAnalyst MIN output rather than a raw/processed metabolite sample matrix.
- The goal is the KEGG-ID-based minfer(input_ids) pathway, which skips example-data preparation entirely and starts from metabolite IDs (e.g., C00042, C00149, C00036).

## Inputs

- metabolite input matrix (met_input) containing samples from multiple conditions, with metabolite identifiers in rows/columns
- condition identifier(s) to select, e.g., 6 (6C) and 16 (16C) as used in the MInfer example data

## Outputs

- one grouped condition data object per condition (data_6C, data_16C)
- serialized RDS file (prepared_condition_data.rds) holding exactly two grouped condition objects derived from the same met_input

## How to apply

Load the metabolite input matrix (met_input) — from the MInfer example via data(example_data) or from your own processed MetaboAnalyst output — and identify the condition grouping variable. Call prepare_data(met_input, condition) once per condition, using the condition codes shown in the README (6 for 6C, 16 for 16C), yielding one grouped data object per condition (data_6C, data_16C). Verify each returned object contains only the samples and metabolite columns belonging to its condition and retains the metabolite identifiers needed by generate_covariance and calculate_jacobian downstream. Serialize both grouped objects (e.g., saveRDS into prepared_condition_data.rds) so the prepare step can be validated and reused before computing covariance matrices with generate_covariance(data_6C, num_tp = 1). The rationale is that covariance and Jacobian matrices must be computed per condition, so the prepare_data step is the mandatory first stage of the README's Jacobian workflow.

## Related tools

- **MInfer** (Provides prepare_data(met_input, condition) used to select and group the metabolite matrix into condition-specific subsets before covariance/Jacobian analysis) — https://github.com/cellbiomaths/MInfer
- **R** (Execution environment for running prepare_data, loading example_data, and serializing grouped objects (e.g., saveRDS to .rds))

## Examples

```
data_6C <- prepare_data(met_input, 6); data_16C <- prepare_data(met_input, 16)
```

## Evaluation signals

- prepared_condition_data.rds exists and loads successfully in R
- The serialized file contains exactly two grouped condition objects — one for condition 6 and one for condition 16
- Both grouped objects derive from the same met_input matrix (no sample leakage across conditions)
- Each grouped object retains the metabolite identifiers required by generate_covariance and calculate_jacobian
- Downstream compatibility check: generate_covariance(data_6C, num_tp = 1) executes without error on the grouped object

## Limitations

- The README only demonstrates conditions 6C and 16C via prepare_data(met_input, 6) and prepare_data(met_input, 16); other condition codes or grouping schemes are not documented.
- The README does not detail the internal structure or return type of prepare_data objects, so validation of the grouped format relies on downstream calls succeeding.
- This prepare step belongs to the example-data/Jacobian workflow; the alternative minfer(input_ids) pathway with KEGG IDs does not use condition-grouped data preparation.

## Evidence
