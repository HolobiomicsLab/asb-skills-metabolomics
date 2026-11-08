---
name: delta-encoding-of-retention-times
description: Use when converting vendor MRM chromatogram files to the Aird format
  and you need to shrink retention-time arrays so hundreds of chromatograms can render
  quickly on the web.
when_to_use_negative:
- The input is already in Aird format or another delta-encoded representation — re-applying
  delta coding to deltas is meaningless.
- The target format must be mzML or another standards-compliant interchange format,
  since this lossy quantization (1e-5 precision) is specific to the computation-oriented
  Aird format.
- Working with non-chromatogram arrays (e.g. m/z arrays in DDA/DIA spectra), which
  Aird handles with different combinable compressors such as Zlib-Diff-PforDelta (ZDPD).
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_3520
tools:
- name: MRMPro
  role: Web-based MRM data analysis platform whose efficient compression algorithm
    (targeted at fast rendering of hundreds of chromatograms on the web side) uses
    this RT delta-encoding scheme in the Aird format
  repo: github.com/CSi-Studio/MRMPro-Doc
- name: AirdPro
  role: GUI client (C#, based on ProteoWizard pwiz_bindings_cli.dll) that converts
    vendor MRM files to Aird format, with MRM/SRM acquisition support since V4.4/V5.1
  repo: https://github.com/CSi-Studio/AirdPro/releases/
- name: Aird-SDK
  role: Java/C#/Python SDK for reading the resulting .aird files during secondary
    development
  repo: https://github.com/CSi-Studio/Aird-SDK
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1186/s12859-024-05685-x
    title: 'MRMPro: a web-based tool to improve the speed of manual calibration for
      multiple reaction monitoring data analysis by mass spectrometry'
  source_content_hash: sha256:88668592629e36b9dbd04a5bb700a0756d4c89084124f740304423b8b0385eb7
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Aird-SDK
  - AirdPro
  - MRMPro
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# delta-encoding-of-retention-times

## Summary

Compresses MRM chromatogram retention-time arrays by quantizing each RT to an integer (1e-5 precision), storing deltas between adjacent RT integers, then applying FastPFor bit-level integer encoding followed by ZSTD compression as part of the Aird format used by MRMPro.

## When to use

Use when converting vendor MRM chromatogram files to the Aird format and you need to shrink retention-time arrays so hundreds of chromatograms can render quickly on the web.

## When NOT to use

- The input is already in Aird format or another delta-encoded representation — re-applying delta coding to deltas is meaningless.
- The target format must be mzML or another standards-compliant interchange format, since this lossy quantization (1e-5 precision) is specific to the computation-oriented Aird format.
- Working with non-chromatogram arrays (e.g. m/z arrays in DDA/DIA spectra), which Aird handles with different combinable compressors such as Zlib-Diff-PforDelta (ZDPD).

## Inputs

- Vendor MRM chromatogram file (e.g. Thermo Xcalibur or SCIEX OS raw output, Wiff2 supported by AirdPro)
- mzML-converted counterpart of the same MRM chromatogram (size-comparison baseline)
- Retention time array and intensity array of each MRM transition

## Outputs

- Aird data file (.aird) containing FastPFor-then-ZSTD-compressed RT delta arrays and separately encoded intensity arrays
- Aird JSON index file (.) stored alongside the .aird file with the same base name, including chromatogramIndex for MRM acquisition mode
- compression_report.csv with file-size ratios of Aird output versus vendor and mzML files

## How to apply

Load the vendor MRM chromatogram and its mzML-converted counterpart as size-comparison baselines. Quantize each retention time to the fifth decimal place (precision 1e-5) to obtain integers, then store the deltas between adjacent RT integers instead of the raw values — adjacent deltas are small, so they compress far better than monotonically increasing raw RT values. Pass the delta-coded integer arrays through the FastPFor bit-level integer compression codec, then compress the FastPFor-encoded blocks with ZSTD; intensity arrays are compressed separately with Aird's own intensity encoding method. Finally write the encoded chromatogram container (.aird plus a . index file with the same base name) to disk, and report size ratios of the encoded output versus vendor and mzML files in compression_report.csv. The MRMPro paper reports this compression scheme reduced data volume by more than 60% and 80% compared to vendor and mzML formats.

## Related tools

- **MRMPro** (Web-based MRM data analysis platform whose efficient compression algorithm (targeted at fast rendering of hundreds of chromatograms on the web side) uses this RT delta-encoding scheme in the Aird format) — github.com/CSi-Studio/MRMPro-Doc
- **AirdPro** (GUI client (C#, based on ProteoWizard pwiz_bindings_cli.dll) that converts vendor MRM files to Aird format, with MRM/SRM acquisition support since V4.4/V5.1) — https://github.com/CSi-Studio/AirdPro/releases/
- **Aird-SDK** (Java/C#/Python SDK for reading the resulting .aird files during secondary development) — https://github.com/CSi-Studio/Aird-SDK

## Examples

```
aird_sdk = AirdSDK; reader = AirdReader('sample.aird', 'sample.'); rt = reader.get_chromatogram_rt(transition_id)  # deltas + FastPFor + ZSTD decoded back to RT values at 1e-5 precision
```

## Evaluation signals

- compression_report.csv shows the Aird output is more than 60% smaller than the vendor file and more than 80% smaller than the mzML file, matching the paper's reported reduction.
- Decoding the .aird file with AirdSDK reproduces the original RT values within the 1e-5 quantization tolerance.
- The output directory contains a .aird file and a . index file with identical base names, as required by AirdScanUtil.
- The AirdInfo JSON declares the RT/intensity compressor strategies and chromatogramIndex populated for MRM acquisition mode.
- Spot-check that delta arrays decode back to monotonically non-decreasing RT integers, confirming correct adjacent-delta reconstruction.

## Limitations

- RT precision is capped at 1e-5 by the quantization step; applications needing finer RT resolution must not use this encoding.
- The scheme is specific to MRM chromatograms; other acquisition types (DIA/SWATH, DDA, PRM, PASEF) use different Aird compressor combinations.
- The MRMPro discussion notes the platform 'relies on classical algorithms' — the compression pipeline is classical (delta + FastPFor + ZSTD) rather than learned, and no more advanced analytical algorithms are incorporated yet.

## Evidence

- [abstract] abstract: "which reduced data volume by more than 60% and 80% respectively compared to vendor and mzML format"
- [intro] intro: "An efficient compression algorithm to meet the needs of fast rendering of hundreds of chromatograms on the web side"
- [readme] readme: [evidence span withheld — anchor status: external_doc]
