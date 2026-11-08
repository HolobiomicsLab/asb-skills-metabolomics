---
name: lc-ms-raw-file-batch-processing
description: Use when a study deposits or generates hundreds to thousands of vendor
  LC-MS raw files spanning multiple acquisition modes and instruments, and the goal
  is fully autonomous, mode-conditional identification without manual per-file intervention.
when_to_use_negative:
- MSConnect is not a standalone data analysis tool — if you need a specific search
  engine itself, deploy that engine directly rather than expecting MSConnect to provide
  analysis software or licenses
- If the data set is small (a handful of files) or already reduced to feature tables
  / identification matrices, a queue-driven batch pipeline adds unnecessary infrastructure
  overhead
- If your instruments or acquisition modes require a search engine for which no ODP
  processing app/wrapper has been set up, first follow the processing-module/wrapper
  setup tutorial before attempting automated submission
edam_operation: http://edamontology.org/operation_3637
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_3520
tools:
- name: MSConnect
  role: Web-based server platform that hosts the processing queue and Omics Data Processor
    apps; ODP workers automatically download tasks from the queue, download the raw
    plus configuration files, invoke the engine CLI, and upload results back to the
    server
  repo: https://github.com/RTKlab-BYU/MSConnect
- name: FragPipe
  role: Search engine invoked via its command line interface by the ODP for DDA and
    wide-window DDA files
- name: DIA-NN
  role: Search engine invoked via its command line interface by the ODP for DIA files
- name: Raw_File_Uploader
  role: Companion Windows tool residing on the instrument control computer that sends
    MS data files and associated metadata to the MSConnect Server, initiating the
    batch
  repo: https://github.com/RTKlab-BYU/Raw_File_Uploader
- name: Proteomics_Data_Processor
  role: Companion data-processing tool used with the platform for analyzing MS-based
    proteomics data
  repo: https://github.com/RTKlab-BYU/Proteomics_Data_Processor
- name: PRIDE
  role: Partner repository from which the deposited raw files and metadata for validation
    dataset PXD037527 are downloaded
- name: msconvert
  role: File-conversion utility listed in the source card's tool set for the processing
    workflow
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/acs.jproteome.4c00854
    title: 'MSConnect: Open-Source, End-to-End Platform for Automated Mass Spectrometry
      Data Management, Analysis, and Visualization'
  source_content_hash: sha256:9f6ca9d2dc65d7cdf1fb61651cc6c210f59d6d80bd25bec1320b218304f6f1a6
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
  edam_operation: http://edamontology.org/operation_3637
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# lc-ms-raw-file-batch-processing

## Summary

Batch process thousands of vendor LC-MS raw files end-to-end by routing each file through mode-appropriate search engines (FragPipe/MSFragger for DDA and wide-window DDA, DIA-NN for DIA) inside a queue-driven platform that automatically downloads raws from a server, invokes the engine CLI, and returns PSM/peptide/protein-group identifications.

## When to use

Use when a study deposits or generates hundreds to thousands of vendor LC-MS raw files spanning multiple acquisition modes and instruments, and the goal is fully autonomous, mode-conditional identification without manual per-file intervention.

## When NOT to use

- MSConnect is not a standalone data analysis tool — if you need a specific search engine itself, deploy that engine directly rather than expecting MSConnect to provide analysis software or licenses
- If the data set is small (a handful of files) or already reduced to feature tables / identification matrices, a queue-driven batch pipeline adds unnecessary infrastructure overhead
- If your instruments or acquisition modes require a search engine for which no ODP processing app/wrapper has been set up, first follow the processing-module/wrapper setup tutorial before attempting automated submission

## Inputs

- Vendor LC-MS raw files (from Orbitrap Astral, Orbitrap Exploris 480, timsTOF Ultra)
- Metadata table mapping each file to acquisition mode (DDA / DIA / WWA) and instrument origin
- Dataset accession from the PRIDE partner repository (e.g., PXD037527)
- Per-engine configuration files accompanying each raw file

## Outputs

- Per-file identification results: PSMs, peptides, and protein groups passing a given false discovery rate (FDR) filter
- Aggregated table with columns for file name, acquisition mode, engine used, and identification statistics
- One processing-task record per file in the MSConnect Server queue showing engine invoked and completion status

## How to apply

First, download the deposited raw MS files and associated metadata (e.g., dataset PXD037527 from the PRIDE partner repository) and read the metadata table to assign each file its acquisition mode — data-dependent acquisition (DDA), data-independent acquisition (DIA), or wide-window DDA (WWA) — and its instrument origin (Orbitrap Exploris 480, Orbitrap Astral, or timsTOF Ultra). Second, configure one Omics Data Processor (ODP) worker per search engine as a processing app in the MSConnect Server queue, e.g., FragPipe/MSFragger for DDA and WWA and DIA-NN for DIA. Third, submit one automated processing task per file so the ODP automatically downloads user-defined tasks from the queue, downloads the raw plus configuration file, invokes the engine's command line interface, and uploads identification results (PSMs, peptides, and protein groups passing a given false discovery rate (FDR) filter) to the MSConnect server. The rationale is that manually shepherding data across multiple programs and in-house scripts is error prone and labor intensive; a queue-driven, vendor-agnostic platform that shepherds data from collection through analysis in a fully automated fashion removes that manual transfer while keeping the workflow traceable. Finally, aggregate the per-file results into a single table with columns for file name, acquisition mode, engine used, and identification statistics, and verify the table contains one row per deposited file (e.g., 6410 total: 6048 Orbitrap Astral plus 362 from timsTOF Ultra and Orbitrap Exploris 480) to confirm all files were processed without incident.

## Related tools

- **MSConnect** (Web-based server platform that hosts the processing queue and Omics Data Processor apps; ODP workers automatically download tasks from the queue, download the raw plus configuration files, invoke the engine CLI, and upload results back to the server) — https://github.com/RTKlab-BYU/MSConnect
- **FragPipe** (Search engine invoked via its command line interface by the ODP for DDA and wide-window DDA files)
- **DIA-NN** (Search engine invoked via its command line interface by the ODP for DIA files)
- **Raw_File_Uploader** (Companion Windows tool residing on the instrument control computer that sends MS data files and associated metadata to the MSConnect Server, initiating the batch) — https://github.com/RTKlab-BYU/Raw_File_Uploader
- **Proteomics_Data_Processor** (Companion data-processing tool used with the platform for analyzing MS-based proteomics data) — https://github.com/RTKlab-BYU/Proteomics_Data_Processor
- **PRIDE** (Partner repository from which the deposited raw files and metadata for validation dataset PXD037527 are downloaded)
- **msconvert** (File-conversion utility listed in the source card's tool set for the processing workflow)

## Evaluation signals

- Aggregated result table contains exactly one row per deposited file — e.g., 6410 rows total: 6048 Orbitrap Astral plus 362 from timsTOF Ultra and Orbitrap Exploris 480 — confirming all files were automatically analyzed without incident
- Every file's assignment of search engine matches its recorded acquisition mode: DDA and WWA files routed to FragPipe, DIA files routed to DIA-NN
- Identification outputs per file include PSM, peptide, and protein-group counts passing the specified FDR filter (no missing identification blocks for successfully completed tasks)
- Logs show the ODP downloading each raw plus configuration file from the queue, invoking the engine's CLI, and uploading results back to the MSConnect server — traceable end-to-end
- No error entries surfaced in the Django Log Viewer (https://pypi.org/project/django-log-viewer/) web interface for the batch

## Limitations

- Third-party search engines are not included with the platform; licensing requirements must be consulted with their respective owners, and MSConnect provides only a wrapper/interface to interact with them
- MSConnect is not a desktop application — it is web-based and requires a server deployment (via Docker, an old PC can suffice)
- Setting up a new processing module/wrapper for an engine requires following the 'Processing module/wrapper setup' tutorial before automated processing can run
- No changelog was found for the repository, and the experimental-design module for formally tracking experimental design and generating data packages was not yet released

## Evidence

- [results] Mode-conditional engine routing: [evidence span withheld — anchor status: not_found]
