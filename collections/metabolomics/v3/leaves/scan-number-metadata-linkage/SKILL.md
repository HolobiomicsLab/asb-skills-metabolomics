---
name: scan-number-metadata-linkage
description: Use when Thermo Fisher RAW files must yield per-scan peak arrays (m/z,
  intensity, resolution, noise, charge) that join to scan-header metadata by scan
  number for downstream or ML analysis.
when_to_use_negative:
- The input is not a Thermo Fisher RAW file — MetaXtract currently supports only Thermo
  RAW files; additional vendor formats are not yet supported
- Only scan-header metadata is needed (QC metrics, retention time, TIC) without spectral
  peak arrays — use the TSV/CSV metadata exports instead of peak-list export
- Running on a Linux container where MS-method/LC-method exports are required — those
  Thermo method-reading calls are not supported on Linux (peak-list export itself
  still works)
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: MetaXtract
  role: 'Performs the export: ExportPeakList writes the MS2 extended peak list and
    ExportMS1PeakList writes the MS1 peak list as Parquet, keyed by scan number'
  repo: https://github.com/Rappsilber-Laboratory/MetaXtract
- name: Thermo Fisher RawFileReader
  role: C# library used via pythonnet to parse scans from the RAW file before peak-list
    export
- name: pyarrow
  role: Columnar Parquet engine used for writing and later loading the peak-list files
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1101/2025.11.12.687968
    title: 'MetaXtract: Extracting Metadata from Raw Files for FAIR Data Practices
      and Workflow Optimisation'
  source_content_hash: sha256:fa51afd10e777c549f0a7ba7f4a1a8ca9b8896702df70f71fb760335837c32e1
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
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.46153846153846156
n_claims: 13
---

# scan-number-metadata-linkage

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Export MS2 spectra as indexed extended peak lists (m/z, intensity, resolution, noise, baseline, charge) and MS1 spectra as m/z/intensity arrays in columnar Parquet, keying each row by scan number so peak arrays join to scan-header metadata (Scan Start Time, Total Ion Current, Base Peak m/z) via the scan-number index.

## When to use

Use when Thermo Fisher RAW files must yield per-scan peak arrays (m/z, intensity, resolution, noise, charge) that join to scan-header metadata by scan number for downstream or ML analysis.

## When NOT to use

- The input is not a Thermo Fisher RAW file — MetaXtract currently supports only Thermo RAW files; additional vendor formats are not yet supported
- Only scan-header metadata is needed (QC metrics, retention time, TIC) without spectral peak arrays — use the TSV/CSV metadata exports instead of peak-list export
- Running on a Linux container where MS-method/LC-method exports are required — those Thermo method-reading calls are not supported on Linux (peak-list export itself still works)

## Inputs

- Native Thermo Fisher RAW mass spectrometry file (.raw)
- MetaXtract config.yml with outputs.ms2_peaklist_export and outputs.ms1_peaklist_export set to true (or equivalent CLI flags --ms1-peaklist-export / --ms2-peaklist-export)

## Outputs

- ms2_peaklist.parquet: one row per MS2 scan with mz_array, intensity_array, resolution_array, noises_array, baselines_array, charges_array, keyed by scan number
- ms1_peaklist.parquet: one row per MS1 scan with mz_array, intensity_array, and the centroid/profile flag, keyed by scan number

## How to apply

Open the Thermo RAW file with MetaXtract, which parses scans through the Thermo Fisher RawFileReader C# library via pythonnet. Call ExportPeakList to write the MS2 extended peak list as a Parquet file with one row per MS2 scan storing mz_array, intensity_array, resolution_array, noises_array, baselines_array, and charges_array. Call ExportMS1PeakList to write the MS1 peak list as a Parquet file with one row per MS1 scan storing mz_array, intensity_array, and the centroid/profile flag. Key each row by its scan number so the peak arrays join to the scan-header tables (e.g. Scan Start Time (min), Total Ion Current, Base Peak m/z) through the scan-number index; in CLI/config runs set ms2_peaklist_export: true and ms1_peaklist_export: true. Choose Parquet because the columnar format enables memory-efficient access without loading entire files into memory, supporting scalable downstream processing and data-driven modelling such as spectral quality assessment or retention time prediction.

## Related tools

- **MetaXtract** (Performs the export: ExportPeakList writes the MS2 extended peak list and ExportMS1PeakList writes the MS1 peak list as Parquet, keyed by scan number) — https://github.com/Rappsilber-Laboratory/MetaXtract
- **Thermo Fisher RawFileReader** (C# library used via pythonnet to parse scans from the RAW file before peak-list export)
- **pyarrow** (Columnar Parquet engine used for writing and later loading the peak-list files)

## Examples

```
docker run --rm -v $(pwd)/data:/data:ro -v $(pwd)/out:/out metaxtract:latest --input /data/small.RAW --output-dir /out --complete-ms1 --complete-ms2 --ms1-peaklist-export --ms2-peaklist-export
```

## Evaluation signals

- Both ms2_peaklist.parquet and ms1_peaklist.parquet exist in the per-RAW output subfolder and are loadable with pandas/pyarrow
- Each scan row yields NumPy arrays of matching lengths for all declared columns (mz_array, intensity_array, and resolution/noise/baseline/charges for MS2)
- Every peak-list row is keyed by a scan number that resolves to a row in the exported scan-header tables (e.g. Total Ion Current, Base Peak m/z) — the join on scan number produces no unmatched rows
- MS2 rows carry the extended peak-list fields (resolution, noise, baseline, charge state) while MS1 rows carry only m/z, intensity, and the centroid/profile flag
- Number of rows in each Parquet file equals the count of MS2 or MS1 scans respectively as reported in the scan-header exports

## Limitations

- Only Thermo Fisher RAW files are supported; additional vendor file formats are not yet supported
- No version number is stated for MetaXtract and no changelog is provided
- MS1/MS2 scan arrays are written per scan with all possible information; the MS2 extended peak list is fixed to the declared columns (m/z, intensity, resolution, noise, baseline, charge)

## Evidence
