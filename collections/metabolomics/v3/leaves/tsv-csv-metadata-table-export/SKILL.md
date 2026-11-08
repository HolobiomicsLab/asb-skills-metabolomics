---
name: tsv-csv-metadata-table-export
description: Use when RAW-file acquisition metadata must be extracted from Thermo
  .RAW mass spectrometry files into structured, analysis-ready TSV/CSV tables for
  QC, provenance, repository submission, or downstream computational workflows.
when_to_use_negative:
- Input is not a Thermo Fisher RAW file — MetaXtract currently supports only Thermo
  RAW; other vendor formats are not supported yet
- A fully graphical/interactive inspection session with Plotly HTML reports or cross-sample
  multi_comparison plots is the primary goal rather than structured TSV/CSV metadata
  export
- MS-method or LC-method extraction is required and the run environment is a Linux
  container — these Thermo method-reading calls are unsupported on Linux
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_3520
tools:
- name: MetaXtract
  role: Primary tool that reads Thermo RAW files via RawFileReader and writes the
    structured TSV/CSV metadata tables (file-based details, MS1/MS2 scan headers,
    technical details)
  repo: https://github.com/Rappsilber-Laboratory/MetaXtract
- name: RawFileReader
  role: Underlying Thermo Fisher C# library used by MetaXtract to parse RAW files
    before transformation into TSV/CSV
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1101/2025.11.12.687968
    title: 'MetaXtract: Extracting Metadata from Raw Files for FAIR Data Practices
      and Workflow Optimisation'
  source_content_hash: sha256:c8fa741704edb8c3c5e06bef92019dc4851fa39e4566f7ebfb6a51d90781af4b
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/Rappsilber-Laboratory/MetaXtract
  repo_url: https://github.com/Rappsilber-Laboratory/MetaXtract
  tools:
  - MetaXtract
  - RawFileReader
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# tsv-csv-metadata-table-export

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Export Thermo RAW-file acquisition metadata (instrument details, scan counts, run statistics, sample information, MS1/MS2 scan headers, and technical trailer details) into structured TSV/CSV tables using MetaXtract, enabling FAIR, analysis-ready downstream use.

## When to use

Use when RAW-file acquisition metadata must be extracted from Thermo .RAW mass spectrometry files into structured, analysis-ready TSV/CSV tables for QC, provenance, repository submission, or downstream computational workflows.

## When NOT to use

- Input is not a Thermo Fisher RAW file — MetaXtract currently supports only Thermo RAW; other vendor formats are not supported yet
- A fully graphical/interactive inspection session with Plotly HTML reports or cross-sample multi_comparison plots is the primary goal rather than structured TSV/CSV metadata export
- MS-method or LC-method extraction is required and the run environment is a Linux container — these Thermo method-reading calls are unsupported on Linux

## Inputs

- Thermo Fisher .RAW mass spectrometry files (e.g. downloaded from PRIDE or local acquisitions)
- MetaXtract YAML configuration file (config.yml) specifying io.input paths, output_dir, and per-output toggles

## Outputs

- File-based details TSV containing instrument details, scan counts, run statistics, and sample information per RAW file
- MS1 and MS2 scan-header TSV/CSV tables (complete or column-selected, PSI-MS/mzML-style column names)
- Per-MS2-scan technical details CSV files (*_technical_details_ms2_*.csv) when ms2_technical_details_export is enabled
- metadata.sdrf.tsv or metadata.sdrf.draft.tsv when the sdrf block is configured

## How to apply

Run MetaXtract in CLI mode with a YAML config (or via the Docker/Apptainer container) pointing io.input at one or more Thermo .RAW files and io.output_dir at a root folder that receives one subfolder per RAW file. Enable outputs.file_based_details to write a TSV of instrument details, scan counts, run statistics, and sample information; enable scan_header.MS1 and scan_header.MS2 (e.g. select_all: true) to export complete scan-header tables as TSV/CSV, choosing either individual PSI-MS/mzML-style columns or the full set. Keep outputs.ms_method and outputs.lc_method set to false when running in a Linux container, since Thermo method-reading calls are unsupported there. Optionally enable ms2_technical_details_export via GetMoreMSInfos to write *_technical_details_ms2_*.csv per MS2 scan, and set sdrf.draft: true if a metadata.sdrf.draft.tsv with RAW-derived fields is needed. Validate that each expected TSV/CSV (file-based details plus MS1/MS2 scan-header tables) exists, is non-empty, and that column names follow the PSI-MS/mzML conventions documented in the README (e.g. Scan Start Time (min), Total Ion Current, Base Peak m/z, Ion Injection Time (ms), with thermo_-prefixed trailer fields).

## Related tools

- **MetaXtract** (Primary tool that reads Thermo RAW files via RawFileReader and writes the structured TSV/CSV metadata tables (file-based details, MS1/MS2 scan headers, technical details)) — https://github.com/Rappsilber-Laboratory/MetaXtract
- **RawFileReader** (Underlying Thermo Fisher C# library used by MetaXtract to parse RAW files before transformation into TSV/CSV)

## Examples

```
docker run --rm -v "$(pwd)/data:/data:ro" -v "$(pwd)/out:/out" metaxtract:latest --input /data/sample.raw --output-dir /out --file-based-details --complete-ms1 --complete-ms2
```

## Evaluation signals

- For each RAW file, a per-file subfolder exists under output_dir containing the file-based details TSV and the expected MS1/MS2 scan-header TSV/CSV files, all non-empty
- Column headers in exported tables match the documented PSI-MS/mzML names (e.g. 'Scan Start Time (min)', 'Total Ion Current', 'Base Peak m/z', 'Ion Injection Time (ms)') with thermo_-prefixed names for unmatched Thermo trailer fields
- When scan_header.select_all is true, the complete MS1 or MS2 column set is present rather than only the individually listed columns
- In Linux container runs, ms_method and lc_method outputs are absent (or were configured false) and no Thermo method-reading errors occur
- When sdrf.draft is enabled, metadata.sdrf.draft.tsv is produced with only RAW-derived fields filled and can be completed afterwards

## Limitations

- Currently supports only Thermo Fisher RAW files; additional vendor file formats are not yet supported
- MS-method and LC-method exports must be disabled on Linux containers because those Thermo method-reading calls are not supported on Linux
- mzIdentML 1.3 does not report critical acquisition parameters such as LC-MS method settings and scan-level metrics, so LC/MS method metadata must come from the RAW files themselves

## Evidence
