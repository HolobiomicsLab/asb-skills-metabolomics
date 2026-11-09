---
name: cross-run-acquisition-pattern-comparison
description: Use when several Thermo RAW files from nominally identical LC-MS/MS QC
  or batch acquisitions must be compared scan-by-scan (retention time, TIC, base-peak
  intensity, ion injection time, charge state) to find underperforming runs.
when_to_use_negative:
- Input files are not native Thermo RAW (MetaXtract currently supports only Thermo
  Fisher RAW files; additional vendor formats are not yet supported)
- The question requires peptide/protein identifications rather than acquisition-side
  QC signals — this skill is identification-free
- On Linux containers, do not expect ms_method or lc_method exports since those Thermo
  method-reading calls are unsupported there
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: MetaXtract
  role: Extracts per-scan metadata (retention time, TIC, base-peak intensity, ion
    injection time, charge state) from each Thermo RAW file and renders cross-sample
    comparison plots
  repo: https://github.com/Rappsilber-Laboratory/MetaXtract
- name: RawFileReader
  role: Thermo C# library invoked through pythonnet to parse the RAW files before
    metadata transformation
- name: Plotly
  role: Generates the interactive HTML visualisations used for cross-run comparison
    of scan-level metrics
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1101/2025.11.12.687968
    title: 'MetaXtract: Extracting Metadata from Raw Files for FAIR Data Practices
      and Workflow Optimisation'
  source_content_hash: sha256:52f98ee6ea16be5a20604a2af6112fa5f95e01c471f2852eca0e2f7007f49f16
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
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9333333333333333
n_claims: 15
---

# Cross-run acquisition pattern comparison

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Compare scan-level metadata exported from multiple Thermo RAW files (e.g. four Pierce HeLa Protein Digest Standard QC runs) using MetaXtract's CSV/TSV scan headers, TIC and base-peak intensity columns, and Plotly cross-sample plots to detect and diagnose instrument underperformance without database searching.

## When to use

Use when several Thermo RAW files from nominally identical LC-MS/MS QC or batch acquisitions must be compared scan-by-scan (retention time, TIC, base-peak intensity, ion injection time, charge state) to find underperforming runs.

## When NOT to use

- Input files are not native Thermo RAW (MetaXtract currently supports only Thermo Fisher RAW files; additional vendor formats are not yet supported)
- The question requires peptide/protein identifications rather than acquisition-side QC signals — this skill is identification-free
- On Linux containers, do not expect ms_method or lc_method exports since those Thermo method-reading calls are unsupported there

## Inputs

- Two or more Thermo Fisher .RAW files from replicate/QC LC-MS/MS runs (e.g. Pierce HeLa Protein Digest Standard, DDA top-10, 2+-6+ charge states, HCD NCE 27%)
- A MetaXtract YAML config listing all RAW files under io.input with scan_header and multi_comparison settings

## Outputs

- Per-RAW-file MS1/MS2 scan-header CSV/TSV tables with PSI-MS-named columns (Scan Start Time (min), Total Ion Current, Base Peak Intensity, Ion Injection Time (ms), Charge State)
- runtime_summary_YYYYMMDD_HHMMSS.tsv with one row per processed RAW file (status, runtime, start/end/peak memory)
- MS1/MS2 peak lists as Parquet (mz_array, intensity_array, plus resolution/noise/baseline/charge arrays for MS2)
- Cross-sample Plotly HTML comparison reports

## How to apply

Run MetaXtract (CLI via a YAML config listing all RAW files under io.input, or the Linux container) with file_based_details, complete MS1/MS2 scan headers, and ms1_peaklist_export/ms2_peaklist_export enabled, so each RAW file yields per-file scan-header tables with PSI-MS-style columns (Scan Start Time (min), Total Ion Current, Base Peak Intensity, Base Peak m/z, Ion Injection Time (ms), Charge State, Filter String). Aggregate the per-file tables and stratify scans by MS level, then compare retention-time distributions, TIC, base-peak intensity, ion injection time, and MS2 charge-state patterns (2+ to 6+) across runs (e.g. reference vs underperforming acquisition dates). Enable multi_comparison with 1-based sample positions (samples: [1, 2, 3]) to trigger cross-sample visual comparison, exported as interactive Plotly HTML reports. Rationale: acquisition-side signals are sufficient for search-free, near real-time performance monitoring, so no mzIdentML or database-search results are needed; in the HeLa QC demonstration this revealed a 9.2-fold signal-intensity reduction explaining lost peptide identifications.

## Related tools

- **MetaXtract** (Extracts per-scan metadata (retention time, TIC, base-peak intensity, ion injection time, charge state) from each Thermo RAW file and renders cross-sample comparison plots) — https://github.com/Rappsilber-Laboratory/MetaXtract
- **RawFileReader** (Thermo C# library invoked through pythonnet to parse the RAW files before metadata transformation)
- **Plotly** (Generates the interactive HTML visualisations used for cross-run comparison of scan-level metrics)

## Examples

```
python main.py --config /path/to/config.yml  # io.input lists all four HeLa QC RAW files; outputs.complete_ms1/complete_ms2/ms1_peaklist_export/ms2_peaklist_export = true; multi_comparison.enabled = true with samples: [1, 2, 3, 4]
```

## Evaluation signals

- runtime_summary_YYYYMMDD_HHMMSS.tsv exists with exactly one row per input RAW file, each marked completed with non-zero runtime and memory values
- Every aggregated scan-level row has non-empty Scan Start Time (min), Total Ion Current, Base Peak Intensity, Ion Injection Time (ms) and Charge State values
- Charge-state distribution per run is restricted to the expected acquisition range (2+ to 6+) for the DDA top-10 HeLa QC method
- Retention-time distributions span the LC gradient window consistently across runs (2% to 95% B over ~47 min gradient)
- A quantified intensity contrast between reference and underperforming runs reproduces the expected fold-change in signal intensity (e.g. ~9.2-fold reduction in the HeLa QC example)

## Limitations

- Currently supports only Thermo Fisher RAW files; additional vendor file formats are not yet supported
- Linking QC deviations to instrument-level causes is described as plausible rather than validated
- No changelog or version number is provided for MetaXtract, so exact behaviour may vary across checkouts
- MS-method and LC-method exports are not supported on Linux containers

## Evidence

- [abstract] MetaXtract enables search-free, near real-time performance monitoring: "MetaXtract enables search-free, near real-time performance monitoring by relying on acquisition-side signals"
- [abstract] Scan-level metrics captured: "scan-level metrics such as retention time, total ion current, and ion injection time"
- [readme] Cross-sample comparison feature: "Cross-sample visual comparisons"
- [readme] multi_comparison selection syntax: "Select any 2 or more inputs using their 1-based positions in io.input."
- [readme] Retention time unit semantics: "Thermo returns retention time in minutes. Replaces old ambiguous `Retention Time (s)` / `Retention Time (min)`."
- [methods] Plotly visualisation step: "visualization of mass spectrometry plots using Plotly."
- [discussion] Causal-linking caveat: "linking deviations to plausible instrument-level causes"
