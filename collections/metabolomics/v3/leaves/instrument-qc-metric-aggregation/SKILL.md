---
name: instrument-qc-metric-aggregation
description: Use when QCing Thermo RAW LC-MS/MS runs (e.g. HeLa digest standards)
  and you need to compare scan-level acquisition metrics across files to flag underperforming
  batches before searching.
when_to_use_negative:
- The input files are not Thermo Fisher RAW — MetaXtract currently supports only Thermo
  RAW and additional vendor formats are planned but not yet implemented
- You need QC metrics that require database search results (e.g. identified-peptide-based
  metrics) — MetaXtract deliberately uses acquisition-side signals only
- You want to export MS-method or LC-method text on a Linux container — those Thermo
  method-reading calls are not supported on Linux and ms_method/lc_method must be
  disabled
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_2269
tools:
- name: MetaXtract
  role: 'Primary tool: extracts scan-level metadata and peak lists from Thermo RAW
    files into structured TSV/CSV and Parquet tables for cross-run QC comparison'
  repo: https://github.com/Rappsilber-Laboratory/MetaXtract
- name: RawFileReader
  role: Thermo C# library used underneath MetaXtract to parse the native RAW files
- name: Plotly
  role: Generates interactive HTML reports for visualising and comparing scan-level
    metrics across files
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1101/2025.11.12.687968
    title: 'MetaXtract: Extracting Metadata from Raw Files for FAIR Data Practices
      and Workflow Optimisation'
  source_content_hash: sha256:0204c71b6a9b586df255be1fea6b84ea0f3ba9211118f0a486a9c893cb06315a
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
  - RawFileReader
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# instrument-qc-metric-aggregation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Aggregates MetaXtract-exported scan-level metadata (TIC, base-peak intensity, ion injection time, charge state, retention time) across RAW files to detect and diagnose instrument underperformance without database searching, as demonstrated on four HeLa QC runs.

## When to use

Use when QCing Thermo RAW LC-MS/MS runs (e.g. HeLa digest standards) and you need to compare scan-level acquisition metrics across files to flag underperforming batches before searching.

## When NOT to use

- The input files are not Thermo Fisher RAW — MetaXtract currently supports only Thermo RAW and additional vendor formats are planned but not yet implemented
- You need QC metrics that require database search results (e.g. identified-peptide-based metrics) — MetaXtract deliberately uses acquisition-side signals only
- You want to export MS-method or LC-method text on a Linux container — those Thermo method-reading calls are not supported on Linux and ms_method/lc_method must be disabled

## Inputs

- Thermo Fisher RAW mass spectrometry files (e.g. Pierce HeLa Protein Digest Standard QC runs)
- MetaXtract YAML configuration file specifying io.input, output_dir, outputs, scan_header column selections and multi_comparison samples

## Outputs

- Per-file scan-header CSV/TSV tables for MS1 and MS2 with PSI-MS-named columns (Scan Start Time (min), Total Ion Current, Base Peak Intensity, Base Peak m/z, Ion Injection Time (ms), Charge State, Filter String)
- MS1 and MS2 peak-list Parquet files (mz_array, intensity_array, plus resolution/noise/baseline/charge arrays for MS2)
- runtime_summary_YYYYMMDD_HHMMSS.tsv with one row per RAW file (status, runtime, start/end/peak memory)
- Aggregated cross-file comparison of TIC, base-peak intensity, retention-time distributions and MS2 charge-state patterns
- Interactive Plotly HTML visualisation reports (optional via visualisation.enabled)

## How to apply

Configure a MetaXtract YAML run listing the RAW files under io.input with outputs file_based_details, ms1_peaklist_export, ms2_peaklist_export and complete MS1/MS2 scan headers set to true, then execute via CLI (`python main.py --config config.yml`). MetaXtract parses each RAW through the Thermo RawFileReader C# library via pythonnet and emits per-file scan-header tables (PSI-MS columns: Scan Start Time (min), Total Ion Current, Base Peak Intensity, Base Peak m/z, Ion Injection Time (ms), Charge State, Filter String) as CSV/TSV plus MS1/MS2 peak lists as Parquet. Aggregate the per-file tables and compare retention-time distributions, TIC, base-peak intensity and MS2 charge-state patterns (2+ to 6+, top-10 DDA, HCD 27%) between acquisition batches (e.g. 19/12/2024 vs 20/02/2025). A large fold-change in signal intensity across batches — the article reports a 9.2-fold reduction in the underperforming HeLa runs — is the primary diagnostic indicator. Validate by checking that runtime_summary_YYYYMMDD_HHMMSS.tsv exists with one row per RAW (status, runtime, peak memory) and that scan-level columns are non-empty and consistent with the acquisition method.

## Related tools

- **MetaXtract** (Primary tool: extracts scan-level metadata and peak lists from Thermo RAW files into structured TSV/CSV and Parquet tables for cross-run QC comparison) — https://github.com/Rappsilber-Laboratory/MetaXtract
- **RawFileReader** (Thermo C# library used underneath MetaXtract to parse the native RAW files)
- **Plotly** (Generates interactive HTML reports for visualising and comparing scan-level metrics across files)

## Examples

```
python main.py --config config.yml  # config.yml: io.input=[ref1.raw, ref2.raw, up1.raw, up2.raw]; outputs: file_based_details/complete-ms1/complete-ms2/peaklist-export=true
```

## Evaluation signals

- runtime_summary_YYYYMMDD_HHMMSS.tsv exists with exactly one row per processed RAW file, recording status, runtime and start/end/peak memory
- Aggregated scan-level columns (retention time, TIC, base-peak intensity, ion injection time, charge state) are non-empty for every scan in every file
- Scan-header column names follow PSI-MS/mzML controlled-vocabulary names (e.g. 'Scan Start Time (min)' for MS:1000016, 'Base Peak Intensity' for MS:1000505); Thermo trailer fields without a PSI-MS match are prefixed with 'thermo_'
- Cross-batch comparison of TIC/base-peak intensity reproduces the expected magnitude of signal difference (article reports a 9.2-fold reduction for the underperforming HeLa runs)
- Each RAW yields its own per-file metadata CSV/TSV and Parquet peak list in the output directory

## Limitations

- MetaXtract currently supports only Thermo Fisher RAW files; additional vendor formats are not yet supported
- Linking QC deviations to instrument-level causes is described as plausible rather than validated in the article
- MS-method and LC-method export options are not supported on Linux containers because the Thermo method-reading calls fail there
- No changelog or version number is stated for MetaXtract

## Evidence

- [results] Scan-level metadata from the four HeLa QC runs reveal a 9.2-fold reduction in signal intensity in the underperforming runs (Figure 2b), very likely the primary reason for the loss of peptide identifications.: [evidence span withheld — anchor status: not_found]
