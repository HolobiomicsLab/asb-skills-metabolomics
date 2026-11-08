---
name: proteomics-dda-dia-workflow-dispatch
description: Use when a large batch of raw LC-MS files carries mixed acquisition modes
  (DDA, DIA, or WWA) and metadata, and each file must be dispatched automatically
  to the matching search engine.
when_to_use_negative:
- You need a turnkey search engine itself — MSConnect is only a wrapper/interface;
  third-party software such as FragPipe and DIA-NN is not included and licensing must
  be checked separately
- Input is already processed identification or quantification results rather than
  raw MS files with acquisition-mode metadata
- You need a desktop, single-machine analysis tool — MSConnect is web-based and requires
  a server deployment (e.g., via Docker)
edam_operation: http://edamontology.org/operation_3767
edam_topics:
- http://edamontology.org/topic_0121
tools:
- name: MSConnect
  role: Web-based platform hosting the task queue and Omics Data Processor workers
    that dispatch each file to the correct search engine and collect results
  repo: https://github.com/RTKlab-BYU/MSConnect
- name: Raw_File_Uploader
  role: Resides on the mass spectrometer control computer and sends raw MS files plus
    metadata to the MSConnect Server, feeding the dispatch queue
  repo: https://github.com/RTKlab-BYU/Raw_File_Uploader
- name: Proteomics_Data_Processor
  role: Companion processor component that works with the MSConnect data manager to
    analyze MS-based proteomics data
  repo: https://github.com/RTKlab-BYU/Proteomics_Data_Processor
- name: FragPipe
  role: Search engine invoked via command line by an ODP worker for DDA and wide-window
    (WWA) acquisition files
- name: DIA-NN
  role: Search engine invoked via command line by an ODP worker for DIA acquisition
    files
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/acs.jproteome.4c00854
    title: 'MSConnect: Open-Source, End-to-End Platform for Automated Mass Spectrometry
      Data Management, Analysis, and Visualization'
  source_content_hash: sha256:424ca57fb8f16b034b0de047fad5d6d62757a487e5612de3959102b150cb1f11
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/RTKlab-BYU/MSConnect
  tools:
  - DIA-NN
  - FragPipe
  - MSConnect
  - PRIDE
  - Proteomics_Data_Processor
  - Raw_File_Uploader
  - msconvert
  edam_operation: http://edamontology.org/operation_3767
  edam_topics:
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# proteomics-dda-dia-workflow-dispatch

## Summary

Automated, mode-conditional routing of raw LC-MS files through MSConnect's Omics Data Processor so DDA and wide-window (WWA) acquisitions are searched with FragPipe while DIA acquisitions are searched with DIA-NN, enabling unattended high-throughput processing of thousands of files.

## When to use

Use when a large batch of raw LC-MS files carries mixed acquisition modes (DDA, DIA, or WWA) and metadata, and each file must be dispatched automatically to the matching search engine.

## When NOT to use

- You need a turnkey search engine itself — MSConnect is only a wrapper/interface; third-party software such as FragPipe and DIA-NN is not included and licensing must be checked separately
- Input is already processed identification or quantification results rather than raw MS files with acquisition-mode metadata
- You need a desktop, single-machine analysis tool — MSConnect is web-based and requires a server deployment (e.g., via Docker)

## Inputs

- Raw LC-MS data files (e.g., Thermo .raw for Orbitrap Astral / Exploris 480, Bruker timsTOF Ultra files) with associated metadata
- Metadata table assigning each file an acquisition mode (DDA, DIA, WWA) and instrument origin
- Search engine configuration files (FragPipe and DIA-NN parameter files)
- Deposited ProteomeXchange/PRIDE dataset (e.g., PXD037527)

## Outputs

- Per-file identification results (PSMs, peptides, and protein groups passing a given FDR filter) uploaded to the MSConnect server
- Aggregated results table with columns for file name, acquisition mode, engine used, and identification statistics

## How to apply

Download the deposited raw MS files plus the associated metadata table for the dataset (e.g., PXD037527 from the PRIDE partner repository), and read the metadata to assign each file its acquisition mode (DDA, DIA, or WWA) and its instrument origin (Orbitrap Astral, Orbitrap Exploris 480, or timsTOF Ultra). Configure one Omics Data Processor (ODP) worker per search engine — FragPipe/MSFragger for DDA and WWA, DIA-NN for DIA — as a processing app in the MSConnect Server queue. Submit one automated processing task per file so the ODP downloads the raw file plus its configuration file, invokes the engine's command line interface, and uploads the identification results back to the MSConnect server. Aggregate per-file outputs into a single table with file name, acquisition mode, engine used, and identification statistics. The rationale is that mode-conditional dispatch avoids manually shepherding files across programs, which the authors note is error prone and harms repeatability.

## Related tools

- **MSConnect** (Web-based platform hosting the task queue and Omics Data Processor workers that dispatch each file to the correct search engine and collect results) — https://github.com/RTKlab-BYU/MSConnect
- **Raw_File_Uploader** (Resides on the mass spectrometer control computer and sends raw MS files plus metadata to the MSConnect Server, feeding the dispatch queue) — https://github.com/RTKlab-BYU/Raw_File_Uploader
- **Proteomics_Data_Processor** (Companion processor component that works with the MSConnect data manager to analyze MS-based proteomics data) — https://github.com/RTKlab-BYU/Proteomics_Data_Processor
- **FragPipe** (Search engine invoked via command line by an ODP worker for DDA and wide-window (WWA) acquisition files)
- **DIA-NN** (Search engine invoked via command line by an ODP worker for DIA acquisition files)

## Evaluation signals

- The aggregated results table contains exactly one row per deposited file — 6410 total: 6048 Orbitrap Astral plus 362 from timsTOF Ultra and Orbitrap Exploris 480
- Every file's engine assignment matches its metadata acquisition mode: DDA/WWA files processed by FragPipe, DIA files processed by DIA-NN
- Per-file results include PSMs, peptides, and protein groups that pass the configured FDR filter
- Processing ran without incident across the full batch (no task-level errors; errors, if any, would surface in the Django Log Viewer web interface)
- All identification results were uploaded back to the MSConnect server by the ODP, not left on the processing worker

## Limitations

- MSConnect is not a standalone data analysis tool — it depends on third-party software for data processing/analysis and does not come with any processor, analysis software, or license
- The platform supports all third-party applications but does not include them; licensing requirements must be consulted with their respective owners
- An experimental design module for formally tracking experimental design and generating data packages was not yet released at publication time

## Evidence
