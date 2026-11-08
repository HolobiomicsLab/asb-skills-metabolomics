---
name: psm-peptide-protein-group-summarization
description: Use when raw MS files (e.g., from Orbitrap Astral, timsTOF Ultra, or
  Orbitrap Exploris 480) have been transferred to storage and you need an automated
  QC summary of identification counts per file.
when_to_use_negative:
- Input is not raw MS data or per-file identification results — MSConnect is not a
  standalone data analysis tool and depends on third-party software for data processing/analysis
- You need peptide/protein identification itself (search scoring) rather than summarizing
  identification counts that already exist in the per-file table
- The goal is deep downstream statistics (e.g., differential expression, gene ontology
  or pathway analysis) rather than a post-transfer QC summary
edam_topics:
- http://edamontology.org/topic_0121
tools:
- name: MSConnect
  role: Web-based platform that hosts the automated QC summarization workflow and
    posts results to its dashboard; customizable by users
  repo: https://github.com/RTKlab-BYU/MSConnect
- name: Raw_File_Uploader
  role: Resides on the mass spectrometer control computer and sends MS data files
    and associated metadata to the MSConnect Server, triggering the QC summarization
    on transfer
  repo: https://github.com/RTKlab-BYU/Raw_File_Uploader
- name: Proteomics_Data_Processor
  role: 'Processor tool through which QC/processing tasks run: automatically downloads
    user-defined tasks from the queue, processes them with the specified software,
    and uploads results to the MSConnect server'
  repo: https://github.com/RTKlab-BYU/Proteomics_Data_Processor
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1021/acs.jproteome.4c00854
    title: 'MSConnect: Open-Source, End-to-End Platform for Automated Mass Spectrometry
      Data Management, Analysis, and Visualization'
  source_content_hash: sha256:deb39b3f93d23c93d43cd2ae9de33de996462995ed520890c869b167d8d2384a
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/RTKlab-BYU/MSConnect
  tools:
  - MSConnect
  - Proteomics_Data_Processor
  - Raw_File_Uploader
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6153846153846154
n_claims: 13
---

# psm-peptide-protein-group-summarization

## Summary

Generates the automated quality-control summary that MSConnect produces after raw MS data files are transferred from the instrument computer to storage, tabulating per-file spectrum identification counts (PSMs, peptides, protein groups passing an FDR filter) and posting results to the dashboard.

## When to use

Use when raw MS files (e.g., from Orbitrap Astral, timsTOF Ultra, or Orbitrap Exploris 480) have been transferred to storage and you need an automated QC summary of identification counts per file.

## When NOT to use

- Input is not raw MS data or per-file identification results — MSConnect is not a standalone data analysis tool and depends on third-party software for data processing/analysis
- You need peptide/protein identification itself (search scoring) rather than summarizing identification counts that already exist in the per-file table
- The goal is deep downstream statistics (e.g., differential expression, gene ontology or pathway analysis) rather than a post-transfer QC summary

## Inputs

- Per-file analysis table (task_001/per_file_analysis_table) from raw MS data files transferred from the instrument computer
- User-defined FDR filter threshold

## Outputs

- qc_summary_table.csv with one row per aggregated QC metric (counts of PSMs, peptides, protein groups)
- QC summary results posted to the MSConnect dashboard

## How to apply

Ingest the per-file analysis table produced by the upstream task (task_001/per_file_analysis_table) as the QC input. Aggregate per-file rows into a tabulated summary of basic spectrum identification information, counting peptide-spectrum matches (PSMs), peptides, and protein groups that pass the chosen false discovery rate (FDR) filter. Write the aggregated QC summary to qc_summary_table.csv with one row per aggregated QC metric so it can be re-run whenever new files are uploaded. On the MSConnect platform, this QC step is triggered automatically after the Raw File Uploader sends data files and metadata from the instrument control computer, with the Omics Data Processor downloading queued tasks, running the processing software, and uploading results to the MSConnect server for dashboard display.

## Related tools

- **MSConnect** (Web-based platform that hosts the automated QC summarization workflow and posts results to its dashboard; customizable by users) — https://github.com/RTKlab-BYU/MSConnect
- **Raw_File_Uploader** (Resides on the mass spectrometer control computer and sends MS data files and associated metadata to the MSConnect Server, triggering the QC summarization on transfer) — https://github.com/RTKlab-BYU/Raw_File_Uploader
- **Proteomics_Data_Processor** (Processor tool through which QC/processing tasks run: automatically downloads user-defined tasks from the queue, processes them with the specified software, and uploads results to the MSConnect server) — https://github.com/RTKlab-BYU/Proteomics_Data_Processor

## Evaluation signals

- The generated qc_summary_table.csv file exists and is parseable as CSV
- The CSV contains field entries for PSMs, peptides and protein groups
- Reported counts are consistent with the aggregated per-file values from the input per-file analysis table
- Counts reflect a single, explicitly stated FDR filter applied uniformly across files
- QC results are visible/posted on the MSConnect dashboard after the processing task completes

## Limitations

- MSConnect is not a standalone data analysis tool: the QC summarization depends on third-party search/processing software, which is not included and whose licensing must be consulted with the respective owners
- The QC script is customizable by users, so default outputs (counts and thresholds) may differ between deployments and must be recorded for reproducibility
- The platform does not ship processing software or licenses — it provides only a wrapper/interface, so a working processing workflow (wrapper setup) must be configured before automated QC can run

## Evidence
