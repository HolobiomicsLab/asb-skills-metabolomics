---
name: peak-table-processing-stage-modeling
description: Use when a metabolomic peak table must be processed and you need to enumerate
  or reason about which ordered combinations of imputation, QCS-correction, transformation
  and normalization methods are applicable to the data scenario (with QCSs, with ISs,
  or neither).
when_to_use_negative:
- Input is not a peak table but raw instrument data — conversion/peak picking (Proteowizard,
  nmrML, XCMS, NMRProc Flow) must happen before stage modeling
- The study already prescribes a single fixed processing pipeline and no comparative
  workflow optimization is intended
- Data come from emerging techniques (e.g. pseudotargeted or spatial metabolomics)
  whose systematic bias structure the staged model has not been validated for
edam_operation: http://edamontology.org/operation_3557
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0219
tools:
- name: NOREVA
  role: Provides the stage definitions (S1–S5), the per-stage method pools, and the
    R functions that scan and rank the enumerated workflows
  repo: https://github.com/idrblab/NOREVA
- name: R
  role: Scripting environment used to generate all ordered method combinations across
    stages and enforce scenario-applicability constraints
  repo: https://www.r-project.org/
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41596-021-00636-9
    title: Optimization of metabolomic data processing using NOREVA
  source_content_hash: sha256:e12bc213704c870a04ce6666e385e2efb4459d2cc7336a58a792b4295ff1e1bc
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/idrblab/NOREVA
  tools:
  - NOREVA
  - R
  edam_operation: http://edamontology.org/operation_3557
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0219
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# peak-table-processing-stage-modeling

## Summary

Models metabolomic peak-table processing as an ordered sequence of method stages (S1 data filtering/imputation, S2 QCS correction, S3 transformation, S4 normalization) whose combinations are enumerated to expose the thousands of candidate workflows NOREVA scans before S5 multi-criterion assessment.

## When to use

Use when a metabolomic peak table must be processed and you need to enumerate or reason about which ordered combinations of imputation, QCS-correction, transformation and normalization methods are applicable to the data scenario (with QCSs, with ISs, or neither).

## When NOT to use

- Input is not a peak table but raw instrument data — conversion/peak picking (Proteowizard, nmrML, XCMS, NMRProc Flow) must happen before stage modeling
- The study already prescribes a single fixed processing pipeline and no comparative workflow optimization is intended
- Data come from emerging techniques (e.g. pseudotargeted or spatial metabolomics) whose systematic bias structure the staged model has not been validated for

## Inputs

- Standardized metabolomic peak table (rows = samples by name, columns = metabolites/features)
- Label file for time-course/multi-class study design
- Scenario specification: presence/absence of quality-control samples (QCSs) and internal standards (ISs), plus IS column indices if present
- Study assumption flags (SAalpha, SAbeta, SAgamma)

## Outputs

- Enumerated workflow table (scenario, stage methods, workflow ID) e.g. workflow_enumeration.csv
- Scenario-specific workflow counts: 5,904 (QCS), 48 (IS), 1,968 (neither)
- Ordered workflow definitions feeding the S5 five-criterion performance assessment

## How to apply

Encode the stage-specific method counts (4 data filtering/imputation methods, 3 QCS-correction methods, 2 transformation methods, 23 normalization methods) into a machine-readable specification, then generate all ordered method combinations across S1 imputation → S2 QCS correction → S3 transformation → S4 normalization with an R script. Apply scenario constraints: include the QCS-correction stage only for data containing quality-control samples, and restrict normalization methods to IS-compatible ones when internal standards are present. This yields the scenario-specific workflow spaces of 5,904 (QCS data), 48 (IS data) and 1,968 (no QCS/IS) workflows, which are then scanned comprehensively and assessed collectively using five independent criteria in S5. Remember that normalization (S4) requires first determining the type of study assumption (alpha: all metabolites equally important; beta: metabolite abundance constant among samples; gamma: majority of metabolites unchanged).

## Related tools

- **NOREVA** (Provides the stage definitions (S1–S5), the per-stage method pools, and the R functions that scan and rank the enumerated workflows) — https://github.com/idrblab/NOREVA
- **R** (Scripting environment used to generate all ordered method combinations across stages and enforce scenario-applicability constraints) — https://www.r-project.org/

## Examples

```
Rscript enumerate_workflows.R --stages S1:4,S2:3,S3:2,S4:23 --scenario qcs --out workflow_enumeration.csv
```

## Evaluation signals

- Enumeration totals must match the reported scenario counts exactly: 5,904 workflows for QCS data, 48 for IS data, 1,968 for neither
- QCS-correction stage (S2) is present in enumerated workflows only when quality-control samples exist in the data
- For IS data, only IS-compatible normalization methods appear among the 48 enumerated workflows
- Every enumerated workflow follows the fixed stage order S1 imputation → S2 QCS correction → S3 transformation → S4 normalization

## Limitations

- Only commonly applied processing methods are covered; the protocol cannot be extended automatically and must be constantly and manually updated by bioinformaticians
- Scanning thousands of workflows is very time-consuming (hundreds to thousands of minutes even with parallel computing), and datasets like PMID28528106 still require >200 min
- Successful application depends on upfront experimental design (adoption of both QCSs and ISs), which the tool does not enforce

## Evidence

- [intro] Combinatorial diversity motivates enumeration: [evidence span withheld — anchor status: not_found]
- [discussion] QCS-based signal correction is integrated into the stage model: [evidence span withheld — anchor status: not_found]
- [readme] README documents the scanning/ranking capability over thousands of workflows: [evidence span withheld — anchor status: external_doc]
