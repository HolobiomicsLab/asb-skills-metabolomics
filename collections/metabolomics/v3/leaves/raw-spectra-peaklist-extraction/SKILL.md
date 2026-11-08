---
name: raw-spectra-peaklist-extraction
description: Use when native Thermo RAW files must be converted into per-scan indexed
  MS1/MS2 peak-list Parquet files (m/z, intensity, and extended MS2 arrays) that can
  be joined to scan-header metadata by scan number.
when_to_use_negative:
- Input RAW files are from vendors other than Thermo Fisher — MetaXtract currently
  supports only Thermo RAW files.
- The goal is peptide identification or database searching rather than raw peak-array
  extraction — the exported lists carry no identifications.
- On Linux containers, MS-method/LC-method reading is unsupported; only peak-list
  and scan-header exports (not method extraction) are appropriate there.
edam_operation: http://edamontology.org/operation_3215
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_3520
tools:
- name: MetaXtract
  role: Provides ExportPeakList and ExportMS1PeakList to write indexed MS2/MS1 peak
    lists as Parquet, via GUI, CLI, Docker, or Python API
  repo: https://github.com/Rappsilber-Laboratory/MetaXtract
- name: Thermo Fisher RawFileReader
  role: C# library used through pythonnet/Mono to parse scans from the native Thermo
    RAW file
- name: pyarrow
  role: Columnar Parquet engine backing the peak-list export and memory-efficient
    reload
- name: pandas
  role: Used to load and join the exported Parquet peak lists with scan-header tables
    by scan number
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1101/2025.11.12.687968
    title: 'MetaXtract: Extracting Metadata from Raw Files for FAIR Data Practices
      and Workflow Optimisation'
  source_content_hash: sha256:4460d58429f19227946c40042dd277f2136315fe617f349a467f729c9636f93a
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
  - Thermo Fisher RawFileReader
  - numpy
  - pandas
  - pyarrow
  edam_operation: http://edamontology.org/operation_3215
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5384615384615384
n_claims: 13
---

# raw-spectra-peaklist-extraction

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Extracts MS1 and MS2 spectral peak lists from Thermo Fisher RAW files into indexed Parquet files, keying each row by scan number so peak arrays join to scan-header metadata. This enables memory-efficient, columnar access to raw spectra for scalable downstream and data-driven analyses.

## When to use

Use when native Thermo RAW files must be converted into per-scan indexed MS1/MS2 peak-list Parquet files (m/z, intensity, and extended MS2 arrays) that can be joined to scan-header metadata by scan number.

## When NOT to use

- Input RAW files are from vendors other than Thermo Fisher — MetaXtract currently supports only Thermo RAW files.
- The goal is peptide identification or database searching rather than raw peak-array extraction — the exported lists carry no identifications.
- On Linux containers, MS-method/LC-method reading is unsupported; only peak-list and scan-header exports (not method extraction) are appropriate there.

## Inputs

- Thermo Fisher RAW mass spectrometry file (e.g. Pierce HeLa Protein Digest Standard QC runs)
- MetaXtract YAML configuration file with outputs.ms1_peaklist_export and outputs.ms2_peaklist_export enabled

## Outputs

- MS2 extended peak-list Parquet file (one row per MS2 scan: mz_array, intensity_array, resolution_array, noises_array, baselines_array, charges_array)
- MS1 peak-list Parquet file (one row per MS1 scan: mz_array, intensity_array, centroid/profile flag)
- Scan-number-indexed rows joinable to scan-header metadata tables (TSV/CSV)

## How to apply

Open the Thermo RAW file with MetaXtract, which parses scans through the Thermo Fisher RawFileReader library via pythonnet. Call ExportPeakList to write the MS2 extended peak list as a Parquet file with one row per MS2 scan storing mz_array, intensity_array, resolution_array, noises_array, baselines_array, and charges_array; call ExportMS1PeakList to write the MS1 peak list with mz_array, intensity_array, and the centroid/profile flag. In CLI/container runs, enable these exports with the --ms2-peaklist-export and --ms1-peaklist-export flags or outputs.ms2_peaklist_export/ms1_peaklist_export: true in the YAML config. Key each row by its scan number so the arrays join to scan-header tables (Scan Start Time (min), Total Ion Current, Base Peak m/z, Ion Injection Time (ms)) through the scan-number index. The columnar Parquet layout allows loading individual scans without reading whole files, supporting scalable QC and modelling.

## Related tools

- **MetaXtract** (Provides ExportPeakList and ExportMS1PeakList to write indexed MS2/MS1 peak lists as Parquet, via GUI, CLI, Docker, or Python API) — https://github.com/Rappsilber-Laboratory/MetaXtract
- **Thermo Fisher RawFileReader** (C# library used through pythonnet/Mono to parse scans from the native Thermo RAW file)
- **pyarrow** (Columnar Parquet engine backing the peak-list export and memory-efficient reload)
- **pandas** (Used to load and join the exported Parquet peak lists with scan-header tables by scan number)

## Examples

```
docker run --rm -v "$(pwd)/data:/data:ro" -v "$(pwd)/output:/out" metaxtract:latest --input /data/small.RAW --output-dir /out --ms1-peaklist-export --ms2-peaklist-export
```

## Evaluation signals

- Both ms2_peaklist.parquet and ms1_peaklist.parquet exist and load with pandas/pyarrow without error.
- Each scan row yields NumPy arrays whose lengths match across the declared columns (e.g. mz_array and intensity_array have equal length in every row).
- MS2 rows carry all six extended arrays (m/z, intensity, resolution, noise, baseline, charge) while MS1 rows carry m/z, intensity, and the centroid/profile flag.
- Rows are keyed by scan number and join cleanly to the scan-header CSV (Scan Start Time (min), Total Ion Current, Base Peak m/z) on that index with no missing keys.
- Data is readable column-by-column without loading entire files, confirming the columnar Parquet layout.

## Limitations

- MetaXtract currently supports only Thermo Fisher RAW files; additional vendor formats are not yet supported.
- No version number stated for MetaXtract and no changelog available.
- On Linux containers, MS-method and LC-method extraction calls are unsupported; use CLI flags or YAML config instead of the GUI.
- If Mono/pythonnet crashes on Apple Silicon (e.g. 'Assertion: should not be reached at tramp-amd64.c'), rebuild and run without --platform.

## Evidence
