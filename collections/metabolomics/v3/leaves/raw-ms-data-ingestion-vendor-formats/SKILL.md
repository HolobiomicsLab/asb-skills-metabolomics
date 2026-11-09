---
name: raw-ms-data-ingestion-vendor-formats
description: Use when the input consists of raw, unconverted MS files in vendor formats
  (Agilent 'd', Thermo '.raw', Bruker 'd', or mzML) from LC-MS, LC-IMS-MS, DDA/DIA,
  or direct infusion acquisitions that need to enter an automated QC pipeline.
when_to_use_negative:
- Data are already converted to open formats or only peak/feature tables are available
  — the pipeline starts from raw vendor files
- The task is identification-based QC (e.g., spectral-match counting) rather than
  omics-agnostic peak-metric and outlier QC
- The instrument or acquisition scheme is not among the supported types (LC-MS, LC-IMS-MS,
  DDA/DIA with/without fragmentation, direct infusion)
tools:
- name: IonToolPack
  role: GUI suite that ingests the raw vendor-format MS files and launches the PeakQC
    pipeline
  repo: https://github.com/pnnl/IonToolPack
- name: MZA
  role: Data conversion tool performing the raw-data-to-mzML conversion step at the
    start of the PeakQC pipeline
  repo: https://github.com/pnnl/IonToolPack
- name: PeakQC
  role: Downstream QC pipeline consuming the ingested raw data (conversion → MS1 imaging/PCA
    → peak metrics → error heatmaps)
  repo: https://github.com/pnnl/IonToolPack
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/jasms.4c00146
    title: 'PeakQC: A Software Tool for Omics-Agnostic Automated Quality Control of
      Mass Spectrometry Data'
  source_content_hash: sha256:1a0776ec07806aea5467b1880d13cf400f519bb4cc21eb146f25b7f4474f60ee
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/pnnl/IonToolPack
  tools:
  - IonToolPack
  - MZA
  - PeakQC
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
  techniques:
  - direct-infusion-MS
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# raw-ms-data-ingestion-vendor-formats

## Summary

Ingest raw MS files from multiple instrument vendors (Agilent 'd', Thermo '.raw', Bruker 'd', and mzML) into the PeakQC/IonToolPack QC pipeline, which internally performs raw data conversion as the first step before MS1 time-versus-m/z imaging, peak metric extraction, and error heatmap generation.

## When to use

Use when the input consists of raw, unconverted MS files in vendor formats (Agilent 'd', Thermo '.raw', Bruker 'd', or mzML) from LC-MS, LC-IMS-MS, DDA/DIA, or direct infusion acquisitions that need to enter an automated QC pipeline.

## When NOT to use

- Data are already converted to open formats or only peak/feature tables are available — the pipeline starts from raw vendor files
- The task is identification-based QC (e.g., spectral-match counting) rather than omics-agnostic peak-metric and outlier QC
- The instrument or acquisition scheme is not among the supported types (LC-MS, LC-IMS-MS, DDA/DIA with/without fragmentation, direct infusion)

## Inputs

- Agilent 'd' raw MS files
- Thermo '.raw' files
- Bruker 'd' directories
- mzML files
- Bundled example files from the IonToolPack test_data directory

## Outputs

- Converted data feeding the PeakQC pipeline
- MS1 time-versus-m/z images with a PCA plot of the batch
- CSV metric reports for auto-tracked and user-specified ions
- JPG/PDF ion metric figures
- Isolation Forest error heatmaps
- Mobiligrams for user-specified ions when IMS separation is present

## How to apply

Download and decompress the latest IonToolPack release from the GitHub Releases section and launch IonToolPack.exe; no installation is required. Import the raw MS files through the GUI (supported: Agilent 'd', Thermo '.raw', Bruker 'd', and mzML), which may come from infusion experiments or LC/IMS front-end separations with or without fragmentation. Click 'Process' to trigger the full pipeline, in which raw data conversion (via the MZA conversion tool) precedes MS1 time-versus-m/z image generation and PCA, extraction of peak metrics for auto-tracked and user-specified ions, and Isolation Forest error heatmap generation. Validate ingestion by confirming all expected output products appear in navigable output folders: CSV metric reports, JPG/PDF ion metric figures, and error heatmaps, ideally compared against the example input/output files in the repository's test_data directory.

## Related tools

- **IonToolPack** (GUI suite that ingests the raw vendor-format MS files and launches the PeakQC pipeline) — https://github.com/pnnl/IonToolPack
- **MZA** (Data conversion tool performing the raw-data-to-mzML conversion step at the start of the PeakQC pipeline) — https://github.com/pnnl/IonToolPack
- **PeakQC** (Downstream QC pipeline consuming the ingested raw data (conversion → MS1 imaging/PCA → peak metrics → error heatmaps)) — https://github.com/pnnl/IonToolPack

## Examples

```
IonToolPack.exe
```

## Evaluation signals

- All expected outputs present after clicking 'Process': time-versus-m/z images with PCA plot, CSV metric reports, JPG/PDF ion metric figures, and error heatmaps
- Generated outputs match the example input and output files published in the repository's test_data folder
- Files in each of the supported vendor formats (Agilent 'd', Thermo '.raw', Bruker 'd', mzML) are read without import errors
- Total execution time spans all steps from raw data conversion to error heatmap generation, confirming conversion occurred
- Output products are organized in navigable output folders rather than dumped in a single directory

## Limitations

- Vendor format support is limited to Agilent 'd', Thermo '.raw', Bruker 'd', and mzML; other vendor formats are not mentioned
- The pipeline is GUI-driven (IonToolPack.exe); command-line functionality for unattended QC is only planned, not yet available
- Mobiligram extraction requires arrival time (AT) in the input and IMS separation in the MS data

## Evidence
