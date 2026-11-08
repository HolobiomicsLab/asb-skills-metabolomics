---
name: simd-accelerated-integer-compression
description: Use when compressing ordered, near-uniformly spaced numeric arrays (e.g.
  m/z values from MS spectra in mzML/vendor-converted data) where controllable precision
  (e.g. 3 dp for 10 ppm, 5 dp for 0.1 ppm) is acceptable and both size and decode
  speed matter.
when_to_use_negative:
- The data are intensity arrays or other slowly varying values where the small-delta
  integer assumption does not hold — Aird applies distinct compressors (e.g. Log10
  Compressor) to intensity data
- Lossless m/z values are required with no quantization — ZDPD depends on converting
  the floating-point m/z array to an integer array after determining the required
  precision
- Random access with flexible indexing is the primary goal rather than compression
  ratio — a general-purpose format like mzML converted with MSConvert may be more
  appropriate
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: FastPfor
  role: SIMD-accelerated library used for differential coding of the integer delta
    array in the ZDPD chain
- name: Zlib (DotNetZip)
  role: Final entropy-compression layer applied to the FastPfor-encoded output at
    level 6
  repo: http://dotnetzip.codeplex.com
- name: Aird-SDK
  role: Java SDK implementing Aird/ZDPD encoding and decoding for m/z arrays
  repo: https://github.com/CSi-Studio/Aird-SDK
- name: AirdPro
  role: C# GUI client that reads vendor files via ProteoWizard SDK and compresses
    m/z/intensity arrays to Aird according to targeted precision
  repo: https://github.com/CSi-Studio/AirdPro/releases/
- name: MSConvert
  role: Converts raw vendor data to mzML/mgf/mz5 for the baseline comparison against
    ZDPD-compressed Aird
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1186/s12859-021-04490-0
    title: 'Aird: a computation-oriented mass spectrometry data format enables a higher
      compression ratio and less decoding time'
  source_content_hash: sha256:cff825a6db712476e9433c524d48f44f38eff8f86bb1e8aeca932f3748986e02
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Aird-SDK
  - AirdPro
  - FastPfor
  - MSConvert
  - Zlib
  - Zlib (DotNetZip)
  - Zlib-Diff-PforDelta (ZDPD)
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8823529411764706
n_claims: 17
---

# simd-accelerated-integer-compression

## Summary

Reproduces Aird's Zlib-Diff-PforDelta (ZDPD) compressor: quantize a sorted floating-point m/z array to integers at a chosen decimal precision, delta-code adjacent integers, compress with the SIMD-accelerated FastPfor library, then wrap the output in Zlib to achieve ~55% smaller m/z data and ~3x faster decoding than Zlib alone.

## When to use

Use when compressing ordered, near-uniformly spaced numeric arrays (e.g. m/z values from MS spectra in mzML/vendor-converted data) where controllable precision (e.g. 3 dp for 10 ppm, 5 dp for 0.1 ppm) is acceptable and both size and decode speed matter.

## When NOT to use

- The data are intensity arrays or other slowly varying values where the small-delta integer assumption does not hold — Aird applies distinct compressors (e.g. Log10 Compressor) to intensity data
- Lossless m/z values are required with no quantization — ZDPD depends on converting the floating-point m/z array to an integer array after determining the required precision
- Random access with flexible indexing is the primary goal rather than compression ratio — a general-purpose format like mzML converted with MSConvert may be more appropriate

## Inputs

- Sorted floating-point m/z array from an MS spectrum (DDA/DIA/SWATH/PRM data, e.g. from ProteomeXchange or MetaboLights accessions such as PXD021390, PXD018139, MTBLS2119)
- Target precision in decimal places (3 dp for 10 ppm accuracy; 5 dp for 0.1 ppm)

## Outputs

- ZDPD-compressed byte block (FastPfor-encoded delta array wrapped in Zlib level 6) for the m/z array
- Comparison metrics: compressed byte size and decoding time of ZDPD vs Zlib-only on the same integer array

## How to apply

First quantize the sorted floating-point m/z array into an integer array by scaling to the target decimal precision (Aird guidance: 3 decimal places corresponds to 10 ppm accuracy, 5 dp to 0.1 ppm). Compute the deltas between adjacent integers of the sorted array, exploiting that consecutive m/z values differ by small, near-fixed amounts rather than compressing the large integers themselves. Apply the FastPfor library to the delta array for differential coding, leveraging its SIMD-accelerated encoding. Finally compress the FastPfor-encoded bytes with Zlib (DotNetZip, compression level 6 as used by Aird). Validate by measuring the ZDPD output byte size against Zlib-only compression of the same integer array: the combined pipeline must be smaller, consistent with the paper's ~55% lower m/z data size and ~33% of Zlib's decoding time.

## Related tools

- **FastPfor** (SIMD-accelerated library used for differential coding of the integer delta array in the ZDPD chain)
- **Zlib (DotNetZip)** (Final entropy-compression layer applied to the FastPfor-encoded output at level 6) — http://dotnetzip.codeplex.com
- **Aird-SDK** (Java SDK implementing Aird/ZDPD encoding and decoding for m/z arrays) — https://github.com/CSi-Studio/Aird-SDK
- **AirdPro** (C# GUI client that reads vendor files via ProteoWizard SDK and compresses m/z/intensity arrays to Aird according to targeted precision) — https://github.com/CSi-Studio/AirdPro/releases/
- **MSConvert** (Converts raw vendor data to mzML/mgf/mz5 for the baseline comparison against ZDPD-compressed Aird)

## Examples

```
import org.aird.*; int[] ints = AirdUtil.quantize(mzArray, 3); int[] deltas = AirdUtil.diff(ints); byte[] zdpd = Zlib.compress(FastPfor.encode(deltas), 6);
```

## Evaluation signals

- ZDPD output (FastPfor+Zlib) must have a smaller byte size than Zlib-only compression of the same integer m/z array — paper reports ~55% lower m/z data size on average (49% reported for ZDPD in discussion)
- Decoding time of ZDPD should be about 33% of Zlib's decoding time (roughly 3x faster) on the same m/z arrays
- Round-trip check: decoding the ZDPD block and rescaling by the precision factor must reproduce the quantized integer array exactly, and within the chosen ppm tolerance (10 ppm at 3 dp, 0.1 ppm at 5 dp) of the original floats
- Delta array values must be small relative to the absolute integers (near-fixed adjacent differences) — if deltas are large or erratic, the precision choice or sort order is wrong
- Aird index (.json) and data (.aird) files must be stored in the same directory with the same file name per the format convention

## Limitations

- No benchmark or benchmark scripts are provided to reproduce the 49% size reduction or the 3x faster decoding claims, so exact ratios may not be reproducible
- ZDPD's advantage depends on small adjacent deltas; later AirdPro releases added ZDVB (10-20% better compression, faster decoding than ZDPD) and Stack-ZDPD (5-20% improvement), indicating ZDPD is not the strongest kernel in the current toolchain
- Aird's Dynamic-Decider for Best ComboComp exists because no single compressor combination is always best; ZDPD may underperform on some inputs
- The 55%/33% figures are averages over seven datasets from SCIEX, Thermo, and Agilent instruments; per-dataset results may differ

## Evidence

- [intro] After determining the required precision, Aird converts the floating-point m/z array to an integer array.: "After determining the required precision, Aird converts the floating-point m/z array to an integer array."
- [intro] FastPfor differential coding with SIMD: "Aird uses the FastPfor library for differential coding of integer arrays [8]. The FastPfor library uses the SIMD technology"
- [abstract] 55% size reduction finding: "Compared with Zlib only, m/z data size is about 55% lower in Aird average."
- [abstract] Decoding speed finding: "Aird merely takes 33% decoding time compared with Zlib."
- [methods] Zlib library identity: "Zlib library is DotNetZip from http://dotnetzip.codeplex.com"
- [readme] README compression claim: "Aird provides a novel compressor called
**ComboComp** for _m/z_ data compression,which makes up an amazing compression rate. Compared with Zlib, m/z data is
about
65% lower in the Aird on average."
