---
name: vendor-to-mzml-format-conversion
description: Use when starting from proprietary vendor raw files of MRM/DDA/DIA datasets
  that must be converted to an open format (mzML, Aird) before analysis or when benchmarking
  storage volume across vendor, mzML, and Aird representations.
when_to_use_negative:
- Input is already mzML, mzXML, or Aird — conversion is only for proprietary vendor
  raw files
- The task targets accurate quantification rather than format standardization or storage
  benchmarking — use an analysis tool like MRMPro or Skyline on already-converted
  data instead
- The dataset relies on acquisition modes outside AirdPro's supported set (unsupported
  vendor-specific formats) — verify method detection before batch conversion
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_0121
tools:
- name: MSConvert
  role: ProteoWizard converter used to produce 32-bit mzML from vendor raw files of
    each evaluation dataset
- name: AirdPro
  role: GUI client used to convert the same vendor raw files to the Aird format so
    on-disk volume can be compared against vendor and mzML
  repo: https://github.com/CSi-Studio/AirdPro/releases/
- name: Aird-SDK
  role: Java/C#/Python SDK for reading the .aird/. pair produced by the conversion,
    for downstream secondary development
  repo: https://github.com/CSi-Studio/Aird-SDK
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1186/s12859-024-05685-x
    title: 'MRMPro: a web-based tool to improve the speed of manual calibration for
      multiple reaction monitoring data analysis by mass spectrometry'
  source_content_hash: sha256:5fadc3fdc0776dcc9c291640ae4f7591e98bc1073edcf45be0237b1a1169b32f
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Aird-SDK
  - AirdPro
  - MRMPro
  - MSConvert
  - Skyline
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# vendor-to-mzml-format-conversion

## Summary

Converts vendor mass spectrometry raw files (e.g. SCIEX .wiff, Thermo .raw, Agilent .d) to open 32-bit mzML with MSConvert, and optionally to Aird with AirdPro, so that MRM datasets from PRIDE (PXD009543, PXD031038) and Skyline test data can be compared by on-disk volume and consumed by downstream analysis tools.

## When to use

Use when starting from proprietary vendor raw files of MRM/DDA/DIA datasets that must be converted to an open format (mzML, Aird) before analysis or when benchmarking storage volume across vendor, mzML, and Aird representations.

## When NOT to use

- Input is already mzML, mzXML, or Aird — conversion is only for proprietary vendor raw files
- The task targets accurate quantification rather than format standardization or storage benchmarking — use an analysis tool like MRMPro or Skyline on already-converted data instead
- The dataset relies on acquisition modes outside AirdPro's supported set (unsupported vendor-specific formats) — verify method detection before batch conversion

## Inputs

- Vendor raw MS files from a PRIDE or Skyline dataset (e.g. .wiff/.wiff2, Thermo .raw, Agilent .d folders)
- Acquisition-method context (MRM/SRM, DDA, DIA/SWATH, PRM, DDA_PASEF/DIA_PASEF) used for automatic method detection

## Outputs

- 32-bit mzML files per raw input (from MSConvert)
- .aird data files with paired . index files (from AirdPro)
- storage_volume_comparison.csv with per-dataset volumes and Aird-vs-vendor / Aird-vs-mzML ratios

## How to apply

Download the vendor raw files (e.g. PXD009543 and PXD031038 from PRIDE, plus the Skyline example MRM dataset) and run MSConvert (ProteoWizard) on them to produce 32-bit mzML. For the compression comparison, run the same vendor files through AirdPro, the C# GUI client based on pwiz_bindings_cli.dll, which supports batch/directory-scan conversion and auto-detects the acquisition method (MRM/SRM, DIA/SWATH, DDA, PRM, PASEF). AirdPro writes a paired .aird data file and . index file that must stay in the same directory with the same base name. Record total on-disk volume per format and compute Aird/Vendor and Aird/mzML ratios; the paper expects the compression algorithm to reduce volume by more than 60% vs vendor and more than 80% vs mzML (measured: Aird was 40%/18% of vendor/mzML on PXD009543, 5%/20% on Skyline, 15%/14% on PXD031038). AirdPro V6 also claims ~20% higher compression for ion mobility raw files, which matters if the dataset includes PASEF data.

## Related tools

- **MSConvert** (ProteoWizard converter used to produce 32-bit mzML from vendor raw files of each evaluation dataset)
- **AirdPro** (GUI client used to convert the same vendor raw files to the Aird format so on-disk volume can be compared against vendor and mzML) — https://github.com/CSi-Studio/AirdPro/releases/
- **Aird-SDK** (Java/C#/Python SDK for reading the .aird/. pair produced by the conversion, for downstream secondary development) — https://github.com/CSi-Studio/Aird-SDK

## Examples

```
msconvert PXD009543_raw/*.wiff --mzML --32 --outdir mzml/ && AirdPro.exe --input PXD009543_raw --output aird/ --autoAcquisitionMethod
```

## Evaluation signals

- Aird-to-vendor volume ratio ≤ 0.40 and Aird-to-mzML ratio ≤ 0.20 per dataset, consistent with the reported >60% and >80% reductions
- Expected per-dataset ratios reproduce: PXD009543 ≈ 40%/18%, Skyline test set ≈ 5%/20%, PXD031038 ≈ 15%/14%
- Each converted Aird file has a matching . index with the same base name in the same directory so AirdScanUtil can scan both
- mzML output is 32-bit (check precision annotation in the mzML header), and spectrum counts match the vendor originals
- AirdPro's AirdInfo metadata (type, totalCount, compressors, fileSize) matches the source dataset's acquisition type and spectrum count

## Limitations

- Compression ratios are dataset-dependent (Aird ranged from 5% to 40% of vendor volume across the three evaluation datasets), so single-dataset results do not generalize
- Conversion of very large datasets can hit memory limits; only AirdPro V6 resolves memory overflow issues during large-scale conversions
- The reported reductions are for the MRM data compression evaluated in MRMPro; ion-mobility gains (~20% higher compression rate) apply only to ion mobility raw files in V6

## Evidence

- [abstract] compression vs mzML reduction: "which reduced data volume by more than 60% and 80% respectively compared to vendor and mzML format"
- [readme] AirdPro purpose: [evidence span withheld — anchor status: external_doc]
- [readme] AirdPro mzML support: [evidence span withheld — anchor status: external_doc]
- [readme] AirdPro workflow: [evidence span withheld — anchor status: external_doc]
- [readme] co-located index requirement: [evidence span withheld — anchor status: external_doc]
