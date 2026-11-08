---
name: stage-applicability-constraint-verification
description: Use when enumerating candidate metabolomic data-processing workflows
  and needing to constrain stage combinations to the dataset scenario (with QCSs,
  with ISs, or without both).
when_to_use_negative:
- The dataset includes quality-control samples but you intend to enumerate workflows
  without the S2 QCS-correction stage (or vice versa, apply S2 to data lacking QCSs).
- The dataset contains internal standards but S4 normalization methods not compatible
  with ISs are enumerated (this would break the 48-workflow IS-scenario count).
- The goal is already workflow ranking or performance assessment (S5, five criteria)
  rather than verifying which stages apply and enumerating valid combinations.
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: NOREVA
  role: Source of the stage-specific method sets and of scenario-specific entry functions
    (qcall / isall / noall) whose applicability conditions the constraints encode
  repo: github.com/idrblab/NOREVA
- name: R
  role: R-based protocol used to generate all ordered method combinations across stages
    and apply the scenario applicability constraints during enumeration
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41596-021-00636-9
    title: Optimization of metabolomic data processing using NOREVA
  source_content_hash: sha256:276f2d0d08a7693c628ce68169da90abb8fb782c937738c10a0a914254f24f9b
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - NOREVA
  - R
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# stage-applicability-constraint-verification

## Summary

Verifies that NOREVA processing stages are only applied to the data scenarios in which they are applicable — the QCS-correction stage (S2) only for data with quality-control samples and IS-compatible normalization (S4) only for data with internal standards — so that combinatorial workflow enumeration yields the correct scenario-specific workflow counts.

## When to use

Use when enumerating candidate metabolomic data-processing workflows and needing to constrain stage combinations to the dataset scenario (with QCSs, with ISs, or without both).

## When NOT to use

- The dataset includes quality-control samples but you intend to enumerate workflows without the S2 QCS-correction stage (or vice versa, apply S2 to data lacking QCSs).
- The dataset contains internal standards but S4 normalization methods not compatible with ISs are enumerated (this would break the 48-workflow IS-scenario count).
- The goal is already workflow ranking or performance assessment (S5, five criteria) rather than verifying which stages apply and enumerating valid combinations.

## Inputs

- Machine-readable specification file of NOREVA stage method counts (S1: 4 imputation, S2: 3 QCS-correction, S3: 2 transformation, S4: 23 normalization methods)
- Data scenario declaration: with quality-control samples (QCSs), with internal standards (ISs), or neither
- NOREVA-compatible standardized peak table (from PrepareInuputFiles) when workflows are to be run on data

## Outputs

- workflow_enumeration.csv with columns for scenario, stage methods, and workflow ID
- Valid workflow counts per scenario: 5,904 (QCS data), 48 (IS data), 1,968 (neither)
- R-generated ordered method combinations across S1→S2→S3→S4 per applicable scenario

## How to apply

Encode the NOREVA stage method counts (4 data filtering/imputation methods in S1, 3 QCS-correction methods in S2, 2 transformation methods in S3, 23 normalization methods in S4) into a machine-readable specification, then apply stage-applicability constraints before generating ordered combinations across S1→S2→S3→S4: include the QCS-correction stage only for data with quality-control samples, and restrict S4 to IS-compatible normalization methods for data with internal standards. Use R scripts (e.g., from github.com/idrblab/NOREVA) to generate all valid ordered method combinations per scenario, mirroring the protocol's three to five defined calculations in combinatorially determined sequences. Count the valid workflows in each of the three scenarios (with QCSs, with ISs, without both) and write a full enumeration table with columns for scenario, stage methods, and workflow ID. Validate only if the computed totals match the reported values exactly: 5,904 workflows for QCS data, 48 for IS data, and 1,968 for data with neither.

## Related tools

- **NOREVA** (Source of the stage-specific method sets and of scenario-specific entry functions (qcall / isall / noall) whose applicability conditions the constraints encode) — github.com/idrblab/NOREVA
- **R** (R-based protocol used to generate all ordered method combinations across stages and apply the scenario applicability constraints during enumeration)

## Examples

```
normulticlassnoall(fileName, SAalpha="Y", SAbeta="Y", SAgamma="Y")
```

## Evaluation signals

- Computed workflow totals match all three reported values exactly: 5,904 for QCS data, 48 for IS data, and 1,968 for data with neither.
- No enumerated workflow for the no-QCS/no-IS scenario contains an S2 QCS-correction method.
- Every workflow in the IS-data scenario uses only IS-compatible normalization methods from S4.
- workflow_enumeration.csv has the required schema (scenario, stage methods, workflow ID) and every workflow ID is a unique ordered method combination.
- The README's scenario-specific functions (normulticlassqcall / normulticlassisall / normulticlassnoall) correspond one-to-one with the three enumerated scenarios.

## Limitations

- Scanning the enumerated workflows for performance is very time-consuming: hundreds to thousands of minutes, and large datasets (e.g., PMID28528106) still require >200 min despite parallel computing and memory management.
- The protocol covers only commonly applied processing methods and cannot be automatically extended; new methods must be added manually by bioinformaticians.
- Successful application depends on upfront experimental design — adoption of both QCSs and ISs is recommended but not enforced by the tool.

## Evidence

- [intro] Combinatorial workflow diversity across stages: [evidence span withheld — anchor status: not_found]
- [readme] README function restricted to QCS-without-IS scenario: [evidence span withheld — anchor status: external_doc]
- [readme] README function restricted to no-QCS/no-IS scenario: [evidence span withheld — anchor status: not_found]
