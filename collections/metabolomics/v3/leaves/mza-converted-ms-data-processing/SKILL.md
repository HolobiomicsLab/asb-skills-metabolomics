---
name: mza-converted-ms-data-processing
description: Use when raw vendor MS files (Agilent .d, Thermo .raw, Bruker .d, or
  mzML) must be converted with MZA and processed through PeakQC's MS1 QC pipeline
  for omics-agnostic batch quality assessment.
when_to_use_negative:
- Input is already a processed feature table or CSV of quantified features rather
  than raw or MZA-converted MS data (Comparador is the comparison tool for such feature
  lists).
- The analysis goal is identification-based QC that relies on spectral matches (e.g.,
  ID-based proteomics QC metrics) rather than omics-agnostic MS1 peak-metric QC.
edam_operation: http://edamontology.org/operation_3219
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: PeakQC
  role: Performs the omics-agnostic automated QC pipeline (MS1 PCA, auto-tracked ion
    selection, peak metric extraction, Isolation Forest outlier detection) on the
    MZA-converted data.
  repo: https://github.com/pnnl/IonToolPack
- name: MZA
  role: Data conversion tool that transforms vendor raw multidimensional MS data into
    a format suitable for downstream software development and processing in the pipeline.
- name: IonToolPack
  role: GUI suite housing PeakQC; reads Agilent .d, Thermo .raw, Bruker .d, and mzML
    data and drives the 'Process' step from raw import to QC reports.
  repo: https://github.com/pnnl/IonToolPack
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1021/jasms.4c00146
    title: 'PeakQC: A Software Tool for Omics-Agnostic Automated Quality Control of
      Mass Spectrometry Data'
  source_content_hash: sha256:4f2f9d5d2141531f252d5e2e6c35ff33527c4579a6f2f1512d1832d20cea8841
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/pnnl/IonToolPack
  tools:
  - IonToolPack
  - MZA
  - PeakQC
  edam_operation: http://edamontology.org/operation_3219
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9
n_claims: 20
---

# mza-converted-ms-data-processing

## Summary

Converts vendor raw MS files into MZA format and runs them through PeakQC's (IonToolPack) MS1-based QC pipeline, covering raw-data conversion, MS1 parsing into time-versus-m/z images, auto-tracked ion selection, and QC report generation.

## When to use

Use when raw vendor MS files (Agilent .d, Thermo .raw, Bruker .d, or mzML) must be converted with MZA and processed through PeakQC's MS1 QC pipeline for omics-agnostic batch quality assessment.

## When NOT to use

- Input is already a processed feature table or CSV of quantified features rather than raw or MZA-converted MS data (Comparador is the comparison tool for such feature lists).
- The analysis goal is identification-based QC that relies on spectral matches (e.g., ID-based proteomics QC metrics) rather than omics-agnostic MS1 peak-metric QC.

## Inputs

- Raw vendor MS files: Agilent .d, Thermo .raw, Bruker .d, or mzML
- MZA-converted MS data files
- Optional user-specified ion target list (theoretical or reference m/z, RT/arrival-time values, and expected fragment ions/intensities for MS2 diagnostics)

## Outputs

- MZA-converted multidimensional MS data files
- Time-versus-m/z MS1 images per run for PCA of the batch
- Auto-tracked ion list (topmost abundant ions, most frequent per sample group)
- CSV reports of peak metrics (mass error, retention time error, arrival time error)
- JPG/PDF diagnostic figures and error heatmaps (Isolation Forest outliers per sample group)

## How to apply

First convert the raw vendor data (Agilent .d, Thermo .raw, Bruker .d, or mzML inputs) using MZA, since raw data conversion is the first step of the pipeline before error heatmap generation. Load the MZA-converted data into the IonToolPack GUI (double-click IonToolPack.exe from the GitHub Releases download; no installation is required), import the files, and click 'Process'. PeakQC then represents each MS run in the batch as a time-versus-m/z image generated from the MS1 data for PCA-based global quality assessment, and finds the topmost abundant ions in the MS1 data that are more frequent per sample group, using them as 'auto-tracked' ions. Peak metrics (mass error, retention time error, arrival time error) are extracted for both auto-tracked and user-specified ions, with outlier detection via the Isolation Forest algorithm. The pipeline works across acquisition types: LC-MS, LC-IMS-MS, direct infusion, with/without fragmentation in DDA or DIA mode.

## Related tools

- **PeakQC** (Performs the omics-agnostic automated QC pipeline (MS1 PCA, auto-tracked ion selection, peak metric extraction, Isolation Forest outlier detection) on the MZA-converted data.) — https://github.com/pnnl/IonToolPack
- **MZA** (Data conversion tool that transforms vendor raw multidimensional MS data into a format suitable for downstream software development and processing in the pipeline.)
- **IonToolPack** (GUI suite housing PeakQC; reads Agilent .d, Thermo .raw, Bruker .d, and mzML data and drives the 'Process' step from raw import to QC reports.) — https://github.com/pnnl/IonToolPack

## Examples

```
IonToolPack.exe  # launch GUI; import raw MS files (Agilent .d, Thermo .raw, Bruker .d, mzML) and click "Process"
```

## Evaluation signals

- Raw data conversion through error heatmap generation completes without error for all files in the batch.
- The auto-tracked ion list consists of the topmost abundant MS1 ions that are most frequent per sample group.
- Each MS run in the batch yields a time-versus-m/z MS1 image usable for PCA.
- Generated CSV peak-metric reports and JPG/PDF figures exist for both auto-tracked and user-specified ions.
- Produced outputs match the example input/output files in the IonToolPack repository's test_data folder.

## Limitations

- No standalone CLI/installation instructions (pip/conda/docker) are documented; usage is via the IonToolPack.exe GUI downloaded from Releases.
- The article's illustrative datasets are representative but the README notes no changelog; some features (e.g., command-line functionality for unattended QC across batches) are planned rather than available.
- Mobiligram extraction requires arrival-time (AT) information in the input and MS data containing IMS separation.

## Evidence
