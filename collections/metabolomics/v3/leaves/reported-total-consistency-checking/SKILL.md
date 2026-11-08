---
name: reported-total-consistency-checking
description: Use when a paper reports exact counts of workflow/method combinations
  (e.g., 5,904/48/1,968 NOREVA workflows under QCS, IS, and neither conditions) and
  you must reproduce them by re-enumerating the underlying combinatorial space.
when_to_use_negative:
- The goal is performance assessment (S5) of the enumerated workflows against the
  five criteria rather than verifying that the combinatorial count itself is reproducible.
- The published method counts per stage are not available or have changed (NOREVA's
  method set cannot be extended automatically and must be manually updated by bioinformaticians).
- The target counts come from a study whose original processing was never described,
  so even NOREVA had to infer outcomes 'across all possible workflows' rather than
  against a stated total.
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: R
  role: Language used to script the ordered combination generation across S1–S4 stages
    and to write the enumeration table (workflow_enumeration.csv)
- name: NOREVA
  role: Source of the stage method catalogue (4 imputation, 3 QCS-correction, 2 transformation,
    23 normalization methods) whose combinations are being enumerated and checked
  repo: https://github.com/idrblab/NOREVA
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41596-021-00636-9
    title: Optimization of metabolomic data processing using NOREVA
  source_content_hash: sha256:b841b1364db417afcc132bea744e834e51568eb2e234e12836eff61ac1a92965
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
yes_rate: 0.75
n_claims: 4
---

# reported-total-consistency-checking

## Summary

A validation skill that independently recomputes combinatorial enumerations (here, NOREVA's 4 imputation × 3 QCS-correction × 2 transformation × 23 normalization method combinations) and accepts the result only when computed scenario totals exactly match published values (5,904 QCS, 48 IS, 1,968 without both).

## When to use

Use when a paper reports exact counts of workflow/method combinations (e.g., 5,904/48/1,968 NOREVA workflows under QCS, IS, and neither conditions) and you must reproduce them by re-enumerating the underlying combinatorial space.

## When NOT to use

- The goal is performance assessment (S5) of the enumerated workflows against the five criteria rather than verifying that the combinatorial count itself is reproducible.
- The published method counts per stage are not available or have changed (NOREVA's method set cannot be extended automatically and must be manually updated by bioinformaticians).
- The target counts come from a study whose original processing was never described, so even NOREVA had to infer outcomes 'across all possible workflows' rather than against a stated total.

## Inputs

- machine-readable specification of NOREVA stage method counts (4, 3, 2, 23 for S1 imputation, S2 QCS correction, S3 transformation, S4 normalization)
- published reported totals to check against (5,904 QCS / 48 IS / 1,968 neither)
- scenario definitions (with QCSs, with ISs, without both)

## Outputs

- workflow_enumeration.csv with columns for scenario, stage methods, and workflow ID
- computed totals per scenario for direct comparison against the reported 5,904 / 48 / 1,968 values

## How to apply

Encode the per-stage method counts (4 data filtering/imputation, 3 QCS-correction, 2 transformation, 23 normalization methods) into a machine-readable specification, then use an R script to generate all ordered S1→S2→S3→S4 combinations, mirroring NOREVA's 'three to five defined calculations in combinatorially determined sequences'. Apply stage-applicability constraints per scenario: include the QCS-correction stage (S2) only for data containing quality-control samples, and restrict S4 normalization to IS-compatible methods for data with internal standards. Count the valid workflows in each of the three scenarios (with QCSs, with ISs, without both) and write a full enumeration table with columns for scenario, stage methods, and workflow ID. Accept the enumeration only if all three computed totals match the reported values exactly — 5,904 (QCS data), 48 (IS data), and 1,968 (neither) — recorded in workflow_enumeration.csv. This mirrors how NOREVA itself ranks candidate workflows discovered by scanning thousands of combinations before S5 multi-criteria assessment.

## Related tools

- **R** (Language used to script the ordered combination generation across S1–S4 stages and to write the enumeration table (workflow_enumeration.csv))
- **NOREVA** (Source of the stage method catalogue (4 imputation, 3 QCS-correction, 2 transformation, 23 normalization methods) whose combinations are being enumerated and checked) — https://github.com/idrblab/NOREVA

## Examples

```
Rscript enumerate_workflows.R --stages S1=4,S2=3,S3=2,S4=23 --scenarios qcs,is,neither --output workflow_enumeration.csv
```

## Evaluation signals

- Computed total for QCS-data scenario equals exactly 5,904 workflows.
- Computed total for IS-data scenario equals exactly 48 workflows.
- Computed total for no-QCS/no-IS scenario equals exactly 1,968 workflows.
- The encoded stage specification reproduces the per-stage method counts (4, 3, 2, 23) from the article.
- Each enumerated workflow in the table carries scenario, stage methods, and workflow ID columns, and QCS-correction methods appear only in QCS scenarios while normalization methods are restricted to IS-compatible ones in IS scenarios.

## Limitations

- The protocol covers only commonly applied processing methods and cannot be extended automatically; new methods require manual updates by bioinformaticians.
- Enumerating and assessing all workflows is time-consuming — hundreds to thousands of minutes, and >200 min even with parallel computing for large datasets such as PMID28528106.
- A passing count check does not guarantee workflow quality: performance is dataset-dependent (e.g., MDI + NON + NON + [CON-AUT] worked well for PMID29520289 but scored 'Fair' and 'Poor' for PMID18582365), so counts must be paired with S5 multi-criteria assessment.

## Evidence

- [intro] combinatorial diversity of NOREVA workflows: [evidence span withheld — anchor status: not_found]
- [discussion] optimal workflow found by scanning thousands of combinations: [evidence span withheld — anchor status: not_found]
- [readme] README: scanning thousands of processing workflows: [evidence span withheld — anchor status: external_doc]
