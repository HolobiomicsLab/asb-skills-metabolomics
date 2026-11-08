---
name: instrument-metadata-based-task-routing
description: Use when a mixed batch of raw MS files from multiple instruments (e.g.,
  Orbitrap Astral, timsTOF Ultra, Orbitrap Exploris 480) spans DDA, WWA, and DIA acquisition
  modes and each file must be sent to the matching search engine automatically.
when_to_use_negative:
- MSConnect is expected to run the analysis itself — it is a wrapper/interface, not
  a standalone data analysis tool, and third-party engines like FragPipe and DIA-NN
  must be separately licensed and installed
- All incoming files share a single acquisition mode and instrument, so per-file routing
  adds no value over a single fixed pipeline
- Files lack acquisition-mode/instrument metadata, so mode-conditional routing decisions
  cannot be made reliably
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: MSConnect
  role: Web-based platform hosting the task queue; the Omics Data Processor downloads
    per-file tasks, invokes the assigned search engine's CLI, and uploads results
  repo: https://github.com/RTKlab-BYU/MSConnect
- name: FragPipe
  role: Search engine invoked via CLI by the DDA/WWA ODP worker to produce PSMs, peptides,
    and protein groups
- name: DIA-NN
  role: Search engine invoked via CLI by the DIA ODP worker for data-independent acquisition
    files
- name: Raw_File_Uploader
  role: Resides on the mass spectrometer control computer and sends MS data files
    and associated metadata (used for routing) to the MSConnect Server
  repo: https://github.com/RTKlab-BYU/Raw_File_Uploader
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/acs.jproteome.4c00854
    title: 'MSConnect: Open-Source, End-to-End Platform for Automated Mass Spectrometry
      Data Management, Analysis, and Visualization'
  source_content_hash: sha256:b4d78122b56a042960723a520a3d9b046b69bae75eb30c830eef1907df6a3cb9
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/RTKlab-BYU/MSConnect
  tools:
  - DIA-NN
  - FragPipe
  - MSConnect
  - PRIDE
  - Raw_File_Uploader
  - msconvert
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 12
---

# instrument-metadata-based-task-routing

## Summary

Routes high-throughput LC-MS raw files to the correct search engine by reading each file's acquisition mode and instrument origin from its metadata, as demonstrated by MSConnect automatically processing 6410 files (FragPipe for DDA/WWA, DIA-NN for DIA) without incident.

## When to use

Use when a mixed batch of raw MS files from multiple instruments (e.g., Orbitrap Astral, timsTOF Ultra, Orbitrap Exploris 480) spans DDA, WWA, and DIA acquisition modes and each file must be sent to the matching search engine automatically.

## When NOT to use

- MSConnect is expected to run the analysis itself — it is a wrapper/interface, not a standalone data analysis tool, and third-party engines like FragPipe and DIA-NN must be separately licensed and installed
- All incoming files share a single acquisition mode and instrument, so per-file routing adds no value over a single fixed pipeline
- Files lack acquisition-mode/instrument metadata, so mode-conditional routing decisions cannot be made reliably

## Inputs

- Raw LC-MS data files from multiple instruments (Orbitrap Astral, timsTOF Ultra, Orbitrap Exploris 480)
- Metadata table assigning each file an acquisition mode (DDA, DIA, or WWA) and instrument origin
- Search engine configuration files (FragPipe for DDA/WWA, DIA-NN for DIA)
- Deposited dataset identifier (e.g., PXD037527 from PRIDE)

## Outputs

- Per-file identification results: PSMs, peptides, and protein groups passing a given FDR filter
- Aggregated table with one row per deposited file containing file name, acquisition mode, engine used, and identification statistics

## How to apply

Download the raw MS files and associated metadata (e.g., dataset PXD037527 from the PRIDE partner repository), then read the metadata table to assign each file its acquisition mode (DDA, DIA, or WWA) and its instrument origin. Configure one MSConnect Omics Data Processor (ODP) worker per search engine—FragPipe/MSFragger for DDA and WWA files, DIA-NN for DIA files—as a processing app in the MSConnect Server queue. Submit one automated processing task per file so the ODP downloads each raw file plus its configuration file, invokes the engine's command-line interface, and returns identification results (PSMs, peptides, protein groups passing the FDR filter). Finally, aggregate per-file results into a single table with columns for file name, acquisition mode, engine used, and identification statistics, verifying one row per deposited file.

## Related tools

- **MSConnect** (Web-based platform hosting the task queue; the Omics Data Processor downloads per-file tasks, invokes the assigned search engine's CLI, and uploads results) — https://github.com/RTKlab-BYU/MSConnect
- **FragPipe** (Search engine invoked via CLI by the DDA/WWA ODP worker to produce PSMs, peptides, and protein groups)
- **DIA-NN** (Search engine invoked via CLI by the DIA ODP worker for data-independent acquisition files)
- **Raw_File_Uploader** (Resides on the mass spectrometer control computer and sends MS data files and associated metadata (used for routing) to the MSConnect Server) — https://github.com/RTKlab-BYU/Raw_File_Uploader

## Evaluation signals

- The aggregated results table contains exactly one row per deposited file — 6410 total (6048 Orbitrap Astral plus 362 from timsTOF Ultra and Orbitrap Exploris 480)
- Every DIA file was processed by DIA-NN and every DDA/WWA file by FragPipe, matching the metadata-derived assignment
- Each output row contains identification statistics (PSMs, peptides, protein groups passing the FDR filter)
- All files were processed automatically without incident over the evaluation window (e.g., the 26-day September 26 to October 21, 2024 period)
- Any processing failures surface in the Django Log Viewer viewed through the web interface rather than silently dropping files

## Limitations

- MSConnect supports third-party applications but does not include them; licensing requirements must be consulted with each engine's owners
- Deployment requires a server (an old PC suffices) with Docker-based installation; it is not a desktop application
- Third-party processing apps must be manually set up as wrappers/modules before auto-processing can begin
- No changelog is available for the repository, and the experimental design tracking module is not yet released

## Evidence
