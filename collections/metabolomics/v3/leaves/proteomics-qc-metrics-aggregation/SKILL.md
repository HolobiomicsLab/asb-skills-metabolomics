---
name: proteomics-qc-metrics-aggregation
description: Use when raw MS data files (e.g., Thermo Orbitrap or Bruker timsTOF output)
  have been transferred from an instrument control computer to a server, and you need
  an automated per-upload QC summary of identification counts.
when_to_use_negative:
- Input is already an aggregated QC summary or a dashboard-ready table rather than
  per-file analysis rows
- MSConnect is being used purely for data upload/backup with no processing workflow
  configured — it is a wrapper/interface and depends on third-party software for analysis
- The goal is downstream differential expression, gene ontology, or pathway analysis
  rather than basic spectrum identification QC tabulation
edam_topics:
- http://edamontology.org/topic_0121
tools:
- name: MSConnect
  role: Platform that runs the automated QC script on file transfer, hosts the dashboard
    where QC results are posted, and allows user customization of the QC script
  repo: https://github.com/RTKlab-BYU/MSConnect
- name: Proteomics_Data_Processor
  role: Processor component that works with MSConnect to analyze MS-based proteomics
    data during the automated processing pipeline
  repo: https://github.com/RTKlab-BYU/Proteomics_Data_Processor
- name: Raw_File_Uploader
  role: Resides on the mass spectrometer control computer and sends MS data files
    and metadata to the MSConnect server, triggering the QC summarization
  repo: https://github.com/RTKlab-BYU/Raw_File_Uploader
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1021/acs.jproteome.4c00854
    title: 'MSConnect: Open-Source, End-to-End Platform for Automated Mass Spectrometry
      Data Management, Analysis, and Visualization'
  source_content_hash: sha256:38142d3c702b0541067566e1d57deed9c11f7a398622df334278ed6954a87249
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
yes_rate: 0.625
n_claims: 8
---

# proteomics-qc-metrics-aggregation

## Summary

Aggregates per-file proteomics QC signals into a tabulated summary of spectrum identification metrics (PSMs, peptides, and protein groups passing a given FDR filter), mirroring the automated QC summarization MSConnect runs when raw MS files are transferred from the instrument computer to server storage.

## When to use

Use when raw MS data files (e.g., Thermo Orbitrap or Bruker timsTOF output) have been transferred from an instrument control computer to a server, and you need an automated per-upload QC summary of identification counts.

## When NOT to use

- Input is already an aggregated QC summary or a dashboard-ready table rather than per-file analysis rows
- MSConnect is being used purely for data upload/backup with no processing workflow configured — it is a wrapper/interface and depends on third-party software for analysis
- The goal is downstream differential expression, gene ontology, or pathway analysis rather than basic spectrum identification QC tabulation

## Inputs

- per-file analysis table produced by task_001 (task_001/per_file_analysis_table)
- raw MS data files transferred from the instrument computer (converted to mzML during QC)
- parsed instrument settings from the raw files
- FDR filter threshold used for spectrum identification counts

## Outputs

- qc_summary_table.csv with one row per aggregated QC metric (counts of PSMs, peptides, protein groups)
- chromatogram plots (extracted ion, total ion, or base peak chromatogram) per file
- QC summary results posted to the MSConnect dashboard

## How to apply

Ingest the per-file analysis table produced by the earlier ingestion step (task_001/per_file_analysis_table) as the QC input. For each file, the default MSConnect QC script converts the raw file to mzML, parses instrument settings, and generates a chromatogram (extracted ion, total ion, or base peak chromatogram) alongside identification statistics. Aggregate the per-file rows by counting peptide-spectrum matches (PSMs), peptides, and protein groups that pass a given false discovery rate (FDR) filter, and write one row per aggregated metric to qc_summary_table.csv. Write the output so the summarization can be re-run whenever new files are uploaded, and post the results to the dashboard; the QC script is customizable, so swap in lab-specific metrics as needed.

## Related tools

- **MSConnect** (Platform that runs the automated QC script on file transfer, hosts the dashboard where QC results are posted, and allows user customization of the QC script) — https://github.com/RTKlab-BYU/MSConnect
- **Proteomics_Data_Processor** (Processor component that works with MSConnect to analyze MS-based proteomics data during the automated processing pipeline) — https://github.com/RTKlab-BYU/Proteomics_Data_Processor
- **Raw_File_Uploader** (Resides on the mass spectrometer control computer and sends MS data files and metadata to the MSConnect server, triggering the QC summarization) — https://github.com/RTKlab-BYU/Raw_File_Uploader

## Evaluation signals

- qc_summary_table.csv exists, is parseable as CSV, and contains field entries for PSMs, peptides, and protein groups
- Aggregated counts in qc_summary_table.csv are consistent with the per-file values from the input task_001/per_file_analysis_table table
- Identification counts reflect only PSMs, peptides, and protein groups passing the specified FDR filter
- Chromatogram output (XIC, TIC, or BPC) is generated from the mzML-converted file along with parsed instrument settings
- QC summary results appear on the MSConnect dashboard and the summarization can be re-run when new files are uploaded

## Limitations

- The default QC script tabulates only basic spectrum identification information; richer metrics require user customization of the QC script
- MSConnect itself is not a standalone data analysis tool — it depends on third-party software for data processing/analysis and provides a wrapper/interface to interact with them
- Third-party applications are not included with MSConnect and licensing requirements should be consulted with their respective owners

## Evidence
