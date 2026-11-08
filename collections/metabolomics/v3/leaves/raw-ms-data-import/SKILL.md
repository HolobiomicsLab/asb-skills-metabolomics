---
name: raw-ms-data-import
description: Use when the input consists of raw MS acquisition files (vendor-native
  directories/files or mzML) from infusion, LC-MS, LC-IMS-MS, DDA, or DIA experiments
  and the goal is to load them into PeakQC for automated omics-agnostic quality control.
when_to_use_negative:
- The data are already processed feature tables (e.g., a CSV of m/z–RT features from
  another tool) rather than raw acquisition files — import raw files or feature lists
  through the appropriate IonToolPack tool instead.
- The files are fragment/spectral library formats (MSP/CSV) intended for MS/MS library
  matching — use IonToolPack's TandemMatch, not PeakQC's raw import.
- Only MS2-level spectra without any MS1 survey scans are needed for downstream analysis
  — PeakQC's initial PCA imaging is built on MS1 data, so MS1-deficient inputs will
  not support the global quality assessment.
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: IonToolPack
  role: The launcher GUI that houses PeakQC; it reads the raw MS files in the supported
    formats and runs the PeakQC pipeline on them after the user clicks 'Process'.
  repo: https://github.com/pnnl/IonToolPack
- name: PeakQC
  role: The automated QC tool inside IonToolPack that consumes the imported raw MS
    data and generates the per-run time-versus-m/z images for downstream PCA.
  repo: https://github.com/pnnl/IonToolPack
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1021/jasms.4c00146
    title: 'PeakQC: A Software Tool for Omics-Agnostic Automated Quality Control of
      Mass Spectrometry Data'
  source_content_hash: sha256:8d927f881625290bfc2fe27bf1ef71e9d56e0c5cf5f15872ad845b1e1905f8b5
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/pnnl/IonToolPack
  tools:
  - IonToolPack
  - PeakQC
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# raw-ms-data-import

## Summary

Import raw mass spectrometry data files (Agilent 'd', Thermo '.raw', Bruker 'd', or mzML) into IonToolPack's PeakQC pipeline so that subsequent quality-control analyses (time-versus-m/z imaging, PCA, ion-metric extraction) can run on each MS run.

## When to use

Use when the input consists of raw MS acquisition files (vendor-native directories/files or mzML) from infusion, LC-MS, LC-IMS-MS, DDA, or DIA experiments and the goal is to load them into PeakQC for automated omics-agnostic quality control.

## When NOT to use

- The data are already processed feature tables (e.g., a CSV of m/z–RT features from another tool) rather than raw acquisition files — import raw files or feature lists through the appropriate IonToolPack tool instead.
- The files are fragment/spectral library formats (MSP/CSV) intended for MS/MS library matching — use IonToolPack's TandemMatch, not PeakQC's raw import.
- Only MS2-level spectra without any MS1 survey scans are needed for downstream analysis — PeakQC's initial PCA imaging is built on MS1 data, so MS1-deficient inputs will not support the global quality assessment.

## Inputs

- Raw MS acquisition files: Agilent 'd' directories
- Thermo '.raw' files
- Bruker 'd' directories
- mzML files
- MS data from LC-MS, LC-IMS-MS, DDA, DIA, or direct infusion acquisitions

## Outputs

- Per-sample time-versus-m/z JPG images generated from MS1 data (the input for PCA of the batch)
- CSV reports and JPG/PDF figures containing peak metrics for auto-tracked and user-specified ions

## How to apply

Open IonToolPack (no installation required — download the latest IonToolPack.exe from the Releases page, decompress it, and double-click to launch). In the PeakQC tab, import the raw MS files for each sample run: IonToolPack reads Agilent 'd' directories, Thermo '.raw' files, Bruker 'd' directories, and mzML files directly, so no prior vendor-to-open-format conversion is necessary. Select or load the acquisitions (LC-MS, LC-IMS-MS, DDA/DIA, or direct infusion) and click 'Process' to trigger PeakQC's fast rendering algorithm on the MS1 data, producing the time-versus-m/z image representation used downstream for PCA of the batch. Raw data conversion is treated as the first step of the pipeline before error heatmap generation, so import failures will cascade to every later QC output.

## Related tools

- **IonToolPack** (The launcher GUI that houses PeakQC; it reads the raw MS files in the supported formats and runs the PeakQC pipeline on them after the user clicks 'Process'.) — https://github.com/pnnl/IonToolPack
- **PeakQC** (The automated QC tool inside IonToolPack that consumes the imported raw MS data and generates the per-run time-versus-m/z images for downstream PCA.) — https://github.com/pnnl/IonToolPack

## Examples

```
IonToolPack.exe  # then in PeakQC: Import raw MS files → click 'Process'
```

## Evaluation signals

- After import + 'Process', each input run should yield one time-versus-m/z JPG image (per-sample image set used downstream for PCA of the batch of samples).
- The pipeline's first stage (raw data conversion) should complete without errors before any later outputs (peak-metric CSV reports, error heatmaps) appear; total execution time is reported from raw data conversion to error heatmap generation.
- The accepted input set should match the documented formats — Agilent 'd', Thermo '.raw', Bruker 'd', and mzML — and the acquisition types LC-MS, LC-IMS-MS, DDA/DIA, or direct infusion.
- Example input and output files for validating a successful import are available in the repository's test_data directory.
- Peak metrics extracted for auto-tracked ions and user-specified ions should populate the CSV reports after the imported data are processed.

## Limitations

- The time-versus-m/z image representation is a log-transformed intensity rendering that distorts individual peaks — it is sufficient for global similarity patterns but not for peak-level inspection.
- No explicit standalone installation instructions (pip/conda/docker) are provided in the article; IonToolPack is distributed as a Windows executable from the Releases page.
- The article does not document a changelog or version history for IonToolPack/PeakQC.
- Whether data are converted internally (e.g., via MZA) before PeakQC processing is not detailed; only 'raw data conversion' is named as a pipeline step without specifying the conversion tool.

## Evidence
