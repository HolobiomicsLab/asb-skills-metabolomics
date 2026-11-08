---
name: instrument-platform-partitioning
description: Use when raw-file-level metadata from a ProteomeXchange/PRIDE deposition
  (e.g., PXD037527) must be grouped and counted by acquisition mode (DDA/WWA/DIA)
  and instrument platform to characterize dataset composition, throughput, or search-tool
  routing (e.g., DIA-NN for DIA files).
when_to_use_negative:
- Input is already an aggregated summary table or results table (e.g., PSM/peptide/protein-group
  FDR-filtered tables) rather than file-level metadata
- The deposition's files lack instrument or acquisition-mode annotations, so grouping
  axes cannot be parsed
- The goal is data processing/analysis of spectra (MSConnect is 'not a standalone
  data analysis tool' and depends on third-party software such as DIA-NN for that)
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: MSConnect
  role: Source platform whose SQL-linked data structure allows searching uploaded
    raw files and metadata programmatically; the partitioning targets files ingested
    through its Raw File Uploader
  repo: https://github.com/RTKlab-BYU/MSConnect
- name: ProteomeXchange Consortium via the PRIDE partner repository
  role: Public repository from which the PXD037527 metadata and raw-file listing are
    retrieved for partitioning
  repo: https://www.ebi.ac.uk/pride/archive/projects/PXD037527
- name: DIA-NN
  role: Software identified during partitioning as the automatic analyzer of DIA-mode
    files across all 6410 files
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1021/acs.jproteome.4c00854
    title: 'MSConnect: Open-Source, End-to-End Platform for Automated Mass Spectrometry
      Data Management, Analysis, and Visualization'
  source_content_hash: sha256:89887ac3d1f5e76168e4f56effe639b2072a8a2e06ebdc9bab74b120b724478c
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/RTKlab-BYU/MSConnect
  tools:
  - DIA-NN
  - MSConnect
  - ProteomeXchange Consortium via the PRIDE partner repository
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# instrument-platform-partitioning

## Summary

Partitions a deposition's raw LC-MS files by acquisition mode (DDA, WWA, DIA) and instrument platform (e.g., Orbitrap Astral, timsTOF Ultra, Orbitrap Exploris 480) to quantify dataset composition and throughput, as demonstrated by reconciling MSConnect's 26-day demo period (6048 + 362 = 6410 analyses) against PRIDE metadata for PXD037527.

## When to use

Use when raw-file-level metadata from a ProteomeXchange/PRIDE deposition (e.g., PXD037527) must be grouped and counted by acquisition mode (DDA/WWA/DIA) and instrument platform to characterize dataset composition, throughput, or search-tool routing (e.g., DIA-NN for DIA files).

## When NOT to use

- Input is already an aggregated summary table or results table (e.g., PSM/peptide/protein-group FDR-filtered tables) rather than file-level metadata
- The deposition's files lack instrument or acquisition-mode annotations, so grouping axes cannot be parsed
- The goal is data processing/analysis of spectra (MSConnect is 'not a standalone data analysis tool' and depends on third-party software such as DIA-NN for that)

## Inputs

- PRIDE/PXD037527 project metadata from https://www.ebi.ac.uk/pride/archive/projects/PXD037527
- Raw-file listing with per-file acquisition mode (DDA, WWA, DIA) and instrument platform annotations
- Reported reference counts from the source manuscript (6048 Orbitrap Astral analyses + 362 timsTOF Ultra/Exploris 480 files over 26 days)

## Outputs

- file_counts_by_mode_and_instrument.csv with columns: acquisition mode, instrument platform, file count
- Reconciled total file count (expected 6410 for the MSConnect 26-day demonstration window)

## How to apply

Retrieve the deposition's metadata and raw-file listing from the PRIDE partner repository using the dataset identifier (e.g., PXD037527 from the MSConnect paper). Parse per-file annotations to extract acquisition mode (DDA, WWA, DIA) and instrument platform (Orbitrap Astral, timsTOF Ultra, Orbitrap Exploris 480) for each raw file. Group the file records by both axes and count each combination, e.g., a pandas groupby over the parsed metadata table. Cross-check the totals against the paper's reported figures: 6048 LC-MS analyses on the Orbitrap Astral plus 362 additional files from a timsTOF Ultra and an Orbitrap Exploris 480 during the 26-day window September 26 to October 21, 2024 (6410 total). Write the partitioned table to file_counts_by_mode_and_instrument.csv with columns for acquisition mode, instrument platform, and file count.

## Related tools

- **MSConnect** (Source platform whose SQL-linked data structure allows searching uploaded raw files and metadata programmatically; the partitioning targets files ingested through its Raw File Uploader) — https://github.com/RTKlab-BYU/MSConnect
- **ProteomeXchange Consortium via the PRIDE partner repository** (Public repository from which the PXD037527 metadata and raw-file listing are retrieved for partitioning) — https://www.ebi.ac.uk/pride/archive/projects/PXD037527
- **DIA-NN** (Software identified during partitioning as the automatic analyzer of DIA-mode files across all 6410 files)

## Examples

```
import pandas as pd; df = pd.read_csv('pxd037527_files.tsv', sep='\t'); df.groupby(['acquisition_mode','instrument_platform']).size().to_csv('file_counts_by_mode_and_instrument.csv')
```

## Evaluation signals

- Summed partition counts reconcile with the manuscript figures: 6048 (Orbitrap Astral) + 362 (timsTOF Ultra + Orbitrap Exploris 480) = 6410 total analyses
- The date window September 26 to October 21, 2024 is respected when filtering files, matching the reported 26-day demonstration period
- Every raw file is assigned to exactly one (acquisition mode, instrument platform) pair — groupby sizes sum to the total file count with no unclassified records
- Output CSV schema contains exactly the columns acquisition mode, instrument platform, and file count
- DIA-mode files are consistently identified as those routed to DIA-NN, matching the paper's statement that DIA files across all 6410 files were analyzed with DIA-NN

## Limitations

- Counts depend on the quality of per-file instrument and acquisition-mode annotations in the PRIDE deposition; missing or inconsistent annotations will produce unclassified records
- The paper's figures are specific to the 26-day September 26 to October 21, 2024 window; the timsTOF Ultra and Orbitrap Exploris 480 files are only reported as a combined 362, so the per-instrument split may need further parsing
- MSConnect's README notes no changelog, and the manuscript notes the experimental design module is 'not yet released', limiting programmatic tracking of experimental-design metadata alongside file counts

## Evidence
