---
name: delta-encoding-compression-benchmarking
description: Use when comparing compressed m/z array size or decoding speed between
  Aird's ZDPD and a Zlib-only baseline across mass-spec datasets, or when implementing
  quantize-then-delta-then-PforDelta m/z compression pipelines.
when_to_use_negative:
- Input arrays are intensity or ion mobility arrays, not ordered m/z arrays — the
  55%/33% metrics apply specifically to m/z data compression
- Lossless raw m/z fidelity is required — Aird quantizes floating-point m/z to integers
  at a chosen precision, so the method is lossy at the quantization level
- Acquisition mode is ScanningSWATH or Ion Mobility — the paper states these formats
  were still under development and unsupported at the time
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_0121
tools:
- name: Aird-SDK
  role: Load ordered m/z arrays from Aird files and access the ZDPD encoder/decoder
    for benchmarking (Java, C#, Python bindings)
  repo: https://github.com/CSi-Studio/Aird-SDK
- name: Zlib-Diff-PforDelta (ZDPD)
  role: 'Treatment compressor: delta-code quantized integer m/z arrays, compress deltas
    with SIMD FastPfor, wrap with Zlib'
  repo: https://github.com/CSi-Studio/Aird-SDK
- name: Zlib
  role: Baseline compressor applied alone at Compression Level = 6 to the identical
    m/z arrays for the size/time comparison
  repo: http://dotnetzip.codeplex.com
- name: AirdPro
  role: GUI client to convert vendor files (SCIEX/Thermo/Agilent) into Aird format
    with the chosen compression parameters and precision
  repo: https://github.com/CSi-Studio/AirdPro/releases/
- name: MSConvert
  role: Convert raw data into mzML, mgf, and mz5 formats for the baseline format comparison
    against Aird
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1186/s12859-021-04490-0
    title: 'Aird: a computation-oriented mass spectrometry data format enables a higher
      compression ratio and less decoding time'
  source_content_hash: sha256:ac1c775614b6b7abc3e7882c97a7c02aeae40b8b737eda0e22a98ab69ab3713e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/CSi-Studio/Aird-SDK
  tools:
  - Aird-SDK
  - AirdPro
  - MSConvert
  - Zlib
  - Zlib-Diff-PforDelta (ZDPD)
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# delta-encoding-compression-benchmarking

## Summary

Benchmark Aird's Zlib-Diff-PforDelta (ZDPD) m/z compressor against Zlib-only compression on mass spectrometry arrays, measuring compressed size and decoding time. ZDPD combines integer delta coding via the SIMD-accelerated FastPfor library with Zlib wrapping to achieve smaller files and ~3x faster decoding.

## When to use

Use when comparing compressed m/z array size or decoding speed between Aird's ZDPD and a Zlib-only baseline across mass-spec datasets, or when implementing quantize-then-delta-then-PforDelta m/z compression pipelines.

## When NOT to use

- Input arrays are intensity or ion mobility arrays, not ordered m/z arrays — the 55%/33% metrics apply specifically to m/z data compression
- Lossless raw m/z fidelity is required — Aird quantizes floating-point m/z to integers at a chosen precision, so the method is lossy at the quantization level
- Acquisition mode is ScanningSWATH or Ion Mobility — the paper states these formats were still under development and unsupported at the time

## Inputs

- Ordered floating-point m/z arrays from mass spectrometry spectra (DDA, DIA/SWATH, PRM acquisitions)
- Vendor raw files from SCIEX, Thermo, or Agilent instruments convertible via AirdPro or MSConvert
- Targeted m/z precision setting (e.g. 10 ppm -> 3 decimal places, 0.1 ppm -> 5 decimal places)

## Outputs

- Compressed byte sizes for ZDPD-encoded vs Zlib-only (level 6) m/z arrays per spectrum and per dataset
- Decoding wall-clock times for ZDPD vs Zlib per spectrum and per dataset
- Aggregate size-reduction ratio (expected ~55%) and decoding-time ratio (expected ~33%)

## How to apply

Load the ordered m/z arrays from converted mass spectrometry datasets using AirdSDK (Java/C#/Python bindings; e.g. net.csibio.aird:aird-sdk 2.5.1.1 via Maven or `pip install AirdSDK`). Quantize each floating-point m/z array to integers at the targeted precision — Aird's scheme maps ten ppm to 3 decimal places and 0.1 ppm to 5 decimal places — then encode each integer array with ZDPD: delta-code adjacent integers, compress the deltas with the FastPfor library (SIMD-accelerated), and wrap with Zlib at Compression Level = 6 (DotNetZip library). Compress the identical m/z arrays with Zlib alone at level 6 as the baseline encoder, and measure compressed byte size and decoding wall-clock time for both encoders across all spectra in each dataset. Datasets in the paper span SCIEX, Thermo, and Agilent instruments (e.g. PXD021390 TripleTOF 5600 DDA, PXD018139 Q-Exactive DIA, MTBLS2119 metabolomics). Judge success against the paper's targets: ZDPD size ~55% of Zlib-only on average and decoding time ~33% of Zlib's (roughly 3x faster); note the discussion cites a more conservative 49% average size reduction.

## Related tools

- **Aird-SDK** (Load ordered m/z arrays from Aird files and access the ZDPD encoder/decoder for benchmarking (Java, C#, Python bindings)) — https://github.com/CSi-Studio/Aird-SDK
- **Zlib-Diff-PforDelta (ZDPD)** (Treatment compressor: delta-code quantized integer m/z arrays, compress deltas with SIMD FastPfor, wrap with Zlib) — https://github.com/CSi-Studio/Aird-SDK
- **Zlib** (Baseline compressor applied alone at Compression Level = 6 to the identical m/z arrays for the size/time comparison) — http://dotnetzip.codeplex.com
- **AirdPro** (GUI client to convert vendor files (SCIEX/Thermo/Agilent) into Aird format with the chosen compression parameters and precision) — https://github.com/CSi-Studio/AirdPro/releases/
- **MSConvert** (Convert raw data into mzML, mgf, and mz5 formats for the baseline format comparison against Aird)

## Examples

```
pip install AirdSDK
```

## Evaluation signals

- ZDPD compressed m/z size should be about 55% of the Zlib-only (level 6) size averaged over the dataset
- ZDPD decoding wall-clock time should be about 33% of Zlib's (roughly 3x faster), attributable to SIMD in FastPfor
- Round-trip decode of ZDPD-compressed arrays must reproduce the quantized integer m/z arrays exactly (lossless above quantization precision)
- Discussion-section benchmark reports ZDPD reduces sizes by 49% on average vs Zlib — treat 49–55% as the plausible range for the original ZDPD kernel
- Verify both encoders process identical input arrays across all spectra of each dataset (e.g. PXD021390, PXD018139, PX0001509000, MTBLS2119, XCMS1197236)

## Limitations

- No benchmark or benchmark scripts are provided to reproduce the 49% size-reduction or 3x-faster-decoding claims; only AirdPro releases are downloadable
- Metrics are inconsistent across sources: abstract says ~55% size reduction, discussion says 49%, and the current README says ~65% (reflecting newer kernels like Stack-ZDPD and ZDVB rather than the paper's ZDPD)
- Aird's supported acquisition modes at publication were DDA, DIA/SWATH, and PRM only; ScanningSWATH and Ion Mobility were under development
- The paper's discussion does not describe trade-offs or situations where ZDPD may underperform Zlib

## Evidence

- [abstract] Compared with Zlib only, m/z data size is about 55% lower in Aird average.: "Compared with Zlib only, m/z data size is about 55% lower in Aird average."
- [abstract] Aird merely takes 33% decoding time compared with Zlib.: "Aird merely takes 33% decoding time compared with Zlib."
- [intro] Aird uses the FastPfor library for differential coding of integer arrays [8]. The FastPfor library uses the SIMD technology: "Aird uses the FastPfor library for differential coding of integer arrays [8]. The FastPfor library uses the SIMD technology"
- [methods] Zlib library is DotNetZip from http://dotnetzip.codeplex.com: "Zlib library is DotNetZip from http://dotnetzip.codeplex.com"
- [discussion] ZDPD can reduce the sizes by 49% than that by Zlib on average: "ZDPD can reduce the sizes by 49% than that by Zlib on average"
- [readme] Compared with Zlib, m/z data is about 65% lower in the Aird on average.: [evidence span withheld — anchor status: external_doc]
