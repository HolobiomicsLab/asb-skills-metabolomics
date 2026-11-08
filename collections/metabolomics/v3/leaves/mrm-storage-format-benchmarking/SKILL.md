---
name: mrm-storage-format-benchmarking
description: Use when converting MRM datasets to Aird, vendor, and mzML formats and
  you need to measure per-dataset file-volume ratios to justify a compressed storage/computation-oriented
  format choice.
when_to_use_negative:
- The input data is already in a compressed/indexed format (e.g. existing .aird +
  . pairs) and only needs reading via AirdSDK, not re-benchmarking
- The task is quantification accuracy or peak-picking evaluation (MRMPro vs Skyline),
  not storage-volume measurement
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0219
tools:
- name: AirdPro
  role: GUI client (C#, built on ProteoWizard pwiz_bindings_cli.dll) used to convert
    vendor raw files to the Aird format with combinable m/z, intensity, and mobility
    compressors
  repo: https://github.com/CSi-Studio/AirdPro/releases/
- name: MSConvert
  role: ProteoWizard converter used to produce the 32-bit mzML baseline files for
    volume comparison
- name: Aird-SDK
  role: Java/C#/Python SDK for reading the resulting .aird/. file pairs during downstream
    verification
  repo: https://github.com/CSi-Studio/Aird-SDK
- name: Skyline
  role: Source of the Skyline test MRM dataset used as one of the three benchmark
    evaluation datasets
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1186/s12859-024-05685-x
    title: 'MRMPro: a web-based tool to improve the speed of manual calibration for
      multiple reaction monitoring data analysis by mass spectrometry'
  source_content_hash: sha256:11d371b007c644f3849541bf5819768241b3feb5747ac0a5d2ac3e9d2d2bc4b6
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
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0219
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# mrm-storage-format-benchmarking

## Summary

Benchmark the on-disk storage volume of the Aird format against vendor raw files and 32-bit mzML for MRM mass spectrometry datasets, quantifying the compression gains (>60% vs vendor, >80% vs mzML) that enable fast web-based chromatogram rendering in MRMPro.

## When to use

Use when converting MRM datasets to Aird, vendor, and mzML formats and you need to measure per-dataset file-volume ratios to justify a compressed storage/computation-oriented format choice.

## When NOT to use

- The input data is already in a compressed/indexed format (e.g. existing .aird + . pairs) and only needs reading via AirdSDK, not re-benchmarking
- The task is quantification accuracy or peak-picking evaluation (MRMPro vs Skyline), not storage-volume measurement

## Inputs

- Vendor raw MRM files from PRIDE dataset PXD009543
- Vendor raw MRM files from PRIDE dataset PXD031038
- Skyline example/test MRM dataset
- MSConvert (ProteoWizard) for 32-bit mzML conversion
- AirdPro GUI client for vendor-to-Aird conversion

## Outputs

- storage_volume_comparison.csv with per-dataset volumes in vendor, 32-bit mzML, and Aird formats
- Per-dataset Aird-to-vendor and Aird-to-mzML volume ratios

## How to apply

Download three public MRM raw datasets (PXD009543 from PRIDE, the Skyline test set, and PXD031038 from PRIDE). Convert each dataset's vendor raw files to 32-bit mzML with MSConvert and to Aird with AirdPro (Aird pairs a .aird data file with a same-named . index file). Record total on-disk volume per dataset for all three formats, then compute the Aird ratio relative to vendor and relative to mzML. The expected per-dataset ratios are PXD009543: 40% of vendor / 18% of mzML; Skyline: 5% / 20%; PXD031038: 15% / 14%, consistent with the paper's >60% and >80% volume reductions versus vendor and mzML. Rationale: Aird's combinable compressors (Zlib-Diff-PforDelta, ZDVB, Stack-ZDPD) achieve controllable precision and fast decoding, which is what allows MRMPro to render hundreds of chromatograms quickly in the browser.

## Related tools

- **AirdPro** (GUI client (C#, built on ProteoWizard pwiz_bindings_cli.dll) used to convert vendor raw files to the Aird format with combinable m/z, intensity, and mobility compressors) — https://github.com/CSi-Studio/AirdPro/releases/
- **MSConvert** (ProteoWizard converter used to produce the 32-bit mzML baseline files for volume comparison)
- **Aird-SDK** (Java/C#/Python SDK for reading the resulting .aird/. file pairs during downstream verification) — https://github.com/CSi-Studio/Aird-SDK
- **Skyline** (Source of the Skyline test MRM dataset used as one of the three benchmark evaluation datasets)

## Examples

```
msconvert PXD009543_raw --mzML --32; # then convert same vendor files to Aird via AirdPro GUI, and compute ratios: du -sb vendor mzml aird > storage_volume_comparison.csv
```

## Evaluation signals

- PXD009543 Aird volume is ~40% of vendor and ~18% of mzML volume
- Skyline test set Aird volume is ~5% of vendor and ~20% of mzML volume
- PXD031038 Aird volume is ~15% of vendor and ~14% of mzML volume
- Measured reductions meet the paper's reported thresholds: >60% smaller than vendor and >80% smaller than mzML
- Each converted Aird dataset yields paired .aird and . index files with identical base names in the same directory

## Limitations

- Compression ratios vary substantially per dataset (5%–40% of vendor volume), so results on new datasets may deviate from the reported figures
- AirdPro's compression strategy relies on classical/kernel algorithms (ZDPD, ZDVB, Stack-ZDPD); AirdPro V6 notes '20% higher compression rate for ion mobility raw files', implying version-dependent results
- The benchmark measures file size only; search/rendering speed benefits (e.g. Aird-Slice 50–500x speedups) are separate claims not validated by volume measurements

## Evidence

- [abstract] MRM data compression algorithm reduced data volume by more than 60% and 80% compared to vendor and mzML format: [evidence span withheld — anchor status: not_found]
- [intro] Objective: efficient compression algorithm: "An efficient compression algorithm to meet the needs of fast rendering of hundreds of chromatograms on the web side"
- [readme] AirdPro is a GUI client for conversion from vendor files to Aird files: [evidence span withheld — anchor status: external_doc]
- [readme] Aird file pairing rule: [evidence span withheld — anchor status: external_doc]
- [readme] Compressors available in AirdPro V3: [evidence span withheld — anchor status: external_doc]
