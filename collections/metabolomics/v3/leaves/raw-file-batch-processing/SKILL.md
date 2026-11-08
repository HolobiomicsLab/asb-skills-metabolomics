---
name: raw-file-batch-processing
description: Use when RAW files from public repositories (e.g., PRIDE accessions such
  as PXD060351) must be bulk-downloaded and annotated with acquisition metadata in
  structured TSV/CSV outputs without a GUI.
when_to_use_negative:
- Input files are non-Thermo vendor formats — MetaXtract currently supports only Thermo
  Fisher RAW files
- The task requires extracting MS or LC method files (*_MS_method.txt / *_LC_method.txt)
  on a Linux container, where these Thermo method-reading calls are not supported
- An interactive, GUI-driven single-file inspection is needed rather than automated
  batch annotation
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_3520
tools:
- name: MetaXtract
  role: Performs the end-to-end metadata annotation of each downloaded RAW file in
    CLI/container mode, exporting file-based details and MS1/MS2 scan-header tables
    as TSV/CSV
  repo: https://github.com/Rappsilber-Laboratory/MetaXtract
- name: Snakemake
  role: 'Orchestrates the batch workflow: RAW download from PRIDE by year and file
    count, per-file MetaXtract execution, and collection of structured outputs'
  repo: https://github.com/Rappsilber-Laboratory/MetaXtract
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1101/2025.11.12.687968
    title: 'MetaXtract: Extracting Metadata from Raw Files for FAIR Data Practices
      and Workflow Optimisation'
  source_content_hash: sha256:3bfd9e78d30156d5be4a8f9a1a98e0cf98f86034f99ee5eeaa5f0e568a6e37af
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
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# raw-file-batch-processing

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A reproducible Snakemake-based batch workflow that downloads RAW files from PRIDE by accession, year, and file count, then runs MetaXtract end-to-end to annotate each file with instrument-, run-, and scan-level metadata exported as structured TSV/CSV.

## When to use

Use when RAW files from public repositories (e.g., PRIDE accessions such as PXD060351) must be bulk-downloaded and annotated with acquisition metadata in structured TSV/CSV outputs without a GUI.

## When NOT to use

- Input files are non-Thermo vendor formats — MetaXtract currently supports only Thermo Fisher RAW files
- The task requires extracting MS or LC method files (*_MS_method.txt / *_LC_method.txt) on a Linux container, where these Thermo method-reading calls are not supported
- An interactive, GUI-driven single-file inspection is needed rather than automated batch annotation

## Inputs

- PRIDE accession identifiers (e.g., PXD060351, PXD060271, PXD060112)
- Year and file-count selection parameters (e.g., last 20 RAW files submitted in January 2025 per accession)
- Native Thermo Fisher .RAW mass spectrometry files
- MetaXtract YAML configuration file (config.yml / config_container_small.yml)

## Outputs

- Per-RAW-file file-based details TSV (instrument details, scan counts, run statistics, sample information)
- Per-RAW-file MS1 scan-header table (TSV/CSV) with select_all columns such as Total Ion Current, Ion Injection Time (ms), Scan Start Time (min), Base Peak Intensity
- Per-RAW-file MS2 scan-header table (TSV/CSV), e.g., Filter String, Scan Mode, Total Ion Current
- One output subfolder per RAW file under a shared root output directory

## How to apply

Configure the Snakemake workflow to fetch the last N RAW files (e.g., 20 per accession, for the January-2025 batch: PXD060351, PXD060271, PXD060112) into a RAW staging directory. For each RAW file, run MetaXtract in CLI/container mode with --file-based-details to produce instrument details, scan counts, run statistics, and sample information as TSV, and enable --complete-ms1/--complete-ms2 (select_all) so full scan-header tables are exported per file. Keep ms_method and lc_method disabled because Thermo method-reading calls are unsupported in the Linux container mode. Direct MetaXtract outputs into one subfolder per RAW file under a shared root output directory and validate that each submission's file-based-details TSV plus MS1/MS2 scan-header TSV/CSV files exist and are non-empty.

## Related tools

- **MetaXtract** (Performs the end-to-end metadata annotation of each downloaded RAW file in CLI/container mode, exporting file-based details and MS1/MS2 scan-header tables as TSV/CSV) — https://github.com/Rappsilber-Laboratory/MetaXtract
- **Snakemake** (Orchestrates the batch workflow: RAW download from PRIDE by year and file count, per-file MetaXtract execution, and collection of structured outputs) — https://github.com/Rappsilber-Laboratory/MetaXtract

## Examples

```
docker run --rm -v "$(pwd)/data:/data:ro" -v "$(pwd)/output:/out" metaxtract:latest --input /data/sample.raw --output-dir /out --file-based-details --complete-ms1 --complete-ms2
```

## Evaluation signals

- For each PRIDE accession, the expected structured TSV/CSV metadata files (file-based details table plus MS1/MS2 scan-header tables) exist and are non-empty
- Each RAW file has its own output subfolder under the shared root output directory
- ms_method and lc_method outputs are absent/disabled in Linux container runs, consistent with README guidance
- MS1/MS2 scan-header tables contain the select_all columns with PSI-MS/mzML-style names (e.g., Total Ion Current, Ion Injection Time (ms), Scan Start Time (min))
- Downloaded RAW file counts match the configured per-accession limit (e.g., 20 files submitted in January 2025)

## Limitations

- ms_method and lc_method exports must be disabled on Linux containers because those Thermo method-reading calls are not supported on Linux
- Currently supports only Thermo Fisher RAW files; additional vendor file formats are not yet supported
- On Apple Silicon, running the container with --platform linux/amd64 can crash Mono/pythonnet ('Assertion: should not be reached at tramp-amd64.c'); use the native ARM64 Linux backend instead
- The container workflow excludes the GUI; only CLI options or YAML configuration are available

## Evidence

- [intro] MetaXtract is a lightweight Python-based tool that directly reads RAW files using the Thermo Fisher RAWFileReader library: [evidence span withheld — anchor status: not_found]
