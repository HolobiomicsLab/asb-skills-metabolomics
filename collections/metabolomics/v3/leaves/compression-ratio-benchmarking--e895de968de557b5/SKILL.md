---
name: compression-ratio-benchmarking
description: Use when you need to quantify whether a specialized compressor (ZDPD/Stack-ZDPD
  vs Zlib, or Brotli/Snappy/Zstd alternatives) actually beats a general-purpose baseline
  on your MS data arrays, in bytes and decode speed.
when_to_use_negative:
- Intensity arrays or ion mobility arrays when the claim under test is specifically
  about m/z compression — the 55%/33% figures apply to m/z data only
- Data where exact lossless m/z values are required and no controllable-precision
  quantization (3–5 decimal places) is acceptable
- Acquisition modes Aird does not yet support — at the time of the paper only DDA,
  DIA/SWATH, and PRM were supported (ScanningSWATH and Ion Mobility were under development)
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3520
tools:
- name: Aird
  role: Format under benchmark; its ZDPD compressor is the candidate being measured
    against Zlib-only
  repo: https://github.com/CSi-Studio/Aird-SDK
- name: Zlib-Diff-PforDelta (ZDPD)
  role: 'Candidate compressor: integer quantization + delta coding via FastPfor +
    Zlib level 6'
  repo: https://github.com/CSi-Studio/Aird-SDK
- name: Zlib
  role: General-purpose baseline compressor (DotNetZip) against which ZDPD size and
    decoding time are compared
- name: AirdPro
  role: GUI client that converts vendor files and applies the compression parameters
    and targeted precision used in the benchmark
  repo: https://github.com/CSi-Studio/AirdPro/releases/
- name: MSConvert
  role: Converts raw vendor data into mzML, mgf, and mz5 comparison formats
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1186/s12859-021-04490-0
    title: 'Aird: a computation-oriented mass spectrometry data format enables a higher
      compression ratio and less decoding time'
  source_content_hash: sha256:634a3485177cb456d9cd38f7e5a242439202ac2cbc59c7d9ce2fa393be28f6c9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/CSi-Studio/Aird-SDK
  tools:
  - Aird
  - AirdPro
  - MSConvert
  - Zlib
  - Zlib-Diff-PforDelta (ZDPD)
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8666666666666667
n_claims: 15
---

# compression-ratio-benchmarking

## Summary

Benchmarks domain-specific lossy/integer-transform compression (e.g., Aird's Zlib-Diff-PforDelta, ZDPD) against a general-purpose baseline (Zlib-only) by comparing compressed byte size and decoding time on real mass spectrometry m/z and intensity arrays.

## When to use

Use when you need to quantify whether a specialized compressor (ZDPD/Stack-ZDPD vs Zlib, or Brotli/Snappy/Zstd alternatives) actually beats a general-purpose baseline on your MS data arrays, in bytes and decode speed.

## When NOT to use

- Intensity arrays or ion mobility arrays when the claim under test is specifically about m/z compression — the 55%/33% figures apply to m/z data only
- Data where exact lossless m/z values are required and no controllable-precision quantization (3–5 decimal places) is acceptable
- Acquisition modes Aird does not yet support — at the time of the paper only DDA, DIA/SWATH, and PRM were supported (ScanningSWATH and Ion Mobility were under development)

## Inputs

- Sorted floating-point m/z arrays from MS vendor files (SCIEX, Thermo, Agilent) or from repository accessions (e.g., ProteomeXchange PXD021390/PXD018139, MetaboLights MTBLS2119)
- Target precision setting in decimal places (3 dp = 10 ppm, 5 dp = 0.1 ppm)
- Baseline and candidate compressors: Zlib (DotNetZip, level 6), FastPfor library, ZDPD pipeline

## Outputs

- Compressed byte sizes for ZDPD (FastPfor+Zlib) and Zlib-only per m/z array and per dataset
- Compression ratios (ZDPD vs Zlib-only; paper reports ~55% lower m/z data size on average, 49% reduction cited in discussion)
- Decoding-time measurements (paper reports Aird takes 33% of Zlib decoding time; ~400 MB/s m/z array decompression)

## How to apply

Convert the vendor raw file (SCIEX, Thermo, or Agilent instruments) into candidate formats, then compress the sorted floating-point m/z array by first quantizing it to integers at a target precision — 3 decimal places for 10 ppm accuracy, 5 dp for 0.1 ppm — computing deltas of adjacent integers, encoding the delta array with the SIMD-accelerated FastPfor library, and finishing with Zlib at compression level 6 (DotNetZip). Run plain Zlib on the same integer array as the baseline and record: (1) byte size of each output, (2) decoding time per array. Repeat across multiple datasets and acquisition modes (DDA, DIA/SWATH, PRM) drawn from repositories such as ProteomeXchange (PXD021390, PXD018139, PX0001509000) and MetaboLights (MTBLS2119, XCMS1197236) so the comparison is averaged rather than dataset-specific. The paper's own validation bar is a ~55% average m/z size reduction vs Zlib and ~33% of Zlib's decoding time; treat substantially worse ratios on your data as a signal that the delta distribution is unfavorable for PforDelta-style coding.

## Related tools

- **Aird** (Format under benchmark; its ZDPD compressor is the candidate being measured against Zlib-only) — https://github.com/CSi-Studio/Aird-SDK
- **Zlib-Diff-PforDelta (ZDPD)** (Candidate compressor: integer quantization + delta coding via FastPfor + Zlib level 6) — https://github.com/CSi-Studio/Aird-SDK
- **Zlib** (General-purpose baseline compressor (DotNetZip) against which ZDPD size and decoding time are compared)
- **AirdPro** (GUI client that converts vendor files and applies the compression parameters and targeted precision used in the benchmark) — https://github.com/CSi-Studio/AirdPro/releases/
- **MSConvert** (Converts raw vendor data into mzML, mgf, and mz5 comparison formats)

## Examples

```
quantized = np.round(mz_sorted * 100000).astype(np.int32); deltas = np.diff(quantized); zdpd_out = zlib.compress(fastpfor_encode(deltas), 6); zlib_out = zlib.compress(quantized.tobytes(), 6)
```

## Evaluation signals

- ZDPD output must be smaller than Zlib-only output on the same integer m/z arrays; the paper reports ~55% lower m/z data size on average (49% reduction cited in discussion)
- Decoding time of ZDPD should be a fraction of Zlib's — paper reports 33% of Zlib decoding time (3x faster) and ~400 MB/s m/z array decompression
- Comparison must span multiple instruments and acquisition modes (seven datasets from SCIEX, Thermo, Agilent; DDA, DIA/SWATH, PRM) rather than a single file
- Quantization step must match the stated precision: 3 decimal places for 10 ppm accuracy, 5 decimal places for 0.1 ppm accuracy
- Reconstruction round-trip: decoding the ZDPD output must recover the quantized integer m/z array without loss (precision loss is confined to the quantization step)

## Limitations

- No benchmark or scripts are provided to reproduce the 49% size reduction or 3x faster decoding claims; the full comparison pipeline has no stated reproducibility artifact beyond the AirdPro release
- The paper provides no limitation section discussing when ZDPD may underperform versus Zlib
- ZDPD figures are specific to m/z data; newer variants exist (Stack-ZDPD with 5–20% improvement, ZDVB with 10–20% improvement over the ZDPD kernel) that may supersede these numbers
- Test datasets' availability beyond the accession IDs in Table 1 (PXD/IPX/MTBLS/XCMS) is not stated

## Evidence

- [abstract] 55% average size reduction: "Compared with Zlib only, m/z data size is about 55% lower in Aird average."
- [abstract] decoding speed advantage: "Aird merely takes 33% decoding time compared with Zlib."
- [intro] integer quantization and delta coding: "After determining the required precision, Aird converts the floating-point m/z array to an integer array."
- [intro] FastPfor SIMD: "Aird uses the FastPfor library for differential coding of integer arrays [8]. The FastPfor library uses the SIMD technology"
- [methods] Zlib library identity: "Zlib library is DotNetZip from http://dotnetzip.codeplex.com"
- [readme] paper claim restated in README: "resulting a smaller compression file size and a faster decoding speed than these of using Zlib merely"
- [readme] newer variant: "a new compression algorithm Stack-ZDPD is released, which has 5%-20% compression performance improvement compared with ZDPD kernel"
