---
name: pride-dataset-batch-processing
description: Use when a benchmark or QC task requires annotating a batch of publicly
  deposited Thermo RAW files from PRIDE (e.g. the most recent N files from a given
  year) via an automated, reproducible pipeline.
when_to_use_negative:
- The RAW files are not Thermo Fisher format — MetaXtract currently supports only
  Thermo RAW files and additional vendor formats are not yet supported
- The goal is interactive inspection of a single file rather than automated batch
  processing — the GUI mode is more appropriate
- The task requires MS-method or LC-method extraction on a Linux/HPC container — those
  Thermo method-reading calls are not supported on Linux
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_3360
tools:
- name: MetaXtract
  role: Extracts metadata (instrument details, MS1/MS2 scan headers, technical trailer
    details, peak lists) from each downloaded Thermo RAW file into structured TSV/CSV/Parquet
    outputs
  repo: https://github.com/Rappsilber-Laboratory/MetaXtract
- name: pandas
  role: Parses the runtime_summary_*.tsv files, extracts per-file runtimes, and computes
    summary statistics for the runtime comparison table
- name: Snakemake
  role: Orchestrates the automated download of RAW files from PRIDE by year and file
    count and the end-to-end MetaXtract annotation run
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1101/2025.11.12.687968
    title: 'MetaXtract: Extracting Metadata from Raw Files for FAIR Data Practices
      and Workflow Optimisation'
  source_content_hash: sha256:5454aa21392beaa9894c1b5d1f2b3f42b9efa92cf2d6e12ccc230a550f3e2f2e
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
  - pandas
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_3360
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# pride-dataset-batch-processing

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Batch-download RAW files from the PRIDE repository by year and file count using the MetaXtract Snakemake workflow, then run end-to-end metadata annotation into structured TSV/CSV outputs while tracking per-file wall-clock runtime.

## When to use

Use when a benchmark or QC task requires annotating a batch of publicly deposited Thermo RAW files from PRIDE (e.g. the most recent N files from a given year) via an automated, reproducible pipeline.

## When NOT to use

- The RAW files are not Thermo Fisher format — MetaXtract currently supports only Thermo RAW files and additional vendor formats are not yet supported
- The goal is interactive inspection of a single file rather than automated batch processing — the GUI mode is more appropriate
- The task requires MS-method or LC-method extraction on a Linux/HPC container — those Thermo method-reading calls are not supported on Linux

## Inputs

- PRIDE repository query parameters (year and file count) for RAW-file selection, e.g. last 20 files submitted in January 2025
- Thermo Fisher .RAW files downloaded from PRIDE
- Snakemake workflow configuration from the MetaXtract GitHub repository
- runtime_summary_*.tsv files written by MetaXtract for each processed RAW file

## Outputs

- Per-RAW-file metadata TSV/CSV exports (instrument details, scan counts, run statistics, sample information)
- runtime_comparison.tsv with per-file runtime, deviation from the ~40 s reference, and per-file status
- Summary statistics (mean, minimum, maximum) of per-file wall-clock runtimes

## How to apply

Use the Snakemake workflow shipped in the MetaXtract GitHub repository, which automatically downloads RAW files from PRIDE (selectable by year and file count, e.g. the last 20 RAW files submitted in January 2025) and runs MetaXtract end-to-end metadata annotation. MetaXtract reads each Thermo RAW file through the RawFileReader C# library and writes structured outputs: file/instrument metadata TSV, scan headers, and optionally MS1/MS2 peak lists in Parquet. On the reference Windows 11 laptop (Intel Core i9-13900HX, 32 GB RAM, NVIDIA RTX 4070), expect roughly 40 s wall-clock runtime per RAW file, so budget batch time accordingly. After the batch run, parse the per-file runtime_summary_*.tsv files with pandas, compute mean/min/max runtime statistics, and compare each file against the ~40 s reference average. Tabulate per-file runtime, deviation from reference, and status into a summary table (e.g. runtime_comparison.tsv) to judge whether batch performance matches the reported benchmark.

## Related tools

- **MetaXtract** (Extracts metadata (instrument details, MS1/MS2 scan headers, technical trailer details, peak lists) from each downloaded Thermo RAW file into structured TSV/CSV/Parquet outputs) — https://github.com/Rappsilber-Laboratory/MetaXtract
- **pandas** (Parses the runtime_summary_*.tsv files, extracts per-file runtimes, and computes summary statistics for the runtime comparison table)
- **Snakemake** (Orchestrates the automated download of RAW files from PRIDE by year and file count and the end-to-end MetaXtract annotation run)

## Examples

```
import pandas as pd; df = pd.read_csv('runtime_comparison.tsv', sep='\t'); assert df['runtime_s'].gt(0).all(); print(df['runtime_s'].describe())
```

## Evaluation signals

- runtime_comparison.tsv exists, is well-formed TSV with one row per processed RAW file (20 rows for the January 2025 batch)
- Each row's runtime value is a positive number
- Mean per-file runtime is close to the reported ~40 s reference on the reference Windows 11 hardware (Intel Core i9-13900HX, 32 GB RAM, RTX 4070)
- One output subfolder per RAW file is produced under the output directory, each containing the expected file-based-details TSV exports
- Per-file deviation from the 40 s reference is finite and reported for every file with a status label

## Limitations

- MetaXtract currently supports only Thermo Fisher RAW files; additional vendor file formats are not yet supported
- No version number is stated for MetaXtract and no changelog is provided
- MS-method and LC-method export options are not supported on Linux containers
- Runtime depends on hardware; the ~40 s per-file figure is specific to the reference Windows 11 laptop (Intel Core i9-13900HX, 32 GB RAM, NVIDIA RTX 4070)

## Evidence

- [discussion] Only Thermo RAW files supported: [evidence span withheld — anchor status: not_found]
