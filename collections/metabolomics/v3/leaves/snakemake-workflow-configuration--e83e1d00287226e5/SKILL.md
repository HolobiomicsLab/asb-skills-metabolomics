---
name: snakemake-workflow-configuration
description: Use when a task requires automated, reproducible downloading of deposited
  Thermo RAW files from PRIDE accessions and their annotation with instrument-, run-,
  and scan-level metadata via MetaXtract in a batch workflow.
when_to_use_negative:
- Input files are not Thermo Fisher RAW files — MetaXtract currently supports only
  Thermo RAW and additional vendor formats are not yet supported
- The goal is extracting Thermo MS method or LC method files on Linux containers —
  ms_method and lc_method exports are not supported on Linux
- Interactive, single-file inspection with a GUI is needed rather than automated batch
  processing — the container workflow is CLI/config-only and excludes the GUI
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_0091
tools:
- name: MetaXtract
  role: Core annotation engine invoked per RAW file by the Snakemake workflow; reads
    RAW via RawFileReader and exports instrument/run/scan-level metadata to TSV/CSV
    (and Parquet peak lists)
  repo: https://github.com/Rappsilber-Laboratory/MetaXtract
- name: Snakemake
  role: Workflow engine orchestrating PRIDE RAW download and end-to-end MetaXtract
    annotation; the workflow is provided in the MetaXtract GitHub repository
  repo: https://github.com/Rappsilber-Laboratory/MetaXtract
- name: Docker / Apptainer-Singularity
  role: Recommended CLI-only Linux container execution mode for macOS and HPC, running
    MetaXtract with bundled Thermo RawFileReader DLLs via Mono and pythonnet
  repo: https://github.com/Rappsilber-Laboratory/MetaXtract
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1101/2025.11.12.687968
    title: 'MetaXtract: Extracting Metadata from Raw Files for FAIR Data Practices
      and Workflow Optimisation'
  source_content_hash: sha256:3336591d6e4a361017a520f34c34da136b7a72ce7c3ae52cc1d6122c3effae93
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
  - Docker / Apptainer-Singularity
  - MetaXtract
  - Snakemake
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# snakemake-workflow-configuration

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A Snakemake-based, open-source workflow shipped with MetaXtract that automatically downloads RAW files from PRIDE (by year and file count) and runs end-to-end metadata annotation, producing structured TSV/CSV metadata (file-based details plus MS1/MS2 scan-header tables) for FAIR, reproducible batch annotation.

## When to use

Use when a task requires automated, reproducible downloading of deposited Thermo RAW files from PRIDE accessions and their annotation with instrument-, run-, and scan-level metadata via MetaXtract in a batch workflow.

## When NOT to use

- Input files are not Thermo Fisher RAW files — MetaXtract currently supports only Thermo RAW and additional vendor formats are not yet supported
- The goal is extracting Thermo MS method or LC method files on Linux containers — ms_method and lc_method exports are not supported on Linux
- Interactive, single-file inspection with a GUI is needed rather than automated batch processing — the container workflow is CLI/config-only and excludes the GUI

## Inputs

- PRIDE accession identifiers (e.g. PXD060351, PXD060271, PXD060112)
- Year and file-count selectors for PRIDE RAW download (e.g. last 20 RAW files from January 2025)
- Native Thermo Fisher .RAW mass spectrometry files
- YAML configuration file (config.yml / config_container_small.yml) specifying io.input, output_dir, outputs flags, and scan_header column selection

## Outputs

- File-based details TSV per RAW file (instrument details, scan counts, run statistics, sample information)
- MS1 scan-header table (TSV/CSV, complete when select_all is enabled)
- MS2 scan-header table (TSV/CSV, complete when select_all is enabled)
- Optional MS1/MS2 peak lists as Parquet
- One subfolder per RAW file under a shared root output directory

## How to apply

Configure the workflow to fetch a defined set of RAW files (e.g. the last 20 files submitted in January 2025) per PRIDE accession such as PXD060351, PXD060271, and PXD060112 into a RAW staging directory. For each RAW file, run MetaXtract with file-based details enabled so instrument details, scan counts, run statistics, and sample information are written as TSV, and enable MS1 and MS2 scan-header extraction with select_all so complete scan-header tables are exported as TSV/CSV (optionally also peak lists as Parquet). Keep ms_method and lc_method exports disabled because Thermo method-reading calls are unsupported in the Linux container mode (Docker/Apptainer via Mono and pythonnet) used for automated runs. Direct outputs into one subfolder per RAW file under a shared root output directory, driven either by a YAML config (io.input, outputs, scan_header sections) or CLI flags. Validate that each submission yields the expected non-empty structured TSV/CSV files matching the metadata-annotation step.

## Related tools

- **MetaXtract** (Core annotation engine invoked per RAW file by the Snakemake workflow; reads RAW via RawFileReader and exports instrument/run/scan-level metadata to TSV/CSV (and Parquet peak lists)) — https://github.com/Rappsilber-Laboratory/MetaXtract
- **Snakemake** (Workflow engine orchestrating PRIDE RAW download and end-to-end MetaXtract annotation; the workflow is provided in the MetaXtract GitHub repository) — https://github.com/Rappsilber-Laboratory/MetaXtract
- **Docker / Apptainer-Singularity** (Recommended CLI-only Linux container execution mode for macOS and HPC, running MetaXtract with bundled Thermo RawFileReader DLLs via Mono and pythonnet) — https://github.com/Rappsilber-Laboratory/MetaXtract

## Examples

```
docker run --rm -v "$(pwd)/data:/data:ro" -v "$(pwd)/output:/out" -v "$(pwd)/config_container_small.yml:/config.yml:ro" metaxtract:latest --config /config.yml
```

## Evaluation signals

- For each PRIDE accession (e.g. PXD060351), confirm the expected file-based-details TSV exists and is non-empty under the per-RAW output subfolder
- Confirm complete MS1 and MS2 scan-header TSV/CSV tables exist and are non-empty for every downloaded RAW file
- Check that ms_method (*_MS_method.txt) and lc_method (*_LC_method.txt) files are absent when running the Linux container config (both flags false)
- Verify scan-header column names follow PSI-MS/mzML-style naming (e.g. 'Scan Start Time (min)', 'Total Ion Current', 'Base Peak m/z') with thermo_ prefix only for unmatched Thermo trailer fields
- Verify the downloaded RAW set matches the configured year/file-count selection (e.g. 20 files per accession from January 2025)

## Limitations

- MS method and LC method export must be disabled on Linux containers because Thermo method-reading calls are unsupported there
- MetaXtract currently supports only Thermo Fisher RAW files; additional vendor formats are planned for future versions
- On Apple Silicon, running with --platform linux/amd64 causes Mono/pythonnet crashes (e.g. 'Assertion: should not be reached at tramp-amd64.c'); run native ARM64 or use an x86_64 host
- No version number stated for MetaXtract and no changelog provided, complicating pinning of the workflow environment

## Evidence
