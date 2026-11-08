---
name: metabolomic-workflow-combinatorial-enumeration
description: Use when a metabolomic peak table (case-control, multiclass, or time-course)
  must be exhaustively scanned across all valid processing method combinations to
  find the study-specific optimal workflow before S5 multi-criteria assessment.
when_to_use_negative:
- The peak table already has a fixed, user-prescribed processing pipeline and no combinatorial
  optimization is intended — enumeration would only add hundreds-to-thousands of minutes
  of scanning time
- The dataset has fewer than 6 samples per class/timepoint (criterion Cc inapplicable)
  or lacks reference metabolites/spike-ins (criterion Ce/LogFC inapplicable), so the
  downstream S5 assessment that consumes the enumeration cannot be completed
- The data are not metabolomic peak tables in NOREVA's unified format and cannot be
  standardized (manual standardization step not yet performed)
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: NOREVA
  role: R package (v2.1.1) whose functions scan thousands of enumerated processing
    workflows and rank them by multi-criteria performance; provides the S1–S4 method
    implementations being enumerated
  repo: https://github.com/idrblab/NOREVA
- name: R
  role: Language used to write the enumeration script that generates all ordered method
    combinations across stages and applies scenario applicability constraints
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41596-021-00636-9
    title: Optimization of metabolomic data processing using NOREVA
  source_content_hash: sha256:9c4e9b2108f9699dae4dd1d0f6ddd14760b1cfb3d82f67440954d63094327019
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/idrblab/NOREVA
  tools:
  - NOREVA
  - R
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 6
---

# metabolomic-workflow-combinatorial-enumeration

## Summary

Enumerates the full combinatorial space of metabolomic peak-table processing workflows by combining NOREVA's stage-specific methods (4 imputation, 3 QCS-correction, 2 transformation, 23 normalization) under scenario-specific applicability constraints, yielding 5,904 workflows for QCS data, 48 for IS data, and 1,968 for data with neither.

## When to use

Use when a metabolomic peak table (case-control, multiclass, or time-course) must be exhaustively scanned across all valid processing method combinations to find the study-specific optimal workflow before S5 multi-criteria assessment.

## When NOT to use

- The peak table already has a fixed, user-prescribed processing pipeline and no combinatorial optimization is intended — enumeration would only add hundreds-to-thousands of minutes of scanning time
- The dataset has fewer than 6 samples per class/timepoint (criterion Cc inapplicable) or lacks reference metabolites/spike-ins (criterion Ce/LogFC inapplicable), so the downstream S5 assessment that consumes the enumeration cannot be completed
- The data are not metabolomic peak tables in NOREVA's unified format and cannot be standardized (manual standardization step not yet performed)

## Inputs

- Machine-readable specification file of NOREVA stage method counts (S1 imputation, S2 QCS correction, S3 transformation, S4 normalization)
- Scenario descriptor indicating data type: with QCSs, with internal standards (IS column indices), or without both
- NOREVA-standardized peak table (rows = samples, columns = metabolites) for downstream scanning

## Outputs

- workflow_enumeration.csv with columns scenario, stage methods, and workflow ID
- Validated workflow counts: 5,904 (QCS data), 48 (IS data), 1,968 (neither QCSs nor ISs)
- Candidate workflow list passed to S5 multi-criteria performance assessment (5 criteria)

## How to apply

Encode the NOREVA stage method counts into a machine-readable specification: 4 data filtering/imputation methods (S1), 3 QCS-correction methods (S2), 2 transformation methods (S3), and 23 normalization methods (S4). Use an R script to generate all ordered combinations across the fixed stage sequence S1 → S2 → S3 → S4, mirroring the protocol's 'three to five defined calculations in combinatorially determined sequences'. Apply scenario-specific stage-applicability constraints: include the QCS-correction stage only for data containing quality-control samples, and restrict S4 normalization methods to IS-compatible ones for data with internal standards — this is why the IS scenario yields only 48 workflows while the QCS scenario yields 5,904 and the no-QCS/no-IS scenario yields 1,968. Write the full enumeration table (scenario, stage methods, workflow ID) to workflow_enumeration.csv, then feed all candidates into the S5 collective assessment using five independent criteria. Validation is exact: computed counts must match 5,904 (with QCSs), 48 (with ISs), and 1,968 (neither) precisely.

## Related tools

- **NOREVA** (R package (v2.1.1) whose functions scan thousands of enumerated processing workflows and rank them by multi-criteria performance; provides the S1–S4 method implementations being enumerated) — https://github.com/idrblab/NOREVA
- **R** (Language used to write the enumeration script that generates all ordered method combinations across stages and applies scenario applicability constraints)

## Examples

```
library(NOREVA); normulticlassqcall(fileName, SAalpha="Y", SAbeta="Y", SAgamma="Y")
```

## Evaluation signals

- Computed workflow totals match all three reported values exactly: 5,904 (QCS data), 48 (IS data), 1,968 (no QCS/IS), in workflow_enumeration.csv
- Every enumerated workflow follows the ordered stage sequence S1 imputation → S2 QCS correction → S3 transformation → S4 normalization with no out-of-order or missing stages
- No workflow in the IS-data scenario uses a normalization method incompatible with internal standards; no workflow in the no-QCS scenario includes an S2 QCS-correction method
- Every enumerated workflow ID appears exactly once per scenario (no duplicates) and the ID set is closed under the stage-method product constrained by scenario
- The enumeration feeds a complete scan in which the top-ranked workflow is reproducible, e.g. HAM + LLR + NON + [SUM-LEV] ranked first among 5,904 workflows for the PMID28528106 dataset

## Limitations

- Only commonly applied processing methods are covered; the protocol cannot be extended automatically and must be manually updated by bioinformaticians when new methods are added
- Assessing all enumerated workflows is very time-consuming — hundreds to thousands of minutes — and even large datasets (e.g., PMID28528106) require >200 min despite parallel computing and memory management in NOREVA 2.1.1
- Valid enumeration counts depend on correct upfront experimental design (adoption of both QCSs and ISs), which the tool does not enforce

## Evidence

- [intro] Combinatorial diversity of workflows: [evidence span withheld — anchor status: not_found]
- [discussion] Optimal workflow among 5,904 scanned: [evidence span withheld — anchor status: not_found]
- [readme] README scan capability: [evidence span withheld — anchor status: external_doc]
- [readme] README five-criteria evaluation: [evidence span withheld — anchor status: external_doc]
