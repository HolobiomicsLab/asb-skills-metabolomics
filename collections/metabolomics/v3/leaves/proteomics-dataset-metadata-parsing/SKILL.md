---
name: proteomics-dataset-metadata-parsing
description: Use when a proteomics dataset has a ProteomeXchange/PRIDE identifier
  (e.g. PXD037527) and you need per-file counts partitioned by acquisition mode (DDA,
  WWA, DIA) and instrument platform (e.g. Orbitrap Astral, timsTOF Ultra, Orbitrap
  Exploris 480).
when_to_use_negative:
- The task requires spectral-level analysis (PSMs, peptides, protein groups at a given
  FDR filter) rather than a file-inventory partition of a deposited dataset.
- No ProteomeXchange/PRIDE identifier or structured metadata listing exists for the
  dataset, so per-file mode and instrument annotations cannot be retrieved.
- MSConnect is not a standalone data analysis tool — do not use this skill to substitute
  for third-party search/quantification software on the raw files themselves.
edam_operation: http://edamontology.org/operation_0222
edam_topics:
- http://edamontology.org/topic_0121
tools:
- name: MSConnect
  role: Platform whose PXD037527 demonstration dataset is being characterized; its
    Django/SQL-backed structure enables searching data and files through a web interface
    or programmatically
  repo: https://github.com/RTKlab-BYU/MSConnect
- name: PRIDE (ProteomeXchange partner repository)
  role: Source of the PXD037527 metadata and raw-file listing that is parsed and grouped
    by acquisition mode and instrument
  repo: https://www.ebi.ac.uk/pride/archive/projects/PXD037527
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1021/acs.jproteome.4c00854
    title: 'MSConnect: Open-Source, End-to-End Platform for Automated Mass Spectrometry
      Data Management, Analysis, and Visualization'
  source_content_hash: sha256:3dbe98b2ef9c6b40e785e8da9031bfb683865165a284367b7ccb6f1f8b2d7d8d
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/RTKlab-BYU/MSConnect
  tools:
  - MSConnect
  - PRIDE (ProteomeXchange partner repository)
  edam_operation: http://edamontology.org/operation_0222
  edam_topics:
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# proteomics-dataset-metadata-parsing

## Summary

Parses ProteomeXchange/PRIDE project metadata (e.g. PXD037527) into a per-file table annotated with acquisition mode (DDA, WWA, DIA) and instrument platform, then partitions file counts by mode and instrument. It grounds high-throughput dataset characterization in verifiable repository records rather than paper claims alone.

## When to use

Use when a proteomics dataset has a ProteomeXchange/PRIDE identifier (e.g. PXD037527) and you need per-file counts partitioned by acquisition mode (DDA, WWA, DIA) and instrument platform (e.g. Orbitrap Astral, timsTOF Ultra, Orbitrap Exploris 480).

## When NOT to use

- The task requires spectral-level analysis (PSMs, peptides, protein groups at a given FDR filter) rather than a file-inventory partition of a deposited dataset.
- No ProteomeXchange/PRIDE identifier or structured metadata listing exists for the dataset, so per-file mode and instrument annotations cannot be retrieved.
- MSConnect is not a standalone data analysis tool — do not use this skill to substitute for third-party search/quantification software on the raw files themselves.

## Inputs

- ProteomeXchange/PRIDE project metadata and raw-file listing for PXD037527
- Per-file acquisition-mode annotations (DDA, WWA, DIA)
- Per-file instrument-platform annotations (Orbitrap Astral, timsTOF Ultra, Orbitrap Exploris 480)

## Outputs

- file_counts_by_mode_and_instrument.csv with columns: acquisition mode, instrument platform, file count
- Reconciled total file count (e.g. 6410 for the 26-day MSConnect demonstration period)

## How to apply

Retrieve the project metadata and raw-file listing from the PRIDE partner repository (e.g. https://www.ebi.ac.uk/pride/archive/projects/PXD037527) and parse per-file annotations for acquisition mode (DDA, WWA, DIA) and instrument platform (Orbitrap Astral, timsTOF Ultra, Orbitrap Exploris 480). Group the file records by acquisition mode and instrument platform, counting raw files per combination (e.g. a pandas groupby over the parsed metadata table). Cross-check totals against reported figures — for PXD037527, 6048 LC-MS analyses on an Orbitrap Astral plus 362 additional files from a timsTOF Ultra and an Orbitrap Exploris 480 during the 26-day window September 26 to October 21, 2024 (6048 + 362 = 6410 total). Write the partitioned summary to file_counts_by_mode_and_instrument.csv with columns for acquisition mode, instrument platform, and file count. The rationale is that manual reconciliation of thousands of files is error prone; a parsed, grouped table makes the partition auditable against both the repository listing and the manuscript.

## Related tools

- **MSConnect** (Platform whose PXD037527 demonstration dataset is being characterized; its Django/SQL-backed structure enables searching data and files through a web interface or programmatically) — https://github.com/RTKlab-BYU/MSConnect
- **PRIDE (ProteomeXchange partner repository)** (Source of the PXD037527 metadata and raw-file listing that is parsed and grouped by acquisition mode and instrument) — https://www.ebi.ac.uk/pride/archive/projects/PXD037527

## Examples

```
import pandas as pd; df = pd.read_csv('pxd037527_metadata.tsv'); df.groupby(['acquisition_mode','instrument']).size().to_csv('file_counts_by_mode_and_instrument.csv')
```

## Evaluation signals

- Summed file counts reconcile with the reported figures: 6048 (Orbitrap Astral) + 362 (timsTOF Ultra + Orbitrap Exploris 480) = 6410 total analyses for September 26 to October 21, 2024.
- file_counts_by_mode_and_instrument.csv contains the three required columns (acquisition mode, instrument platform, file count) with non-negative integer counts.
- Every file in the PRIDE listing is assigned to exactly one acquisition mode among DDA, WWA, and DIA (no unclassified or double-counted records).
- Per-instrument subtotals match the paper's partition: 6048 files on the Orbitrap Astral, 362 across timsTOF Ultra and Orbitrap Exploris 480.
- Re-running the parse over a fresh PRIDE metadata download reproduces the same grouped counts (deterministic, repository-grounded result).

## Limitations

- Counts depend on the completeness and annotation quality of the PRIDE submission; missing or inconsistent per-file instrument/mode annotations will produce an incorrect partition.
- The 26-day window (September 26 to October 21, 2024) must be respected when filtering uploads — files deposited outside that period will inflate totals beyond 6410.
- The paper notes method descriptions are often insufficient to ensure reproducibility; the cross-check against published figures (6048 + 362) is the primary validation, so discrepancies may reflect paper-level reporting rather than parsing errors.

## Evidence
