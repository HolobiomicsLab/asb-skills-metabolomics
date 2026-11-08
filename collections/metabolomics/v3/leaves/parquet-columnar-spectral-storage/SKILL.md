---
name: parquet-columnar-spectral-storage
description: Use when Thermo RAW spectral peak data must be converted to scalable,
  analysis-ready arrays for downstream ML/QC workflows, joining per-scan spectra to
  scan-header metadata via scan numbers.
when_to_use_negative:
- Input is not a Thermo Fisher RAW file — MetaXtract currently supports only Thermo
  RAW files and additional vendor formats are not yet supported
- Only scan-header metadata is needed (retention time, TIC, injection time) — the
  plain metadata/TSV export path suffices without peak-list Parquet export
- Running on Linux containers for MS-method/LC-method extraction — those Thermo method-reading
  calls are unsupported on Linux (peak-list export itself still works in containers)
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: MetaXtract
  role: Provides ExportPeakList and ExportMS1PeakList methods that write the MS2 extended
    and MS1 Parquet peak lists keyed by scan number
  repo: https://github.com/Rappsilber-Laboratory/MetaXtract
- name: Thermo Fisher RawFileReader
  role: Underlying C# library used through pythonnet to parse scans and centroided
    peak arrays from the native RAW file
- name: pyarrow
  role: Columnar Parquet reader/writer dependency enabling memory-efficient loading
    of the exported spectral arrays
- name: pandas
  role: Loads the Parquet peak-list tables and joins them to scan-header metadata
    via the scan-number index
- name: numpy
  role: Represents the per-scan mz/intensity/resolution/noise/baseline/charge arrays
    stored in each Parquet row
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1101/2025.11.12.687968
    title: 'MetaXtract: Extracting Metadata from Raw Files for FAIR Data Practices
      and Workflow Optimisation'
  source_content_hash: sha256:26d1b605d9c7f95f3dd82cb160f6b8e2df95467aa9a40638d747bfa549a206f5
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
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# parquet-columnar-spectral-storage

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Exports Thermo RAW-file MS1 and MS2 peak lists as indexed, columnar Parquet files — one row per scan holding NumPy-style arrays (m/z, intensity, and for MS2 also resolution, noise, baseline, and charge) — so spectra join to scan-header metadata by scan number and can be read memory-efficiently without loading whole RAW files.

## When to use

Use when Thermo RAW spectral peak data must be converted to scalable, analysis-ready arrays for downstream ML/QC workflows, joining per-scan spectra to scan-header metadata via scan numbers.

## When NOT to use

- Input is not a Thermo Fisher RAW file — MetaXtract currently supports only Thermo RAW files and additional vendor formats are not yet supported
- Only scan-header metadata is needed (retention time, TIC, injection time) — the plain metadata/TSV export path suffices without peak-list Parquet export
- Running on Linux containers for MS-method/LC-method extraction — those Thermo method-reading calls are unsupported on Linux (peak-list export itself still works in containers)

## Inputs

- Thermo Fisher .RAW mass spectrometry file (parsed via RawFileReader)
- MetaXtract config.yml with outputs.ms1_peaklist_export and outputs.ms2_peaklist_export set to true (or equivalent CLI flags)
- Optional: bundled small.RAW demo file for a validation run

## Outputs

- ms2_peaklist.parquet — one row per MS2 scan with mz_array, intensity_array, resolution_array, noises_array, baselines_array, charges_array, keyed by scan number
- ms1_peaklist.parquet — one row per MS1 scan with mz_array, intensity_array, and centroid/profile flag, keyed by scan number
- Scan-header tables (TSV/CSV with Scan Start Time (min), Total Ion Current, Base Peak m/z, etc.) joinable to the Parquet rows via scan number

## How to apply

Parse the Thermo RAW file via the Thermo Fisher RawFileReader C# library through pythonnet (MetaXtract wraps this). For MS2 spectra call raw.ExportPeakList (e.g. raw.ExportPeakList("ms2_peaklist.parquet")) to write a Parquet file with one row per MS2 scan storing mz_array, intensity_array, resolution_array, noises_array, baselines_array, and charges_array. For MS1 spectra call raw.ExportMS1PeakList (e.g. raw.ExportMS1PeakList("ms1_peaklist.parquet")) to write one row per MS1 scan storing mz_array, intensity_array, and the centroid/profile flag. Key each row by its scan number so peak arrays join to the scan-header tables (Scan Start Time (min), Total Ion Current, Base Peak m/z) through the scan-number index. Enable the exports with the ms1_peaklist_export/ms2_peaklist_export flags in config.yml or the corresponding CLI flags; Parquet is chosen because columnar storage enables memory-efficient access without loading entire files into memory, supporting scalable downstream processing and data-driven modelling such as spectral quality assessment or retention-time prediction.

## Related tools

- **MetaXtract** (Provides ExportPeakList and ExportMS1PeakList methods that write the MS2 extended and MS1 Parquet peak lists keyed by scan number) — https://github.com/Rappsilber-Laboratory/MetaXtract
- **Thermo Fisher RawFileReader** (Underlying C# library used through pythonnet to parse scans and centroided peak arrays from the native RAW file)
- **pyarrow** (Columnar Parquet reader/writer dependency enabling memory-efficient loading of the exported spectral arrays)
- **pandas** (Loads the Parquet peak-list tables and joins them to scan-header metadata via the scan-number index)
- **numpy** (Represents the per-scan mz/intensity/resolution/noise/baseline/charge arrays stored in each Parquet row)

## Examples

```
raw.ExportPeakList("ms2_peaklist.parquet"); raw.ExportMS1PeakList("ms1_peaklist.parquet")
```

## Evaluation signals

- Both ms1_peaklist.parquet and ms2_peaklist.parquet exist in the output directory and are loadable with pandas/pyarrow
- Each scan row yields NumPy arrays of matching lengths for all declared columns (e.g. len(mz_array) == len(intensity_array) == len(charges_array) for MS2 rows)
- The MS2 file contains the extended peak-list columns (mz, intensity, resolution, noise, baseline, charge); the MS1 file contains mz, intensity, and the centroid/profile flag
- Parquet row scan numbers join one-to-one with the scan-header tables (Scan Start Time (min), Total Ion Current, Base Peak m/z) via the scan-number index
- Peak counts per scan are consistent with the scan-header 'Total Number of Peaks' values for the same scans

## Limitations

- Currently supports only Thermo Fisher RAW files; additional vendor file formats are not yet supported
- Peak-list export is enabled per run via config/CLI flags (ms1_peaklist_export / ms2_peaklist_export default off in some configurations) — must be explicitly turned on
- Parquet export is separate from the HDF5 export option (hdf5_export, default false); pick one format appropriate for the downstream workflow
- The export supports data-driven modelling (spectral quality assessment, retention time prediction, fragmentation analysis) but linking QC deviations to instrument-level causes is described as plausible rather than validated

## Evidence
