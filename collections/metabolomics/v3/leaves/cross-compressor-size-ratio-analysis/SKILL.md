---
name: cross-compressor-size-ratio-analysis
description: Use when evaluating whether a domain-specific MS data compressor (e.g.
  ZDPD, Stack-ZDPD, ZDVB) beats a general-purpose codec like Zlib on the same m/z
  arrays, using size ratio and decoding-time ratio as the metrics.
when_to_use_negative:
- When compressing intensity arrays rather than m/z arrays — the ZDPD-vs-Zlib result
  is specific to m/z data; Aird uses separate strategies for intensity (e.g. Log10
  compressor, Ignore Zero Intensity Point filtering)
- When the goal is a lossless byte-identical format comparison, since ZDPD relies
  on controllable-precision quantization of floating-point m/z to integers before
  compression
- When evaluating the newest Aird compressors — Stack-ZDPD and ZDVB supersede the
  ZDPD kernel (5–20% and 10–20% better compression respectively), so a ZDPD-only baseline
  understates current Aird performance
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: Zlib-Diff-PforDelta (ZDPD)
  role: 'Candidate compressor under test: delta coding + FastPfor (SIMD) + Zlib level
    6 wrapper for m/z arrays'
  repo: https://github.com/CSi-Studio/Aird-SDK
- name: Zlib
  role: General-purpose baseline encoder (DotNetZip implementation, compression level
    6) against which size and decoding time are compared
  repo: http://dotnetzip.codeplex.com
- name: Aird-SDK
  role: Provides Java/C#/Python bindings to load the ordered m/z arrays and exercise
    ZDPD encode/decode for the benchmark
  repo: https://github.com/CSi-Studio/Aird-SDK
- name: AirdPro
  role: GUI client used to convert vendor files to Aird format (reading via ProteoWizard
    SDK) before benchmarking
  repo: https://github.com/CSi-Studio/AirdPro/releases/
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1186/s12859-021-04490-0
    title: 'Aird: a computation-oriented mass spectrometry data format enables a higher
      compression ratio and less decoding time'
  source_content_hash: sha256:ce8d76848af57c67fe02ac523f4f79dea3acd971aca947d2cccaa058018f3019
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/CSi-Studio/Aird-SDK
  tools:
  - Aird-SDK
  - AirdPro
  - Zlib
  - Zlib-Diff-PforDelta (ZDPD)
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 16
---

# cross-compressor-size-ratio-analysis

## Summary

Benchmark methodology for comparing a specialized m/z compressor (Aird's ZDPD: delta coding + FastPfor + Zlib wrapper) against a general-purpose baseline (Zlib-only, level 6) by measuring compressed byte size and decoding wall-clock time over the same spectra. The paper reports ZDPD reaches ~55% of Zlib-only m/z size while decoding in ~33% of Zlib's time via SIMD.

## When to use

Use when evaluating whether a domain-specific MS data compressor (e.g. ZDPD, Stack-ZDPD, ZDVB) beats a general-purpose codec like Zlib on the same m/z arrays, using size ratio and decoding-time ratio as the metrics.

## When NOT to use

- When compressing intensity arrays rather than m/z arrays — the ZDPD-vs-Zlib result is specific to m/z data; Aird uses separate strategies for intensity (e.g. Log10 compressor, Ignore Zero Intensity Point filtering)
- When the goal is a lossless byte-identical format comparison, since ZDPD relies on controllable-precision quantization of floating-point m/z to integers before compression
- When evaluating the newest Aird compressors — Stack-ZDPD and ZDVB supersede the ZDPD kernel (5–20% and 10–20% better compression respectively), so a ZDPD-only baseline understates current Aird performance

## Inputs

- Ordered floating-point m/z arrays from converted mass spectrometry datasets (vendor files converted via AirdPro/ProteoWizard SDK; comparison formats mzML, mgf, mz5 produced by MSConvert)
- Targeted mass precision setting (ten ppm → 3 decimal places; 0.1 ppm → 5 decimal places)
- Zlib compression parameters (level 6, DotNetZip library)

## Outputs

- Per-spectrum compressed byte sizes for ZDPD-encoded and Zlib-only m/z arrays
- Per-spectrum decoding wall-clock times for ZDPD and Zlib
- Aggregate size-reduction ratio (expected ~55% of Zlib-only size) and decoding-time ratio (expected ~33% of Zlib time)

## How to apply

Load the ordered m/z arrays from converted mass spectrometry datasets (DDA/DIA/SWATH/PRM, from SCIEX, Thermo, Agilent instruments) using AirdSDK's Java/C#/Python bindings. Quantize each floating-point m/z array to an integer array at the targeted precision — Aird's scheme maps ten ppm to 3 decimal places and 0.1 ppm to 5 decimal places — then encode with ZDPD: delta-code adjacent integers, compress the deltas with the SIMD-accelerated FastPfor library, and wrap the result with Zlib (Compression Level = 6, DotNetZip library). Compress the identical arrays with Zlib alone at level 6 as the baseline. For every spectrum in each dataset, record the compressed byte size and decoding wall-clock time for both encoders; report the mean size ratio (ZDPD/Zlib ≈ 55%, i.e. ~49% average size reduction cited in the discussion) and the mean decoding-time ratio (ZDPD ≈ 33% of Zlib, roughly 3x faster). The rationale is that delta coding of small adjacent-integer deltas plus SIMD bit-packing exploits the monotonic, dense structure of m/z arrays that general-purpose codecs cannot.

## Related tools

- **Zlib-Diff-PforDelta (ZDPD)** (Candidate compressor under test: delta coding + FastPfor (SIMD) + Zlib level 6 wrapper for m/z arrays) — https://github.com/CSi-Studio/Aird-SDK
- **Zlib** (General-purpose baseline encoder (DotNetZip implementation, compression level 6) against which size and decoding time are compared) — http://dotnetzip.codeplex.com
- **Aird-SDK** (Provides Java/C#/Python bindings to load the ordered m/z arrays and exercise ZDPD encode/decode for the benchmark) — https://github.com/CSi-Studio/Aird-SDK
- **AirdPro** (GUI client used to convert vendor files to Aird format (reading via ProteoWizard SDK) before benchmarking) — https://github.com/CSi-Studio/AirdPro/releases/

## Examples

```
pip install AirdSDK
```

## Evaluation signals

- Mean compressed m/z size under ZDPD is about 55% of the Zlib-only size (paper discussion cites 49% average size reduction)
- Mean ZDPD decoding time is about 33% of Zlib decoding time, i.e. roughly 3x faster
- Quantization correctness: 10 ppm precision rounds m/z to 3 decimal places and 0.1 ppm to 5 decimal places before integer delta coding
- Decoding speed reaches the reported order of ~400 MB/s m/z array decompression enabled by SIMD in FastPfor
- Both encoders are run over the identical spectra and datasets (e.g. PXD021390, PXD018139, PX0001509000, MTBLS2119, XCMS1197236) so the comparison is paired

## Limitations

- The paper provides no benchmark or benchmark scripts to reproduce the 49% size-reduction or 3x faster-decoding claims; only downloadable AirdPro releases and AirdSDK source are available
- The README claims a different figure ('about 65% lower in the Aird on average') than the paper's 55%, indicating the reported ratio is version- and dataset-dependent
- Results are specific to m/z arrays at Zlib level 6 with the DotNetZip implementation; they may not transfer to other precision settings, codecs (Brotli, Snappy, Zstd added in AirdPro V3), or acquisition modes
- Aird supported only DDA, DIA/SWATH, and PRM at publication time; ScanningSWATH and Ion Mobility formats were under development, limiting benchmark coverage

## Evidence

- [abstract] ZDPD size advantage: "Compared with Zlib only, m/z data size is about 55% lower in Aird average."
- [abstract] ZDPD decoding speed: "Aird merely takes 33% decoding time compared with Zlib."
- [abstract] SIMD rationale: "With the high-speed decoding and encoding performance of the single instruction multiple data technology used in the ZDPD"
- [intro] Delta-coding rationale: "it is an effective way to compress relatively small deltas of adjacent integers instead of the large integers themselves"
- [intro] FastPfor library: "Aird uses the FastPfor library for differential coding of integer arrays [8]. The FastPfor library uses the SIMD technology"
- [methods] Zlib baseline implementation: "Zlib library is DotNetZip from http://dotnetzip.codeplex.com"
- [readme] README headline metric: [evidence span withheld — anchor status: external_doc]
- [readme] SIMD decoding claim in README: [evidence span withheld — anchor status: external_doc]
