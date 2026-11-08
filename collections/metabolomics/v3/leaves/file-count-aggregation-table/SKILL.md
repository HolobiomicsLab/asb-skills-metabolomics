---
name: file-count-aggregation-table
description: Use when you have per-file metadata (raw-file listing with acquisition-mode
  and instrument annotations, e.g., from a PRIDE PXD identifier) and need a partitioned
  count table grouped by mode and instrument to audit dataset composition.
when_to_use_negative:
- Input is already an aggregated count table or summary statistic — no per-file records
  to group
- The task requires spectral-level or PSM-level analysis rather than file-level dataset
  composition auditing
- File records lack acquisition-mode or instrument annotations, making groupby partitioning
  impossible without re-annotation
edam_operation: http://edamontology.org/operation_0306
edam_topics:
- http://edamontology.org/topic_0121
tools:
- name: MSConnect
  role: Platform whose deposited dataset (PXD037527) and uploaded raw files are the
    object of the count aggregation
  repo: https://github.com/RTKlab-BYU/MSConnect
- name: ProteomeXchange Consortium via the PRIDE partner repository
  role: Source of the project metadata and raw-file listing that is parsed and grouped
- name: DIA-NN
  role: Software noted in the finding as automatically analyzing DIA-mode files across
    the 6410-file corpus
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1021/acs.jproteome.4c00854
    title: 'MSConnect: Open-Source, End-to-End Platform for Automated Mass Spectrometry
      Data Management, Analysis, and Visualization'
  source_content_hash: sha256:34d00490a2e1ba3dd6a43fe220bc70f1069176c52923ee33df53bc5966c83720
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/RTKlab-BYU/MSConnect
  tools:
  - DIA-NN
  - MSConnect
  - ProteomeXchange Consortium via the PRIDE partner repository
  edam_operation: http://edamontology.org/operation_0306
  edam_topics:
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# file-count-aggregation-table

## Summary

Aggregates raw LC-MS file records by acquisition mode (DDA, WWA, DIA) and instrument platform into a summary count table, reconciling totals against paper-reported figures. Mirrors the MSConnect PXD037527 audit where 6410 files were partitioned across Orbitrap Astral, timsTOF Ultra, and Orbitrap Exploris 480.

## When to use

Use when you have per-file metadata (raw-file listing with acquisition-mode and instrument annotations, e.g., from a PRIDE PXD identifier) and need a partitioned count table grouped by mode and instrument to audit dataset composition.

## When NOT to use

- Input is already an aggregated count table or summary statistic — no per-file records to group
- The task requires spectral-level or PSM-level analysis rather than file-level dataset composition auditing
- File records lack acquisition-mode or instrument annotations, making groupby partitioning impossible without re-annotation

## Inputs

- PRIDE project metadata / raw-file listing for a PXD identifier (e.g., PXD037527)
- Per-file records annotated with acquisition mode (DDA, WWA, DIA) and instrument platform
- Reported file totals from the source manuscript for cross-validation (e.g., 6048 + 362 = 6410)

## Outputs

- file_counts_by_mode_and_instrument.csv with columns: acquisition mode, instrument platform, file count
- Reconciled total count check against paper-reported figures

## How to apply

Retrieve the project metadata and raw-file listing from the PRIDE partner repository (e.g., https://www.ebi.ac.uk/pride/archive/projects/PXD037527 for PXD037527). Parse each file record to extract its acquisition mode annotation (DDA, WWA, DIA) and instrument platform (Orbitrap Astral, timsTOF Ultra, Orbitrap Exploris 480). Load the records into a pandas DataFrame and apply a groupby over acquisition mode and instrument platform, counting raw files per combination. Sum all counts and reconcile against any reported totals — for PXD037527, 6048 LC-MS analyses on the Orbitrap Astral plus 362 additional files from a timsTOF Ultra and an Orbitrap Exploris 480 during the 26-day window September 26 to October 21, 2024 must sum to 6410. Write the result to file_counts_by_mode_and_instrument.csv with columns for acquisition mode, instrument platform, and file count. As an enrichment, note which software handled each mode (e.g., DIA files were automatically analyzed using DIA-NN).

## Related tools

- **MSConnect** (Platform whose deposited dataset (PXD037527) and uploaded raw files are the object of the count aggregation) — https://github.com/RTKlab-BYU/MSConnect
- **ProteomeXchange Consortium via the PRIDE partner repository** (Source of the project metadata and raw-file listing that is parsed and grouped)
- **DIA-NN** (Software noted in the finding as automatically analyzing DIA-mode files across the 6410-file corpus)

## Examples

```
df.groupby(['acquisition_mode','instrument_platform']).size().reset_index(name='file_count').to_csv('file_counts_by_mode_and_instrument.csv', index=False)
```

## Evaluation signals

- Summed file counts reconcile exactly with paper-reported totals: 6048 (Orbitrap Astral) + 362 (timsTOF Ultra + Orbitrap Exploris 480) = 6410 analyses
- Output CSV contains exactly the three expected columns: acquisition mode, instrument platform, file count
- Every input file record appears in exactly one (mode, instrument) group — no records dropped or double-counted
- Only the expected instrument platforms (Orbitrap Astral, timsTOF Ultra, Orbitrap Exploris 480) and modes (DDA, WWA, DIA) appear in the table
- Counts fall within the 26-day window September 26 to October 21, 2024 when the files were uploaded

## Limitations

- Counts depend on the completeness and annotation quality of the PRIDE metadata; missing acquisition-mode or instrument annotations break the partition
- The 362-file bucket is itself an aggregate of two instruments (timsTOF Ultra and Orbitrap Exploris 480) in the paper's reporting, so finer per-instrument splits require the raw-file listing, not just the manuscript figures
- The aggregation validates a fixed reporting window (26 days); files outside that window will not reconcile against the 6410 total

## Evidence
