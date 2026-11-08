---
name: mz-integer-quantization
description: Use when preparing m/z array data for lossless integer-based compression
  (e.g. ZDPD, ZDVB, Stack-ZDPD) or when a controlled-precision integer representation
  of m/z values must be derived from floating-point spectra.
when_to_use_negative:
- When full floating-point m/z fidelity is required beyond the quantization precision
  (e.g. sub-0.1 ppm calibration workflows)
- When compressing intensity arrays, which Aird handles with separate compressors
  (e.g. log10 compressor) rather than m/z integer quantization
- For formats that already store m/z as integers or whose readers do not support Aird's
  precision-controllable integer scheme
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: Aird-SDK
  role: Java/C#/Python SDK implementing the m/z integer storage and ZDPD encoding
    chain; used to read back quantized integer arrays
  repo: https://github.com/CSi-Studio/Aird-SDK
- name: AirdPro
  role: C# GUI client that converts vendor files and performs the precision-targeted
    compression of m/z arrays
  repo: https://github.com/CSi-Studio/AirdPro/releases/
- name: Aird
  role: File format that stores the m/z array as integers at controllable precision
  repo: https://github.com/CSi-Studio/Aird-SDK
provenance:
  source_task_ids:
  - task_004
  - task_002
  source_papers:
  - doi: 10.1186/s12859-021-04490-0
    title: 'Aird: a computation-oriented mass spectrometry data format enables a higher
      compression ratio and less decoding time'
  source_content_hash: sha256:bb07d30c4662aa538dcc53d39c92800fbbc1b5c7104a1b3cdaead772133145df
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/CSi-Studio/Aird-SDK
  tools:
  - Aird
  - Aird-SDK
  - AirdPro
  - Zlib
  - Zlib-Diff-PforDelta (ZDPD)
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8095238095238095
n_claims: 21
---

# mz-integer-quantization

## Summary

Quantizes floating-point m/z arrays into integer arrays at a target decimal precision (3 dp for 10 ppm, 5 dp for 0.1 ppm), the first stage of Aird's ZDPD compression chain that enables delta coding of adjacent small integers.

## When to use

Use when preparing m/z array data for lossless integer-based compression (e.g. ZDPD, ZDVB, Stack-ZDPD) or when a controlled-precision integer representation of m/z values must be derived from floating-point spectra.

## When NOT to use

- When full floating-point m/z fidelity is required beyond the quantization precision (e.g. sub-0.1 ppm calibration workflows)
- When compressing intensity arrays, which Aird handles with separate compressors (e.g. log10 compressor) rather than m/z integer quantization
- For formats that already store m/z as integers or whose readers do not support Aird's precision-controllable integer scheme

## Inputs

- Sorted floating-point m/z array from an MS spectrum (DDA, DIA/SWATH, or PRM acquisition)
- Target mass accuracy in ppm (maps to decimal places: 10 ppm → 3 dp, 0.1 ppm → 5 dp)
- Vendor raw file or mzML/mzXML input convertible via AirdPro/ProteoWizard

## Outputs

- Integer array representation of the m/z values at the chosen decimal precision
- Metadata recording the precision/decimal-places used for reconstruction of float values on read

## How to apply

Take the sorted floating-point m/z array from a mass spectrometry dataset (vendor-converted via AirdPro or from a ProteomeXchange/MetaboLights accession) and scale it to integers at the precision needed for the analysis: 10 ppm mass accuracy corresponds to 3 decimal places, 0.1 ppm to 5 decimal places. Aird chooses the integer type as the storage type for the m/z array precisely because, after quantization, consecutive values differ by small, near-fixed deltas — making it far more effective to compress deltas of adjacent integers than the large integers themselves. The quantized integer array then feeds downstream differential coding with the FastPfor library (SIMD-accelerated) followed by Zlib wrapping (DotNetZip, compression level 6) in the ZDPD pipeline. The quantization step itself is lossy at the chosen precision, so the decimal-places setting is the key decision point balancing file size against mass accuracy.

## Related tools

- **Aird-SDK** (Java/C#/Python SDK implementing the m/z integer storage and ZDPD encoding chain; used to read back quantized integer arrays) — https://github.com/CSi-Studio/Aird-SDK
- **AirdPro** (C# GUI client that converts vendor files and performs the precision-targeted compression of m/z arrays) — https://github.com/CSi-Studio/AirdPro/releases/
- **Aird** (File format that stores the m/z array as integers at controllable precision) — https://github.com/CSi-Studio/Aird-SDK

## Examples

```
AirdPro conversion with precision setting (GUI): vendor .raw/.wiff → .aird + .json, choosing 3 decimal places (10 ppm) or 5 decimal places (0.1 ppm) in the Compression Config Panel; programmatic read-back via `pip install AirdSDK`
```

## Evaluation signals

- Round-trip check: multiplying quantized integers by the precision factor and comparing against the original floats must not exceed the error implied by the chosen decimal places (3 dp ≈ 10 ppm, 5 dp ≈ 0.1 ppm)
- Adjacent-delta distribution: deltas of the quantized sorted integer array should be small, near-fixed values — if deltas are large and irregular, the precision choice or sorting is wrong
- Downstream ZDPD output (FastPfor+Zlib) on the quantized integers must be smaller than Zlib-only compression of the same array (paper reports ~55% lower size on average)
- Aird JSON index file (.json) must record the compression parameters and targeted precision consistent with the encoded integer arrays

## Limitations

- Quantization is lossy at the chosen precision; the article frames precision as 'controllable' but does not evaluate error propagation in downstream identifications
- Aird currently supports only DDA, DIA/SWATH, and PRM acquisition modes; ScanningSWATH and Ion Mobility formats were under development
- No benchmark scripts are provided to independently reproduce the reported size-reduction and decoding-speed claims

## Evidence

- [intro] After determining the required precision, Aird converts the floating-point m/z array to an integer array.: "After determining the required precision, Aird converts the floating-point m/z array to an integer array."
- [intro] it is an effective way to compress relatively small deltas of adjacent integers instead of the large integers themselves: "it is an effective way to compress relatively small deltas of adjacent integers instead of the large integers themselves"
- [methods] Aird chooses the integer type as the storage type for the m/z array: "Aird chooses the integer type as the storage type for the m/z array"
- [abstract] Compared with Zlib only, m/z data size is about 55% lower in Aird average.: "Compared with Zlib only, m/z data size is about 55% lower in Aird average."
- [readme] an opensource and computation-oriented format with controllable precision, flexible indexing strategies, and high compression rate: "an opensource and computation-oriented format with controllable precision, flexible indexing strategies, and high compression rate"
