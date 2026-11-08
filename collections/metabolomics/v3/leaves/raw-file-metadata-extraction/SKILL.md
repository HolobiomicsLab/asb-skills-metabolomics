---
name: raw-file-metadata-extraction
description: Use when input is one or more Thermo Fisher .RAW files and the goal is
  to obtain structured, analysis-ready metadata (LC-MS method, scan headers, TIC,
  base-peak intensity, ion injection time, charge state) without running a database
  search — e.
when_to_use_negative:
- Input is mzML, mzIdentML or any non-Thermo vendor format — MetaXtract reads native
  Thermo RAW only via RawFileReader; additional vendor formats are not yet supported.
- A completed, searchable SDRF is required immediately — only RAW-derived fields can
  be auto-filled (draft mode); users must complete SDRF metadata manually afterward.
- Linux container/HPC and you need the MS method or LC method text exports — those
  Thermo method-reading calls are not supported on Linux and should be disabled in
  the config.
edam_operation: http://edamontology.org/operation_3359
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_3520
tools:
- name: MetaXtract
  role: Primary tool that parses Thermo RAW files via RawFileReader through pythonnet,
    extracts scan headers, technical trailer details, and MS1/MS2 peak arrays, and
    exports them as CSV/TSV, Parquet, SDRF and Plotly HTML reports.
  repo: https://github.com/Rappsilber-Laboratory/MetaXtract
- name: RawFileReader
  role: Thermo Fisher C# library used by MetaXtract (invoked via pythonnet) to read
    the native RAW file contents; provides GetMoreMSInfos, ExportPeakList and ExportMS1PeakList
    calls.
- name: Plotly
  role: Generates the interactive HTML reports visualizing MS1/MS2 scans and cross-sample
    comparisons when visualisation.enabled=true.
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1101/2025.11.12.687968
    title: 'MetaXtract: Extracting Metadata from Raw Files for FAIR Data Practices
      and Workflow Optimisation'
  source_content_hash: sha256:49601386194c515f1706efa55708797746713b1c82e2f0c394aecbb97f86afd3
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
  edam_operation: http://edamontology.org/operation_3359
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4166666666666667
n_claims: 12
---

# raw-file-metadata-extraction

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Extract structured acquisition metadata and scan-level spectral arrays directly from Thermo Fisher RAW files with MetaXtract, bypassing database searching. Produces PSI-MS-style CSV/TSV scan headers, Parquet peak lists, and runtime summaries that support search-free QC, provenance capture, and downstream data-driven modelling.

## When to use

Use when input is one or more Thermo Fisher .RAW files and the goal is to obtain structured, analysis-ready metadata (LC-MS method, scan headers, TIC, base-peak intensity, ion injection time, charge state) without running a database search — e.g. instrument QC, RAW-file annotation for FAIR/repository submission, or training data for spectral-quality or retention-time models.

## When NOT to use

- Input is mzML, mzIdentML or any non-Thermo vendor format — MetaXtract reads native Thermo RAW only via RawFileReader; additional vendor formats are not yet supported.
- A completed, searchable SDRF is required immediately — only RAW-derived fields can be auto-filled (draft mode); users must complete SDRF metadata manually afterward.
- Linux container/HPC and you need the MS method or LC method text exports — those Thermo method-reading calls are not supported on Linux and should be disabled in the config.

## Inputs

- Thermo Fisher RAW mass-spectrometry files (one or more .RAW)
- YAML configuration file (config.yml) specifying io.input paths, outputs toggles, scan-header column selection, and optional SDRF draft

## Outputs

- Per-file CSV/TSV scan-header tables with PSI-MS/mzML-style column names (Scan Start Time (min), Total Ion Current, Base Peak Intensity, Base Peak m/z, Ion Injection Time (ms), Charge State, Filter String)
- *_technical_details_ms2_*.csv files from GetMoreMSInfos
- MS2 peak lists as Parquet (mz_array, intensity_array, resolution_array, noises_array, baselines_array, charges_array per scan)
- MS1 peak lists as Parquet (mz_array, intensity_array with centroid/profile flag)
- SDRF-Proteomics metadata file (metadata.sdrf.tsv or metadata.sdrf.draft.tsv)
- runtime_summary_YYYYMMDD_HHMMSS.tsv with status, runtime, start/end/peak memory per processed RAW file
- Interactive Plotly HTML visual reports when visualisation.enabled=true

## How to apply

1. Write a YAML config listing the Thermo RAW file paths under io.input and setting the desired outputs (file_based_details, complete-ms1/complete-ms2 scan headers, ms1_peaklist_export, ms2_peaklist_export) to true. 2. Run MetaXtract via CLI (python main.py --config config.yml) or Docker/Apptainer container; RAW parsing goes through the RawFileReader C# library invoked via pythonnet. 3. Collect per-file scan-header tables — PSI-MS-style columns such as Scan Start Time (min), Total Ion Current, Base Peak Intensity, Base Peak m/z, Ion Injection Time (ms), Charge State, Filter String — as CSV/TSV, plus *_technical_details_ms2_*.csv from GetMoreMSInfos. 4. If extended spectral arrays are needed, export MS2 peak lists (mz_array, intensity_array, resolution_array, noises_array, baselines_array, charges_array) and MS1 peak lists (mz_array, intensity_array) as Parquet via ExportPeakList and ExportMS1PeakList. 5. Aggregate the per-file tables across inputs to compare acquisition-side signals (e.g. TIC, base-peak intensity, retention-time distribution, 2+–6+ charge-state pattern) between runs — the paper shows this alone reveals instrument underperformance (a 9.2-fold intensity drop in the HeLa QC runs). 6. Verify a runtime_summary_YYYYMMDD_HHMMSS.tsv exists with one row per processed RAW file and that exported columns are non-empty, consistent with the reported DDA top-10 / 2+-6+ / HCD 27% settings.

## Related tools

- **MetaXtract** (Primary tool that parses Thermo RAW files via RawFileReader through pythonnet, extracts scan headers, technical trailer details, and MS1/MS2 peak arrays, and exports them as CSV/TSV, Parquet, SDRF and Plotly HTML reports.) — https://github.com/Rappsilber-Laboratory/MetaXtract
- **RawFileReader** (Thermo Fisher C# library used by MetaXtract (invoked via pythonnet) to read the native RAW file contents; provides GetMoreMSInfos, ExportPeakList and ExportMS1PeakList calls.)
- **Plotly** (Generates the interactive HTML reports visualizing MS1/MS2 scans and cross-sample comparisons when visualisation.enabled=true.)

## Examples

```
python main.py --config config.yml
```

## Evaluation signals

- runtime_summary_YYYYMMDD_HHMMSS.tsv exists with exactly one row per input RAW file, each carrying status, runtime and start/end/peak memory values.
- Each exported scan-header CSV/TSV contains non-empty values for the mandatory acquisition columns (Scan Start Time (min), Total Ion Current, Base Peak Intensity, Ion Injection Time (ms), Charge State) matching the acquisition method (e.g. DDA top-10, 2+–6+, HCD 27%).
- Column names follow the documented PSI-MS/mzML mapping (e.g. 'Base Peak m/z' not the legacy 'Base Peak Mass'; 'Scan Start Time (min)' not the ambiguous 'Retention Time (s)').
- Parquet peak lists parse successfully with pyarrow and each scan row exposes the expected arrays (MS2: mz, intensity, resolution, noise, baseline, charge; MS1: mz, intensity) indexed by scan number.
- Cross-run comparison of scan-level TIC / base-peak intensity reproduces the expected order-of-magnitude behaviour (e.g. a large fold-change between reference and underperforming HeLa QC runs rather than near-identical distributions).

## Limitations

- Supports only Thermo Fisher RAW files; additional vendor file formats are not yet supported (planned for future versions).
- MS-method and LC-method extraction ('*_MS_method.txt', '*_LC_method.txt') are not supported on Linux containers and must be disabled there.
- In draft mode metadata.sdrf.draft.tsv contains only RAW-derived fields; SDRF metadata must be completed manually by the user.
- No explicit version number or changelog is published for the tool.
- Interpretation of metadata-derived QC deviations is a diagnostic hint (e.g. intensity drop ⇒ instrument underperformance), not a validated root-cause assignment.
- Linking scan-level metadata with search results in a single interactive view is only 'potentially' supported.

## Evidence
