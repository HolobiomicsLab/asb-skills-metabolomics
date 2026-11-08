---
name: percentage-change-recomputation
description: Use when validating or recomputing a reported percent change in annotation
  or feature counts between two replicate groups (e.g., original vs PNNL PreProcessor-preprocessed
  LC-DT IM-MS lipid annotations) from deposited feature/annotation lists.
when_to_use_negative:
- Input is raw LC-IM-MS data (Agilent .d files) rather than already-deposited feature/annotation
  lists — feature finding and lipid annotation with Lipid Annotator is the upstream
  step, not this skill.
- The two groups being compared have unequal replicate numbers or fewer than 3 replicates
  — the 'at least half of analyzed MS-files' consensus threshold (≥2 of 3) must be
  redefined before percentages are comparable.
- The quantity of interest is MS1 lipid feature counts rather than lipid annotations
  — the 70.7% smoothing-driven feature increase is a different measure from the 19.4%
  annotation increase and needs its own counting basis.
edam_operation: http://edamontology.org/operation_3454
edam_topics:
- http://edamontology.org/topic_3391
tools:
- name: PNNL PreProcessor
  role: Produced the preprocessed LC-DT IM-MS files whose lipid annotation counts
    are compared against the original files' counts to compute the percentage change
  repo: https://omics.pnl.gov/software/pnnl-preprocessor
- name: Lipid Annotator
  role: Feature finding and lipid annotation engine for both file groups; its consensus
    rule (annotation present in at least half of analyzed MS-files) defines the ≥2-of-3
    replicate counting basis
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1021/acs.jproteome.1c00425
    title: A Preprocessing Tool for Enhanced Ion Mobility–Mass Spectrometry-Based
      Omics Workflows
  source_content_hash: sha256:c916bbd78f804a025ccd4364ec5fad79a3a93cdafd17ddd88aef8f6ea3529a6c
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Lipid Annotator
  - PNNL PreProcessor
  edam_operation: http://edamontology.org/operation_3454
  edam_topics:
  - http://edamontology.org/topic_3391
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# percentage-change-recomputation

## Summary

Recomputes the reported percent change in total lipid annotations between original and PNNL PreProcessor-preprocessed LC-DT IM-MS replicate groups by counting annotations supported in at least 2 of 3 replicates (Lipid Annotator consensus rule), e.g. reproducing the 19.4% annotation increase for a human plasma lipid extract.

## When to use

Use when validating or recomputing a reported percent change in annotation or feature counts between two replicate groups (e.g., original vs PNNL PreProcessor-preprocessed LC-DT IM-MS lipid annotations) from deposited feature/annotation lists.

## When NOT to use

- Input is raw LC-IM-MS data (Agilent .d files) rather than already-deposited feature/annotation lists — feature finding and lipid annotation with Lipid Annotator is the upstream step, not this skill.
- The two groups being compared have unequal replicate numbers or fewer than 3 replicates — the 'at least half of analyzed MS-files' consensus threshold (≥2 of 3) must be redefined before percentages are comparable.
- The quantity of interest is MS1 lipid feature counts rather than lipid annotations — the 70.7% smoothing-driven feature increase is a different measure from the 19.4% annotation increase and needs its own counting basis.

## Inputs

- Deposited lists of detected features and lipid annotations for the 3 original LC-DT IM-MS files of the human plasma lipid extract (MassIVE MSV000087295)
- Deposited lists of detected features and lipid annotations for the 3 PNNL PreProcessor-preprocessed LC-DT IM-MS files (same accession)
- Agilent .d MS-files (original and PreProcessor-generated), only if regenerating annotations with Lipid Annotator

## Outputs

- lipid_annotation_comparison.csv: table with per-group total lipid annotation counts and the recomputed percentage increase
- Per-group annotation counts restricted to features present in at least 2 of 3 replicates

## How to apply

Download the deposited lists of detected features and lipid annotations for the 3 original and the 3 PreProcessor-preprocessed LC-DT IM-MS lipid files of the human plasma lipid extract (MassIVE MSV000087295). Apply the Lipid Annotator consensus rule — an MS1-level feature must be present in at least half of the analyzed MS-files — so each annotation is counted only if supported in at least 2 of 3 replicates, independently for the original and preprocessed groups. Compute the percent increase as (preprocessed count − original count) / original count × 100, keeping the identical ≥2-of-3 replicate basis for both groups. Record the per-group counts and the recomputed percentage in lipid_annotation_comparison.csv. Note that the counting basis changes the result: counting each file individually gives an average 41% ± 7% increase, while MS1 lipid features consistently detected across 3 replicates increased by 70.7% with smoothing, so bases must not be mixed. Validation: the recomputed percentage must fall within the 19–20% range reported in the paper, and the counts must match the deposited annotation lists.

## Related tools

- **PNNL PreProcessor** (Produced the preprocessed LC-DT IM-MS files whose lipid annotation counts are compared against the original files' counts to compute the percentage change) — https://omics.pnl.gov/software/pnnl-preprocessor
- **Lipid Annotator** (Feature finding and lipid annotation engine for both file groups; its consensus rule (annotation present in at least half of analyzed MS-files) defines the ≥2-of-3 replicate counting basis)

## Evaluation signals

- Recomputed percentage increase lands within the reported 19–20% range (paper value: 19.4% for features in at least 2 replicates).
- Per-group annotation counts exactly match the deposited annotation lists in MassIVE MSV000087295.
- Both groups are counted on the identical consensus basis: annotation present in at least 2 of 3 replicates (at least half of analyzed MS-files).
- lipid_annotation_comparison.csv contains both per-group counts and the percent-increase column and parses cleanly as a table.
- Per-file sanity check: analyzing each file individually should reproduce an average increase near 41% ± 7%, distinct from the 19.4% consensus figure.

## Limitations

- The percentage is counting-basis-dependent: 19.4% applies to annotations for features in ≥2 of 3 replicates, per-file analysis gave 41% ± 7%, and 3-replicate-consistent MS1 features rose 70.7% — mixing bases misrepresents the result.
- Counts depend on the upstream parameters of the preprocessed files (demultiplexing with 75% pulses, smoothing with 3 points in LC and 7 in IM, 50-count intensity threshold) and on the completeness of the deposited annotation lists.
- No PreProcessor version number is specified in the article, and the tool is distributed via the omics.pnl.gov download page, which may complicate exact reproduction.

## Evidence

- [results] Alternative counting basis (features vs annotations) yields a different percent change that must not be conflated: [evidence span withheld — anchor status: not_found]
