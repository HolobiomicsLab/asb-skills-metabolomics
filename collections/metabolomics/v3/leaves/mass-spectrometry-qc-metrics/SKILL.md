---
name: mass-spectrometry-qc-metrics
description: Use when evaluating Thermo Fisher RAW-file acquisition quality — e.g.
  verifying that MS2 scan fractions (e.g. 60%/62% reference vs 56%/59% underperforming)
  or scan-level TIC/injection-time trends match expected instrument behaviour.
when_to_use_negative:
- Input is already an mzIdentML 1.3 file or database-search result — MetaXtract monitors
  acquisition-side signals and does not consume search output for QC.
- Files are non-Thermo vendor formats (e.g. Bruker/Waters) — MetaXtract currently
  supports only Thermo Fisher RAW files.
- You need MS-method or LC-method extraction on a Linux container — those Thermo method-reading
  calls are not supported on Linux.
edam_operation: http://edamontology.org/operation_3632
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: MetaXtract
  role: 'Primary tool: reads native Thermo RAW files via RawFileReader and exports
    structured scan-header/QC tables and Plotly HTML reports for near real-time monitoring.'
  repo: https://github.com/Rappsilber-Laboratory/MetaXtract
- name: pandas
  role: 'Post-processing: loads exported scan-header CSV/TSV to classify Scan Mode,
    count MS2 scans, and compute per-run MS2 fractions.'
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1101/2025.11.12.687968
    title: 'MetaXtract: Extracting Metadata from Raw Files for FAIR Data Practices
      and Workflow Optimisation'
  source_content_hash: sha256:ab21e1fcdf6fcffe33668f27a6485f8492c00a24fbe82f788639b2f4ceb70702
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
  - pandas
  edam_operation: http://edamontology.org/operation_3632
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# mass-spectrometry-qc-metrics

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Extracts acquisition-side QC metrics (retention time, total ion current, ion injection time, scan-mode fractions) directly from Thermo RAW files with MetaXtract, enabling search-free, near real-time monitoring of LC-MS performance such as comparing MS2 scan proportions across reference and underperforming HeLa QC runs.

## When to use

Use when evaluating Thermo Fisher RAW-file acquisition quality — e.g. verifying that MS2 scan fractions (e.g. 60%/62% reference vs 56%/59% underperforming) or scan-level TIC/injection-time trends match expected instrument behaviour.

## When NOT to use

- Input is already an mzIdentML 1.3 file or database-search result — MetaXtract monitors acquisition-side signals and does not consume search output for QC.
- Files are non-Thermo vendor formats (e.g. Bruker/Waters) — MetaXtract currently supports only Thermo Fisher RAW files.
- You need MS-method or LC-method extraction on a Linux container — those Thermo method-reading calls are not supported on Linux.

## Inputs

- Thermo Fisher .RAW mass-spectrometry files (e.g. HeLa Pierce Protein Digest QC runs)
- MetaXtract config.yml specifying scan_header columns (e.g. 'Scan Mode', 'Total Ion Current')
- Optional SDRF metadata TSV for provenance annotation

## Outputs

- Per-run TSV/CSV scan-header tables with MS1/MS2 metrics (retention time, TIC, injection time, base peak m/z/intensity)
- File-based-details TSV containing instrument details, scan counts, and run statistics
- MS2 fraction summary (e.g. ms2_scan_proportions.csv: run, MS2 count, total scans, MS2 %)
- Interactive Plotly HTML QC reports

## How to apply

Run MetaXtract on the RAW files to export scan-header tables (TSV/CSV) with columns such as 'Scan Mode', 'Total Ion Current', 'Ion Injection Time (ms)', 'Scan Start Time (min)', and 'Base Peak Intensity'; the GUI allows selecting individual columns or 'complete MS1/MS2'. Classify each scan as MS1 or MS2 via the Scan Mode value, count MS2 and total scans per run, and compute the MS2 fraction (MS2 count / total scans × 100). For file-based summaries (e.g. Figure S3 of the HeLa Pierce digest runs), compare the computed proportions across runs — the article reports 60% (Ref1), 62% (Ref2), 56% (UP1), 59% (UP2) despite similar total MS1/MS2 counts. For near real-time monitoring, rely on acquisition-side signals only (no database search results), enabling immediate troubleshooting after acquisition; structured outputs support integration into automated pipelines and downstream machine-learning workflows.

## Related tools

- **MetaXtract** (Primary tool: reads native Thermo RAW files via RawFileReader and exports structured scan-header/QC tables and Plotly HTML reports for near real-time monitoring.) — https://github.com/Rappsilber-Laboratory/MetaXtract
- **pandas** (Post-processing: loads exported scan-header CSV/TSV to classify Scan Mode, count MS2 scans, and compute per-run MS2 fractions.)

## Examples

```
docker run --rm -v "$(pwd)/data:/data:ro" -v "$(pwd)/out:/out" metaxtract:latest --input /data/small.RAW --output-dir /out --file-based-details --complete-ms2 --ms2-peaklist-export --graphical-representation
```

## Evaluation signals

- Computed MS2 fractions reproduce the reported values: 60% (Ref1), 62% (Ref2), 56% (UP1), 59% (UP2) for the HeLa QC runs (Figure S3).
- Total MS1 and MS2 scan counts are similar across the four runs, so the MS2-fraction difference is attributable to scan-mode distribution, not total scan count.
- Exported scan-header tables contain the expected PSI-MS/mzML-style column names (e.g. 'Total Ion Current', 'MS:1000285'; 'Scan Start Time (min)', 'MS:1000016').
- No database search results are required — QC metrics derive solely from acquisition-side signals in the RAW file.
- Output directory contains one subfolder per RAW file with *_technical_details_ms2_*.csv and, if configured, ms2_peaklist.parquet.

## Limitations

- Currently supports only Thermo Fisher RAW files; additional vendor file formats are not yet supported.
- MS-method and LC-method extraction is not supported on Linux containers.
- MetaXtract is not yet integrated as a Proteome Discoverer node (planned for future versions).
- No version number stated for MetaXtract in the documentation.
- Linking QC deviations to instrument-level causes is described as plausible rather than validated.

## Evidence

- [results] MS2 fraction differences: [evidence span withheld — anchor status: not_found]
