---
name: file-volume-measurement-and-ratio-computation
description: Use when evaluating or reporting the storage footprint of MS data formats
  — e.g., MRM raw vendor files must be compared against converted 32-bit mzML and
  Aird volumes, or compression ratios must be validated against reported reductions
  of >60% (vs vendor) and >80% (vs mzML).
when_to_use_negative:
- Input data are already in a compressed or analysis-ready format (e.g., feature tables
  or peak lists) rather than vendor raw files — no meaningful vendor-vs-converted
  volume comparison is possible.
- The goal is quantification accuracy or manual inspection benchmarking (MRMPro vs
  Skyline) rather than storage footprint measurement.
- Datasets use ion-mobility or non-MRM acquisition modes where the reported MRM compression
  ratios (40%/5%/15% of vendor) do not apply.
edam_topics:
- http://edamontology.org/topic_0121
tools:
- name: MSConvert
  role: ProteoWizard converter used to produce 32-bit mzML files from vendor raw files
    for the mzML-volume baseline
- name: AirdPro
  role: C# GUI client (built on pwiz_bindings_cli.dll from ProteoWizard) used to convert
    the same vendor raw files into Aird format (.aird data file + . index file)
  repo: https://github.com/CSi-Studio/AirdPro/releases/
- name: MRMPro
  role: Web-based MRM analysis platform whose storage layer motivates the Aird compression
    benchmark
- name: Skyline
  role: Source of the example MRM test dataset used as one of the three evaluation
    datasets
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1186/s12859-024-05685-x
    title: 'MRMPro: a web-based tool to improve the speed of manual calibration for
      multiple reaction monitoring data analysis by mass spectrometry'
  source_content_hash: sha256:0c268d5adb6795608ed801ed903c4dcb341d7257daec1a37c82c3aabc0ecfe4a
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - AirdPro
  - MRMPro
  - MSConvert
  - Skyline
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# file-volume-measurement-and-ratio-computation

## Summary

A benchmarking procedure that converts public MRM mass-spectrometry datasets to three storage formats (vendor raw, 32-bit mzML via MSConvert, Aird via AirdPro), records total on-disk volume per format, and computes Aird-vs-vendor and Aird-vs-mzML storage ratios to quantify the compression benefit of the Aird format.

## When to use

Use when evaluating or reporting the storage footprint of MS data formats — e.g., MRM raw vendor files must be compared against converted 32-bit mzML and Aird volumes, or compression ratios must be validated against reported reductions of >60% (vs vendor) and >80% (vs mzML).

## When NOT to use

- Input data are already in a compressed or analysis-ready format (e.g., feature tables or peak lists) rather than vendor raw files — no meaningful vendor-vs-converted volume comparison is possible.
- The goal is quantification accuracy or manual inspection benchmarking (MRMPro vs Skyline) rather than storage footprint measurement.
- Datasets use ion-mobility or non-MRM acquisition modes where the reported MRM compression ratios (40%/5%/15% of vendor) do not apply.

## Inputs

- Vendor raw MRM dataset PXD009543 (PRIDE)
- Vendor raw MRM dataset PXD031038 (PRIDE)
- Skyline example MRM test dataset
- MSConvert tool for 32-bit mzML conversion
- AirdPro GUI client for vendor-to-Aird conversion

## Outputs

- storage_volume_comparison.csv with per-dataset total volumes in vendor, 32-bit mzML, and Aird formats
- Per-dataset Aird-vs-vendor storage ratios (expected: 40%, 5%, 15%)
- Per-dataset Aird-vs-mzML storage ratios (expected: 18%, 20%, 14%)

## How to apply

Download the three public MRM evaluation datasets (PXD009543 and PXD031038 from PRIDE, plus the Skyline example test set). Convert each dataset's vendor raw files to 32-bit mzML with MSConvert, and convert the same vendor files to Aird format with AirdPro, ensuring the .aird data file and . index file are counted together since Aird stores them as pairs with identical filenames in the same directory. Sum total on-disk volume per dataset per format, then compute the Aird storage ratio relative to vendor volume and relative to 32-bit mzML volume. Rationale: Aird uses combinable compressors (ZDPD/ZDVB, Brotli, Snappy, Zstd, Zlib, Variable Byte, Binary Packing) with a Dynamic-Decider for the best combo, which the MRMPro paper reports as reducing data volume by more than 60% and 80% respectively compared to vendor and mzML formats. The measured ratios should approximate the reported per-dataset values: PXD009543 40% of vendor / 18% of mzML, Skyline 5% / 20%, PXD031038 15% / 14%.

## Related tools

- **MSConvert** (ProteoWizard converter used to produce 32-bit mzML files from vendor raw files for the mzML-volume baseline)
- **AirdPro** (C# GUI client (built on pwiz_bindings_cli.dll from ProteoWizard) used to convert the same vendor raw files into Aird format (.aird data file + . index file)) — https://github.com/CSi-Studio/AirdPro/releases/
- **MRMPro** (Web-based MRM analysis platform whose storage layer motivates the Aird compression benchmark)
- **Skyline** (Source of the example MRM test dataset used as one of the three evaluation datasets)

## Examples

```
msconvert --mzML --32 PXD009543/*.raw -o PXD009543_mzML32 && du -sb PXD009543_vendor PXD009543_mzML32 PXD009543_aird
```

## Evaluation signals

- Aird-vs-vendor ratios approximate 40% (PXD009543), 5% (Skyline test set), and 15% (PXD031038).
- Aird-vs-mzML ratios approximate 18% (PXD009543), 20% (Skyline test set), and 14% (PXD031038).
- Aggregate reductions meet the paper's claim of >60% versus vendor and >80% versus 32-bit mzML.
- storage_volume_comparison.csv contains all three datasets with volumes recorded in all three formats (vendor, 32-bit mzML, Aird).
- Aird volume accounting includes both the .aird data file and the paired . index file, which share the same filename in the same directory.

## Limitations

- Ratios are dataset-dependent: PXD009543 compresses to 40% of vendor but the Skyline test set reaches 5%, so a single dataset will not reproduce the aggregate claim on its own.
- 32-bit mzML is an uncompressed-format baseline; conclusions do not transfer to other mzML bit depths or to alternative compressors.
- Aird requires paired index/data files (. + .aird) stored together; partial or split file accounting will bias the measured volume.
- AirdPro's compression performance varies by version (e.g., V6 reports 20% higher compression for ion mobility raw files), so tool version must be reported alongside ratios.

## Evidence

- [abstract] MRM data compression algorithm reduced data volume by more than 60% and 80% compared to vendor and mzML format: "which reduced data volume by more than 60% and 80% respectively compared to vendor and mzML format"
- [readme] AirdPro converts vendor files to Aird format: [evidence span withheld — anchor status: external_doc]
- [readme] Aird index and data file pairing requirement: [evidence span withheld — anchor status: external_doc]
- [readme] Aird format compression properties: [evidence span withheld — anchor status: external_doc]
