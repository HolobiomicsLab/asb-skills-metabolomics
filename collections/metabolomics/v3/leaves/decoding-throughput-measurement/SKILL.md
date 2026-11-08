---
name: decoding-throughput-measurement
description: Use when comparing MS data encoders (ZDPD vs Zlib-only vs other compressors)
  on m/z arrays and you must quantify decode wall-clock time or MB/s throughput, not
  just compressed size.
when_to_use_negative:
- You are benchmarking compressed file SIZE only — use the ZDPD vs Zlib size-reduction
  benchmark (55% lower size) rather than timing decode throughput
- The input is raw vendor instrument data (SCIEX/Thermo/Agilent) that has not yet
  been converted/quantized into integer m/z arrays
- The comparison target is intensity-array compression (log10 compressor, zero-intensity
  filtering) rather than m/z decoding speed
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_3172
tools:
- name: Zlib-Diff-PforDelta (ZDPD)
  role: Encoder under test whose SIMD-accelerated decode path is being timed against
    Zlib
  repo: https://github.com/CSi-Studio/Aird-SDK
- name: Zlib (DotNetZip)
  role: Baseline general-purpose encoder whose inflate time is the reference for the
    33%-decoding-time comparison
  repo: http://dotnetzip.codeplex.com
- name: Aird-SDK
  role: Java/C#/Python library used to load the m/z arrays and read back compressed
    blocks for timed decoding
  repo: https://github.com/CSi-Studio/Aird-SDK
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1186/s12859-021-04490-0
    title: 'Aird: a computation-oriented mass spectrometry data format enables a higher
      compression ratio and less decoding time'
  source_content_hash: sha256:c0cef0b5aa6c652ca6e9ad400bc1771a6f27b0f0fc266c6e2dcea8e36123a9fd
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/CSi-Studio/Aird-SDK
  tools:
  - Aird-SDK
  - Zlib
  - Zlib (DotNetZip)
  - Zlib-Diff-PforDelta (ZDPD)
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# decoding-throughput-measurement

## Summary

Benchmark skill for measuring wall-clock decoding time (decode throughput) of compressed mass-spectrometry m/z arrays, used to show that Aird's SIMD-accelerated ZDPD compressor decodes in only ~33% of the time Zlib needs (~3x faster, ~400 MB/s).

## When to use

Use when comparing MS data encoders (ZDPD vs Zlib-only vs other compressors) on m/z arrays and you must quantify decode wall-clock time or MB/s throughput, not just compressed size.

## When NOT to use

- You are benchmarking compressed file SIZE only — use the ZDPD vs Zlib size-reduction benchmark (55% lower size) rather than timing decode throughput
- The input is raw vendor instrument data (SCIEX/Thermo/Agilent) that has not yet been converted/quantized into integer m/z arrays
- The comparison target is intensity-array compression (log10 compressor, zero-intensity filtering) rather than m/z decoding speed

## Inputs

- Ordered/quantized integer m/z arrays from mass spectrometry spectra (DDA and DIA/SWATH datasets)
- ZDPD-compressed m/z blocks (delta + FastPfor SIMD + Zlib level 6 wrapper)
- Zlib-only compressed m/z blocks (DotNetZip, compression level 6) as baseline

## Outputs

- Decoding wall-clock time per encoder (seconds) aggregated across all spectra per dataset
- Decode throughput in MB of m/z array per second (target: ~400 MB/s for Aird; ZDPD ≈ 33% of Zlib time)
- Ratio comparison table: ZDPD vs Zlib decoding time (~3x speedup)

## How to apply

Load the ordered m/z arrays from converted mass spectrometry datasets (e.g. PXD021390 TripleTOF 5600, PXD018139 Q-Exactive DIA, MTBLS2119, XCMS1197236) using Aird-SDK (Java/C#/Python bindings). Encode each quantized integer m/z array with ZDPD (delta-code adjacent integers, compress deltas with the SIMD-enabled FastPfor library, wrap with Zlib at Compression Level 6 via the DotNetZip library) and with Zlib-only at level 6 as baseline. Then measure the decoding wall-clock time for both encoders across all spectra in each dataset under identical hardware and byte counts. The rationale is that ZDPD's single-instruction-multiple-data (SIMD) decoding in FastPfor processes integer deltas far faster than Zlib's general-purpose byte-stream inflation, so throughput in MB/s of decoded m/z array per second is the discriminating metric. Validate that ZDPD decoding takes about 33% of Zlib's decoding time (roughly 3x faster, ~400 MB/s decompression for the m/z array), consistent with the paper's abstract and discussion.

## Related tools

- **Zlib-Diff-PforDelta (ZDPD)** (Encoder under test whose SIMD-accelerated decode path is being timed against Zlib) — https://github.com/CSi-Studio/Aird-SDK
- **Zlib (DotNetZip)** (Baseline general-purpose encoder whose inflate time is the reference for the 33%-decoding-time comparison) — http://dotnetzip.codeplex.com
- **Aird-SDK** (Java/C#/Python library used to load the m/z arrays and read back compressed blocks for timed decoding) — https://github.com/CSi-Studio/Aird-SDK

## Examples

```
import time, airdsdk; t0=time.perf_counter(); [airdsdk.read_spectrum_mz(i) for i in range(n_spectra)]; print(time.perf_counter()-t0)
```

## Evaluation signals

- ZDPD decoding wall-clock time is ~33% of Zlib-only decoding time (i.e. ~3x faster), matching the abstract metric
- Sustained decode throughput for the m/z array reaches ~400 MB/s with Aird/SIMD decoding
- Decoded m/z arrays are byte-identical (or precision-identical at 3 decimal places for 10 ppm / 5 for 0.1 ppm) to the pre-encoding quantized arrays for both encoders
- Timing is averaged across all spectra of the seven ProteomeXchange/Metabolights datasets (PXD021390, PXD018139, PX0001509000, MTBLS2119, XCMS1197236), not a single spectrum
- Both encoders decode the same number of bytes/arrays so the time comparison is apples-to-apples

## Limitations

- The paper provides no benchmark scripts or reproducibility artifact for the 3x-faster-decoding claim; timing numbers cannot be regenerated from released code alone
- Reported metrics differ slightly between sections (33% of Zlib time / 3x faster in abstract vs discussion; 55% vs 49% for size), so treat exact ratios as approximate
- SIMD decode gains depend on the FastPfor library and integer-array layout; Zlib-only comparison was made with DotNetZip at compression level 6 — different Zlib implementations/levels will shift the ratio

## Evidence

- [abstract] Aird takes 33% decoding time vs Zlib: "Aird merely takes 33% decoding time compared with Zlib."
- [intro] 400 MB/s decode throughput: "Faster decoding speed. Aird can decompress an m/z array of 400 MB per second [8]"
- [discussion] 3x faster decode claim: "ZDPD can decode the data 3 times faster than Zlib."
- [readme] SIMD optimization rationale: "Through SIMD
optimization, the decoding speed of Aird is much higher than that of Zlib."
- [abstract] SIMD technology in ZDPD: "With the high-speed decoding and encoding performance of the single instruction multiple data technology used in the ZDPD"
