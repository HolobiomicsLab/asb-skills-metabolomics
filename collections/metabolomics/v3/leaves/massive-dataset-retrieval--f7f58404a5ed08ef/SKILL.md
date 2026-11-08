---
name: massive-dataset-retrieval
description: Decision support for validating a published preprocessing effect (e.g.
  the 19.4% increase in lipid annotations) by pulling the exact per-file annotation
  lists from the paper's MassIVE accession instead of re-running the LC-DT IM-MS instrument
  data.
when_to_use_negative:
- When raw instrument data (.d files) are available and the user wants to reproduce
  preprocessing itself rather than verify its downstream annotation effect from deposited
  outputs.
edam_operation: http://edamontology.org/operation_2422
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: Lipid Annotator
  role: Tool whose consensus rule (feature in ≥ half of MS-files) defines the counting
    basis when re-tallying annotations from the retrieved lists
- name: PNNL PreProcessor
  role: Source of the preprocessed MS-files whose deposited annotation lists are retrieved
    for the comparison
  repo: https://omics.pnl.gov/software/pnnl-preprocessor
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1021/acs.jproteome.1c00425
    title: A Preprocessing Tool for Enhanced Ion Mobility–Mass Spectrometry-Based
      Omics Workflows
  source_content_hash: sha256:d4c81c84463ff8887cd530a92fb4ff864a2bc5bb0742e87011f571c6f5f00cfc
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Lipid Annotator
  - PNNL PreProcessor
  edam_operation: http://edamontology.org/operation_2422
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# massive-dataset-retrieval

## Summary

Skill for retrieving the deposited feature and lipid-annotation lists from the MassIVE repository accession MSV000087295 (3 original and 3 preprocessed LC-DT IM-MS lipid files), so that the reported 19.4% annotation increase can be recomputed independently.

## When to use

Decision support for validating a published preprocessing effect (e.g. the 19.4% increase in lipid annotations) by pulling the exact per-file annotation lists from the paper's MassIVE accession instead of re-running the LC-DT IM-MS instrument data.

## When NOT to use

- When raw instrument data (.d files) are available and the user wants to reproduce preprocessing itself rather than verify its downstream annotation effect from deposited outputs.

## Inputs

- MassIVE accession MSV000087295 (deposited lists of detected features and lipid annotations for 3 original and 3 preprocessed LC-DT IM-MS files)

## Outputs

- Table (e.g. lipid_annotation_comparison.csv) with per-group lipid annotation counts and recomputed percent increase

## How to apply

Query the MassIVE repository with accession MSV000087295 to obtain the deposited lists of detected features and lipid annotations for the 3 original and 3 preprocessed LC-DT IM-MS files from a human plasma lipid extract analyzed in triplicate. Apply the Lipid Annotator consensus rule — an MS1-level feature must be present in at least half of the analyzed MS-files — so each group's annotations are counted on the features present in at least 2 of 3 replicates. This mirrors the published analysis basis and lets the recomputed percent increase be compared directly with the reported 19.4% figure; per-group counts are written to a table such as lipid_annotation_comparison.csv.

## Related tools

- **Lipid Annotator** (Tool whose consensus rule (feature in ≥ half of MS-files) defines the counting basis when re-tallying annotations from the retrieved lists)
- **PNNL PreProcessor** (Source of the preprocessed MS-files whose deposited annotation lists are retrieved for the comparison) — https://omics.pnl.gov/software/pnnl-preprocessor

## Examples

```
massive.fetch --accession MSV000087295 --output lipid_annotation_comparison.csv
```

## Evaluation signals

- Recomputed percentage increase falls within the 19–20% range reported in the paper
- Per-group annotation counts match the deposited annotation lists in MSV000087295
- Counting uses features present in at least 2 of 3 replicates, consistent with the consensus rule
- Both groups (original vs preprocessed) are represented with 3 files each

## Limitations

- Relies on deposited outputs, so it cannot verify upstream preprocessing behaviour (e.g. saturation repair or demultiplexing quality) directly
- Requires that the original and preprocessed groups both contain 3 replicates; results are not transferable to unbalanced replicate sets

## Evidence

- [methods] Finding: [evidence span withheld — anchor status: not_found]
- [results] Workflow: [evidence span withheld — anchor status: not_found]
