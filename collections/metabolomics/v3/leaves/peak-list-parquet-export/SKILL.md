---
name: peak-list-parquet-export
description: Use when Thermo .RAW files must be converted into columnar (Parquet)
  spectral peak lists for downstream machine learning, secondary analysis, or QC —
  e.g. when per-scan m/z, intensity, noise, baseline and charge arrays are needed.
when_to_use_negative:
- Input files are not Thermo Fisher RAW format — MetaXtract currently supports only
  Thermo RAW; additional vendor formats are not yet supported
- The goal is database-search-dependent QC metrics — this skill is search-free and
  acquisition-side only, so it cannot substitute for tools like MSQC that compute
  metrics from identified spectra
- ms_method/lc_method extraction on Linux containers is needed — those Thermo method-reading
  calls are not supported on Linux (peak-list export itself is unaffected)
edam_topics:
- http://edamontology.org/topic_0121
tools:
- name: MetaXtract
  role: 'Primary tool: parses Thermo RAW via RawFileReader through pythonnet and writes
    ms1_peaklist.parquet and ms2_peaklist.parquet via ExportMS1PeakList and ExportPeakList'
  repo: https://github.com/Rappsilber-Laboratory/MetaXtract
- name: RawFileReader
  role: Thermo Fisher C# library used by MetaXtract to access native RAW file scan
    arrays and trailer information
- name: PyArrow
  role: Provides the Parquet columnar serialization backend for the exported peak
    lists (installed among MetaXtract CLI dependencies)
  repo: https://github.com/Rappsilber-Laboratory/MetaXtract
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1101/2025.11.12.687968
    title: 'MetaXtract: Extracting Metadata from Raw Files for FAIR Data Practices
      and Workflow Optimisation'
  source_content_hash: sha256:3b3a1388c00e6485a085d629896293dbfb8e0cf205e22e8bce043d8cebebb0c3
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
  - Plotly
  - PyArrow
  - RawFileReader
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6923076923076923
n_claims: 13
---

# peak-list-parquet-export

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Exports MS1 and MS2 spectral peak lists from Thermo RAW files as Parquet files: MS2 as indexed extended peak lists (mz_array, intensity_array, resolution_array, noises_array, baselines_array, charges_array) linked to scan-level metadata via scan numbers, and MS1 as m/z/intensity arrays with a centroid/profile flag. This makes raw spectra available for data-driven modelling such as spectral quality assessment, retention-time prediction, or anomaly detection.

## When to use

Use when Thermo .RAW files must be converted into columnar (Parquet) spectral peak lists for downstream machine learning, secondary analysis, or QC — e.g. when per-scan m/z, intensity, noise, baseline and charge arrays are needed.

## When NOT to use

- Input files are not Thermo Fisher RAW format — MetaXtract currently supports only Thermo RAW; additional vendor formats are not yet supported
- The goal is database-search-dependent QC metrics — this skill is search-free and acquisition-side only, so it cannot substitute for tools like MSQC that compute metrics from identified spectra
- ms_method/lc_method extraction on Linux containers is needed — those Thermo method-reading calls are not supported on Linux (peak-list export itself is unaffected)

## Inputs

- Thermo Fisher .RAW mass spectrometry files (e.g. DDA HeLa QC RAW files acquired with top-10 method, 2+ to 6+ charge states, HCD 27% NCE)
- YAML config (config.yml) with outputs.ms1_peaklist_export and outputs.ms2_peaklist_export set to true, or CLI flags --ms1-peaklist-export / --ms2-peaklist-export

## Outputs

- ms2_peaklist.parquet per RAW file: indexed extended peak list with mz_array, intensity_array, resolution_array, noises_array, baselines_array, charges_array per MS2 scan
- ms1_peaklist.parquet per RAW file: MS1 mz_array and intensity_array with centroid/profile flag
- Scan-header CSV/TSV tables (PSI-MS-style columns) joinable to the Parquet peak lists via scan numbers
- runtime_summary_YYYYMMDD_HHMMSS.tsv with one row per processed RAW file (status, runtime, start/end/peak memory)

## How to apply

Configure the run with ms2_peaklist_export and ms1_peaklist_export set to true in the YAML 'outputs' block (or pass --ms2-peaklist-export and --ms1-peaklist-export on the CLI). MetaXtract parses each RAW file through the Thermo RawFileReader C# library via pythonnet and calls ExportPeakList("ms2_peaklist.parquet") to write the extended MS2 peak list (mz_array, intensity_array, resolution_array, noises_array, baselines_array, charges_array per scan) and ExportMS1PeakList("ms1_peaklist.parquet") to write MS1 arrays with a centroid/profile flag. Each peak list is indexed by scan number so it joins to the corresponding MS1/MS2 scan-header CSV/TSV tables (e.g. Scan Start Time (min), Total Ion Current, Ion Injection Time (ms)). Run one RAW file at a time or list several under io.input; a per-RAW output subfolder is written under output_dir. Note that MS-method/LC-method exports are separate flags and are unsupported on Linux containers, but peak-list export is container-safe.

## Related tools

- **MetaXtract** (Primary tool: parses Thermo RAW via RawFileReader through pythonnet and writes ms1_peaklist.parquet and ms2_peaklist.parquet via ExportMS1PeakList and ExportPeakList) — https://github.com/Rappsilber-Laboratory/MetaXtract
- **RawFileReader** (Thermo Fisher C# library used by MetaXtract to access native RAW file scan arrays and trailer information)
- **PyArrow** (Provides the Parquet columnar serialization backend for the exported peak lists (installed among MetaXtract CLI dependencies)) — https://github.com/Rappsilber-Laboratory/MetaXtract

## Examples

```
docker run --rm -v "$(pwd)/data:/data:ro" -v "$(pwd)/output:/out" metaxtract:latest --input /data/small.RAW --output-dir /out --ms1-peaklist-export --ms2-peaklist-export
```

## Evaluation signals

- Each processed RAW file yields both ms1_peaklist.parquet and ms2_peaklist.parquet in its per-file output subfolder
- MS2 Parquet rows carry non-empty mz_array, intensity_array, resolution_array, noises_array, baselines_array and charges_array keyed by scan number that matches scan-header rows
- MS1 peak lists include a centroid/profile flag so profile-mode and centroided scans are distinguishable
- runtime_summary_YYYYMMDD_HHMMSS.tsv contains one row per RAW file with completion status and memory/runtime statistics
- Scan counts in the peak lists are consistent with file-level scan statistics in the file_based_details TSV (MS1 survey scans plus MS2 for top-10 DDA, charge states 2+ to 6+)

## Limitations

- Only Thermo Fisher RAW files are supported; additional vendor formats are not yet supported
- No version number is stated for MetaXtract and no changelog is provided, so reproducibility of export behaviour across releases is unverified
- On Apple Silicon, running the container with --platform linux/amd64 can crash Mono/pythonnet ('Assertion: should not be reached at tramp-amd64.c'); build/run natively or use an x86_64 host

## Evidence
