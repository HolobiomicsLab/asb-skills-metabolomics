---
name: pride-repository-raw-file-retrieval
description: Use when RAW files deposited in a public repository such as PRIDE must
  be fetched automatically by accession (e.g. PXD060351) and annotated with acquisition
  metadata in a reproducible, containerizable batch pipeline.
when_to_use_negative:
- Input files are not Thermo Fisher RAW files — MetaXtract currently supports only
  Thermo RAW; additional vendor file formats are not yet supported
- The run requires extraction of MS or LC methods on a Linux container, since ms_method
  and lc_method exports are not supported in that mode
- The task needs interactive GUI-based inspection rather than automated, reproducible
  batch retrieval and annotation
edam_operation: http://edamontology.org/operation_2422
edam_topics:
- http://edamontology.org/topic_0121
tools:
- name: MetaXtract
  role: Performs the end-to-end metadata annotation of downloaded RAW files, extracting
    instrument-, run-, and scan-level metadata into structured TSV/CSV via its CLI
    or Linux-container mode
  repo: https://github.com/Rappsilber-Laboratory/MetaXtract
- name: Snakemake
  role: Orchestrates the automated PRIDE download (by year and file count) and per-file
    MetaXtract annotation steps
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1101/2025.11.12.687968
    title: 'MetaXtract: Extracting Metadata from Raw Files for FAIR Data Practices
      and Workflow Optimisation'
  source_content_hash: sha256:29c11c26a2e7fb8d2e438d2d2b35e07a63be968457f92f3ef6916f4560044749
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
  - Snakemake
  edam_operation: http://edamontology.org/operation_2422
  edam_topics:
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# pride-repository-raw-file-retrieval

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A Snakemake-based, fully open-source workflow shipped in the MetaXtract GitHub repository that automatically downloads Thermo RAW files from PRIDE (selected by year and file count) and runs MetaXtract's end-to-end metadata annotation, exporting instrument-, run-, and scan-level metadata into structured TSV/CSV.

## When to use

Use when RAW files deposited in a public repository such as PRIDE must be fetched automatically by accession (e.g. PXD060351) and annotated with acquisition metadata in a reproducible, containerizable batch pipeline.

## When NOT to use

- Input files are not Thermo Fisher RAW files — MetaXtract currently supports only Thermo RAW; additional vendor file formats are not yet supported
- The run requires extraction of MS or LC methods on a Linux container, since ms_method and lc_method exports are not supported in that mode
- The task needs interactive GUI-based inspection rather than automated, reproducible batch retrieval and annotation

## Inputs

- PRIDE accession(s) (e.g. PXD060351, PXD060271, PXD060112) with year and file-count selection parameters (e.g. last 20 RAW files submitted in January 2025)
- Thermo Fisher .RAW mass spectrometry files downloaded into a RAW staging directory
- MetaXtract YAML configuration file (e.g. config.yml with io.input, outputs, scan_header blocks)

## Outputs

- File-based details TSV per RAW file containing instrument details, scan counts, run statistics, and sample information
- Complete MS1 and MS2 scan-header tables exported per file as TSV/CSV (with PSI-MS/mzML-style column names such as Total Ion Current, Scan Start Time (min), Base Peak Intensity, Ion Injection Time (ms))
- One output subfolder per RAW file under a shared root output directory

## How to apply

Configure the Snakemake workflow to fetch the last N RAW files submitted in a given year (e.g. the last 20 RAW files submitted to PRIDE in January 2025 for accessions PXD060351, PXD060271, and PXD060112) into a RAW staging directory. For each downloaded RAW file, run MetaXtract in CLI or Linux-container mode with file-based details enabled so instrument details, scan counts, run statistics, and sample information are written as a TSV. Enable MS1 and MS2 scan-header extraction with select_all so complete scan-header tables are exported per file as TSV/CSV. Keep ms_method and lc_method exports disabled because Thermo method-reading calls are not supported on Linux containers. Direct outputs into one subfolder per RAW file under a shared root output directory, then validate that each submission produced non-empty file-based-details and MS1/MS2 scan-header tables.

## Related tools

- **MetaXtract** (Performs the end-to-end metadata annotation of downloaded RAW files, extracting instrument-, run-, and scan-level metadata into structured TSV/CSV via its CLI or Linux-container mode) — https://github.com/Rappsilber-Laboratory/MetaXtract
- **Snakemake** (Orchestrates the automated PRIDE download (by year and file count) and per-file MetaXtract annotation steps)

## Examples

```
docker run --rm -v /path/to/raw_files:/data:ro -v /path/to/output:/out metaxtract:latest --input /data/sample.raw --output-dir /out --file-based-details --complete-ms1 --complete-ms2
```

## Evaluation signals

- For each PRIDE accession (PXD060351, PXD060271, PXD060112), the expected structured TSV/CSV metadata files — the file-based details table plus MS1/MS2 scan-header tables — exist and are non-empty
- Outputs are organized as one subfolder per RAW file under the shared root output directory
- The RAW staging directory contains the configured number of files (e.g. 20 per accession for January 2025 submissions)
- MS-method and LC-method export files are absent, confirming ms_method/lc_method were disabled in the Linux container run
- Scan-header table columns follow PSI-MS/mzML-style naming (e.g. Scan Start Time (min), Total Ion Current, Ion Injection Time (ms)) as documented in the README

## Limitations

- Currently supports only Thermo Fisher RAW files; additional vendor file formats are not yet supported
- MS-method and LC-method exports are not supported in the Linux container mode used for the automated pipeline, since Thermo method-reading calls are unsupported on Linux
- On Apple Silicon Macs, running the container with --platform linux/amd64 can crash Mono or pythonnet (e.g. 'Assertion: should not be reached at tramp-amd64.c'); rebuild and run natively or use an x86_64 node

## Evidence
