---
name: ms-file-analysis-table-parsing
description: Use when per-file analysis rows for transferred MS data (e.g. task_001/per_file_analysis_table)
  need to be rolled up into a QC summary of PSM, peptide and protein group counts
  at a given FDR filter, posted to a dashboard.
when_to_use_negative:
- Input is raw MS data (e.g. Thermo RAW or mzML) rather than an already-produced per-file
  analysis table — conversion and parsing are upstream steps
- A full third-party search/analysis engine run is required — MSConnect is a wrapper/interface
  and 'does not come with any data processor or analysis software/license'
- Quantitative downstream analysis (imputing missing values, normalization, differential
  expression) is the goal rather than identification-count QC summarization
edam_operation: http://edamontology.org/operation_0279
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_0092
tools:
- name: MSConnect
  role: Generates the automated QC summary on raw file transfer, posts results to
    the dashboard, and hosts the customizable QC script
  repo: https://github.com/RTKlab-BYU/MSConnect
- name: Raw File Uploader
  role: Uploads raw MS files and associated metadata from the instrument control computer,
    triggering the QC summarization pipeline
  repo: https://github.com/RTKlab-BYU/Raw_File_Uploader
- name: Proteomics Data Processor
  role: Processing component that runs user-defined tasks (including the QC script)
    and uploads results to the MSConnect server
  repo: https://github.com/RTKlab-BYU/Proteomics_Data_Processor
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1021/acs.jproteome.4c00854
    title: 'MSConnect: Open-Source, End-to-End Platform for Automated Mass Spectrometry
      Data Management, Analysis, and Visualization'
  source_content_hash: sha256:865258aa03e3467021c77816da41a8f446ccaff746fc592915a20c1a1fa315b2
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/RTKlab-BYU/MSConnect
  tools:
  - MSConnect
  - Proteomics Data Processor
  - Raw File Uploader
  edam_operation: http://edamontology.org/operation_0279
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# ms-file-analysis-table-parsing

## Summary

Aggregates per-file MS analysis results into a tabulated QC summary of basic spectrum identification information — PSMs, peptides and protein groups passing a given FDR filter — mirroring the automated QC artifact MSConnect generates after raw files are transferred to storage.

## When to use

Use when per-file analysis rows for transferred MS data (e.g. task_001/per_file_analysis_table) need to be rolled up into a QC summary of PSM, peptide and protein group counts at a given FDR filter, posted to a dashboard.

## When NOT to use

- Input is raw MS data (e.g. Thermo RAW or mzML) rather than an already-produced per-file analysis table — conversion and parsing are upstream steps
- A full third-party search/analysis engine run is required — MSConnect is a wrapper/interface and 'does not come with any data processor or analysis software/license'
- Quantitative downstream analysis (imputing missing values, normalization, differential expression) is the goal rather than identification-count QC summarization

## Inputs

- per_file_analysis_table (CSV of per-file analysis results, e.g. task_001/per_file_analysis_table)
- FDR filter threshold used to count PSMs, peptides and protein groups
- contextual per-file instrument metadata (instrument settings parsed from mzML conversion)

## Outputs

- qc_summary_table.csv with one row per aggregated QC metric (counts of PSMs, peptides and protein groups passing the FDR filter)
- QC summary entry posted to the MSConnect dashboard

## How to apply

Ingest the per-file analysis table produced by the prior analysis step as the QC input data, then aggregate its rows into counts of peptide-spectrum matches (PSMs), peptides and protein groups passing a given false discovery rate (FDR) filter. Write the aggregated metrics to qc_summary_table.csv with one row per QC metric (PSM count, peptide count, protein group count) so the summarization is idempotent and can be re-run whenever new files are uploaded. In MSConnect this aggregation is triggered automatically on file transfer — the default QC pipeline converts the raw file to mzML, parses instrument settings, creates an extracted ion, total ion or base peak chromatogram, and tabulates the identification counts, posting results to the dashboard. The QC script is user-customizable, so the FDR threshold and which metrics are tabulated can be adapted to a specific workflow.

## Related tools

- **MSConnect** (Generates the automated QC summary on raw file transfer, posts results to the dashboard, and hosts the customizable QC script) — https://github.com/RTKlab-BYU/MSConnect
- **Raw File Uploader** (Uploads raw MS files and associated metadata from the instrument control computer, triggering the QC summarization pipeline) — https://github.com/RTKlab-BYU/Raw_File_Uploader
- **Proteomics Data Processor** (Processing component that runs user-defined tasks (including the QC script) and uploads results to the MSConnect server) — https://github.com/RTKlab-BYU/Proteomics_Data_Processor

## Examples

```
import pandas as pd; df = pd.read_csv('task_001/per_file_analysis_table.csv'); df.groupby('qc_metric').sum().to_csv('qc_summary_table.csv')
```

## Evaluation signals

- Generated file qc_summary_table.csv exists and is parseable as CSV
- File contains field entries for PSMs, peptides and protein groups consistent with aggregated per-file values from the input table
- Each aggregated count equals the sum of per-file rows passing the stated FDR filter (one row per QC metric)
- Re-running the summary after additional file uploads yields updated counts without duplication (idempotent one-row-per-metric schema)
- Summary is visible/posted on the MSConnect dashboard after processing

## Limitations

- MSConnect is not a standalone data analysis tool — QC tabulation depends on third-party software results; 'while the platform supports all 3rd party applications, they are not included and licensing requirements should be consulted with their respective owners'
- The default QC pipeline's exact output contents depend on the user-customizable QC script, so counts may vary between deployments
- Errors during automated processing are logged to the Django Log Viewer and viewed through the web interface, requiring manual inspection on failure

## Evidence
