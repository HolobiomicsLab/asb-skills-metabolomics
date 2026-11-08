---
name: omics-processing-queue-automation
description: Use when hundreds to thousands of acquired or deposited LC-MS raw files
  spanning mixed acquisition modes (DDA, DIA, wide-window DDA) and instruments (e.g.,
  Orbitrap Astral, timsTOF Ultra, Orbitrap Exploris 480) must be automatically routed
  to appropriate search engines via a processing queue.
when_to_use_negative:
- MSConnect is not a standalone data analysis tool — if you need the search engine
  itself or lack the third-party software licenses (FragPipe, DIA-NN), the queue provides
  only a wrapper/interface and cannot process files
- Input is already processed identification results and no raw-file search is required
  — the queue's task is to drive raw files through search engines, not to re-tabulate
  existing outputs
- Only a handful of files need ad-hoc desktop analysis — MSConnect is a web-based
  platform requiring a server (Docker deployment), which is overkill for single-file
  interactive work
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0593
tools:
- name: MSConnect
  role: Web-based server platform hosting the processing queue into which mode-specific
    ODP worker apps are configured and from which results are collected
  repo: https://github.com/RTKlab-BYU/MSConnect
- name: FragPipe
  role: Search engine invoked via its command-line interface by an ODP worker for
    DDA and wide-window DDA (WWA) raw files
- name: DIA-NN
  role: Search engine invoked via its command-line interface by an ODP worker for
    data-independent acquisition (DIA) raw files
- name: Raw_File_Uploader
  role: Resides on the mass spectrometer control computer and sends MS data files
    and associated metadata to the MSConnect Server so tasks can be queued
  repo: https://github.com/RTKlab-BYU/Raw_File_Uploader
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/acs.jproteome.4c00854
    title: 'MSConnect: Open-Source, End-to-End Platform for Automated Mass Spectrometry
      Data Management, Analysis, and Visualization'
  source_content_hash: sha256:955919049b93a50b19e6d1bc8b9a85e33948bd9c38c8b872d50787291213f84f
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
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0593
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# omics-processing-queue-automation

## Summary

Automated, queue-driven batch processing of high-throughput LC-MS raw files through the MSConnect Server, where Omics Data Processor (ODP) workers pull per-file tasks and invoke mode-matched search engines (FragPipe for DDA/WWA, DIA-NN for DIA) without manual intervention. This replaces error-prone manual shepherding of files across programs and scripts.

## When to use

Use when hundreds to thousands of acquired or deposited LC-MS raw files spanning mixed acquisition modes (DDA, DIA, wide-window DDA) and instruments (e.g., Orbitrap Astral, timsTOF Ultra, Orbitrap Exploris 480) must be automatically routed to appropriate search engines via a processing queue.

## When NOT to use

- MSConnect is not a standalone data analysis tool — if you need the search engine itself or lack the third-party software licenses (FragPipe, DIA-NN), the queue provides only a wrapper/interface and cannot process files
- Input is already processed identification results and no raw-file search is required — the queue's task is to drive raw files through search engines, not to re-tabulate existing outputs
- Only a handful of files need ad-hoc desktop analysis — MSConnect is a web-based platform requiring a server (Docker deployment), which is overkill for single-file interactive work

## Inputs

- Raw LC-MS data files (e.g., Thermo .raw from Orbitrap Astral / Exploris 480, Bruker timsTOF files)
- Metadata table mapping each file to acquisition mode (DDA, DIA, WWA) and instrument origin
- Per-file search-engine configuration files (FragPipe and DIA-NN parameter files)
- Dataset accession from a repository such as PRIDE (e.g., PXD037527)

## Outputs

- Per-file identification results: PSMs, peptides, and protein groups passing a given FDR filter, uploaded to the MSConnect server
- Aggregated table with file name, acquisition mode, engine used, and identification statistics
- Logged processing errors viewable through the web interface

## How to apply

Download the deposited raw MS files and metadata (e.g., dataset PXD037527 from the PRIDE partner repository) and read the metadata table to assign each file its acquisition mode (DDA, DIA, or WWA) and instrument origin. Configure one Omics Data Processor worker per search engine — FragPipe/MSFragger for DDA and WWA, DIA-NN for DIA — as a processing app in the MSConnect Server queue, installing wrappers via the apps marketplace the way one downloads a mobile app. Submit one automated processing task per file so the ODP downloads each raw file plus its configuration file, invokes the engine's command-line interface, and uploads identification results (PSMs, peptides, protein groups passing the FDR filter) back to the MSConnect server. Aggregate per-file results into a single table with columns for file name, acquisition mode, engine used, and identification statistics. Rationale: manual transfer across programs is error prone and poorly repeatable, whereas the queue transparently documents every processing step for facile reanalysis.

## Related tools

- **MSConnect** (Web-based server platform hosting the processing queue into which mode-specific ODP worker apps are configured and from which results are collected) — https://github.com/RTKlab-BYU/MSConnect
- **FragPipe** (Search engine invoked via its command-line interface by an ODP worker for DDA and wide-window DDA (WWA) raw files)
- **DIA-NN** (Search engine invoked via its command-line interface by an ODP worker for data-independent acquisition (DIA) raw files)
- **Raw_File_Uploader** (Resides on the mass spectrometer control computer and sends MS data files and associated metadata to the MSConnect Server so tasks can be queued) — https://github.com/RTKlab-BYU/Raw_File_Uploader

## Evaluation signals

- Aggregated table contains exactly one row per deposited file — 6410 total (6048 Orbitrap Astral plus 362 from timsTOF Ultra and Orbitrap Exploris 480)
- Every file completed processing without incident across the 26-day evaluation period (September 26 to October 21, 2024)
- Each output file reports PSMs, peptides, and protein groups passing the defined FDR filter
- Each row records the correct engine-to-mode assignment: DIA-NN for DIA files, FragPipe for DDA/WWA files
- No unresolved errors in the Django Log Viewer web interface during the run

## Limitations

- MSConnect does not come with any data processor or analysis software or licenses — third-party tools must be installed and licensed separately
- The Raw File Uploader and Proteomics Data Processor components are built on C# with .NET 4.5.1 for the Windows platform
- Experimental design module not yet released, so formal tracking of experimental design is not yet automated
- No changelog was found for the repository, complicating version traceability of the deployment

## Evidence
