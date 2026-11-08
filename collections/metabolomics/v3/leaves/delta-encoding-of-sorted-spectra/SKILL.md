---
name: delta-encoding-of-sorted-spectra
description: Use when a monotonically ordered numeric array (e.g. a centroided m/z
  list) contains large integers whose adjacent values differ by small, near-constant
  amounts, and you need a smaller compressed footprint than generic Zlib-only compression.
when_to_use_negative:
- The array is unordered or has large, irregular adjacent differences (e.g. intensity
  arrays, which Aird instead compresses with options like the log10 compressor or
  ignores at zero-intensity points)
- Bit-exact lossless reproduction of the original vendor floating-point m/z values
  is required, since Aird stores m/z as integers at a user-chosen controllable precision
- Working with acquisition modes the format does not yet support (the paper states
  Aird supports only DDA, DIA/SWATH, and PRM; ScanningSWATH and Ion Mobility formats
  were under development)
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: Aird
  role: Mass spectrometry data format whose ZDPD compressor implements the float-to-integer
    quantization plus delta coding chain for m/z arrays
  repo: https://github.com/CSi-Studio/Aird-SDK
- name: AirdPro
  role: C# GUI client that converts vendor files to Aird, applying compression parameters
    and the user's targeted precision to m/z and intensity arrays
  repo: https://github.com/CSi-Studio/AirdPro/releases/
- name: Aird-SDK
  role: Java SDK for reading/writing Aird files and its ZDPD-compressed m/z blocks
    in downstream tools
  repo: https://github.com/CSi-Studio/Aird-SDK
- name: Zlib (DotNetZip)
  role: Final general-purpose stage of the ZDPD chain; Aird uses the DotNetZip Zlib
    library for the outer compression layer
  repo: http://dotnetzip.codeplex.com
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1186/s12859-021-04490-0
    title: 'Aird: a computation-oriented mass spectrometry data format enables a higher
      compression ratio and less decoding time'
  source_content_hash: sha256:42cbd489f1cd1314f62023d8655a1017392b1d96bb77474d32005030f2d0b8fd
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/CSi-Studio/Aird-SDK
  tools:
  - Aird
  - Aird-SDK
  - AirdPro
  - Zlib
  - Zlib (DotNetZip)
  - Zlib-Diff-PforDelta (ZDPD)
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9375
n_claims: 16
---

# delta-encoding-of-sorted-spectra

## Summary

Compress ordered, numeric spectra arrays (m/z) by quantizing floats to integers at a chosen decimal precision and encoding only the small deltas between adjacent sorted integers, as in Aird's ZDPD (Zlib-Diff-PforDelta) compressor.

## When to use

Use when a monotonically ordered numeric array (e.g. a centroided m/z list) contains large integers whose adjacent values differ by small, near-constant amounts, and you need a smaller compressed footprint than generic Zlib-only compression.

## When NOT to use

- The array is unordered or has large, irregular adjacent differences (e.g. intensity arrays, which Aird instead compresses with options like the log10 compressor or ignores at zero-intensity points)
- Bit-exact lossless reproduction of the original vendor floating-point m/z values is required, since Aird stores m/z as integers at a user-chosen controllable precision
- Working with acquisition modes the format does not yet support (the paper states Aird supports only DDA, DIA/SWATH, and PRM; ScanningSWATH and Ion Mobility formats were under development)

## Inputs

- Sorted floating-point m/z array from a mass spectrum (e.g. from a vendor file, mzML, or a ProteomeXchange/MetaboLights dataset such as PXD021390 or MTBLS2119)
- Target precision in decimal places (3 dp for 10 ppm accuracy, 5 dp for 0.1 ppm accuracy)

## Outputs

- Integer array of quantized m/z values
- Delta array of adjacent integer differences
- FastPfor-encoded (optionally Zlib-wrapped) compressed byte stream implementing ZDPD
- Compression ratio and decoding-time comparison vs Zlib-only baseline

## How to apply

First determine the required precision: the Aird paper maps 10 ppm mass accuracy to 3 decimal places (dp) and 0.1 ppm to 5 dp, then converts the floating-point m/z array to an integer array by scaling to that precision. Sort the array if not already ordered and compute the deltas between adjacent integers, since compressing these relatively small deltas is more effective than compressing the large integers themselves. Pass the delta array to the FastPfor library, which performs differential (PForDelta) coding of small integer arrays using SIMD technology. Optionally wrap the FastPfor output in Zlib (DotNetZip at compression level 6, as used by Aird) to complete the ZDPD chain. Validate by measuring the byte size of the delta-coded output against Zlib-only compression of the same integer array: ZDPD should yield about 55% lower m/z data size and about 3x faster decoding on average.

## Related tools

- **Aird** (Mass spectrometry data format whose ZDPD compressor implements the float-to-integer quantization plus delta coding chain for m/z arrays) — https://github.com/CSi-Studio/Aird-SDK
- **AirdPro** (C# GUI client that converts vendor files to Aird, applying compression parameters and the user's targeted precision to m/z and intensity arrays) — https://github.com/CSi-Studio/AirdPro/releases/
- **Aird-SDK** (Java SDK for reading/writing Aird files and its ZDPD-compressed m/z blocks in downstream tools) — https://github.com/CSi-Studio/Aird-SDK
- **Zlib (DotNetZip)** (Final general-purpose stage of the ZDPD chain; Aird uses the DotNetZip Zlib library for the outer compression layer) — http://dotnetzip.codeplex.com

## Examples

```
import numpy as np; ints = np.round(mz * 1000).astype(np.int64); deltas = np.diff(ints, prepend=ints[0])
```

## Evaluation signals

- The combined ZDPD output (FastPfor+Zlib) must be smaller than Zlib-only compression of the same integer m/z array; the paper reports about 55% lower m/z data size on average (49% reduction cited in the discussion)
- Decoding time of the delta-coded array should be substantially faster than Zlib decoding of the raw integers; Aird takes about 33% of Zlib's decoding time (~3x speedup) via SIMD-accelerated FastPfor
- Round-trip check: dequantizing the decoded integer array by the chosen scaling factor (3 dp or 5 dp) must reproduce m/z values within the target mass accuracy (10 ppm or 0.1 ppm)
- The quantized integer array must be monotonically non-decreasing after sorting, so all deltas are non-negative and small relative to the absolute m/z values

## Limitations

- ZDPD results are lossy at the chosen precision: m/z floats are stored as integers scaled to 3 dp (10 ppm) or 5 dp (0.1 ppm), so ultra-high-accuracy use cases need higher precision settings
- Delta coding is only effective because consecutive sorted m/z values differ by small, near-fixed amounts; data violating that regularity will compress poorly
- The paper provides no benchmark scripts to reproduce the 49% size reduction or 3x decoding speed claims, and no discussion of when ZDPD may underperform
- Newer kernels supersede ZDPD: Stack-ZDPD improved compression by 5-20% and ZDVB by 10-20% with faster decoding (per AirdPro release notes)

## Evidence

- [intro] After determining the required precision, Aird converts the floating-point m/z array to an integer array.: "After determining the required precision, Aird converts the floating-point m/z array to an integer array."
- [intro] it is an effective way to compress relatively small deltas of adjacent integers instead of the large integers themselves: "it is an effective way to compress relatively small deltas of adjacent integers instead of the large integers themselves"
- [intro] Aird uses the FastPfor library for differential coding of integer arrays [8]. The FastPfor library uses the SIMD technology: "Aird uses the FastPfor library for differential coding of integer arrays [8]. The FastPfor library uses the SIMD technology"
- [abstract] Compared with Zlib only, m/z data size is about 55% lower in Aird average.: "Compared with Zlib only, m/z data size is about 55% lower in Aird average."
- [methods] Zlib library is DotNetZip from http://dotnetzip.codeplex.com: "Zlib library is DotNetZip from http://dotnetzip.codeplex.com"
- [readme] A new compression algorithm Stack-ZDPD is released, which has 5%-20% compression performance improvement compared with ZDPD kernel: "A new compression algorithm Stack-ZDPD is released, which has 5%-20% compression performance improvement compared with ZDPD kernel"
