---
name: compression-ratio-benchmarking
description: Use when converting vendor MRM chromatograms to a compressed format (e.g.,
  Aird) and you must quantify storage savings versus vendor and mzML files, or when
  implementing RT delta + FastPFor + ZSTD encoding.
when_to_use_negative:
- Data are ion mobility or DIA/SWATH spectra rather than MRM chromatograms — the RT-delta
  chromatogram scheme targets MRM acquisition mode
- 'You need lossless raw retention times: RTs are quantized at 1e-5 precision, so
  finer-resolution RT information is not preserved'
- The goal is faster search-time decoding of m/z arrays (use Aird-Slice/ZDPD-style
  m/z compression instead of the chromatogram RT pipeline)
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0121
tools:
- name: MRMPro
  role: MRM data analysis platform whose Aird-format MRM chromatogram storage implements
    the delta + FastPFor + ZSTD compression being benchmarked
  repo: https://github.com/CSi-Studio/MRMPro-Doc
- name: AirdPro
  role: C# GUI client that converts vendor files into Aird format, supporting MRM/SRM
    acquisition method conversion
  repo: https://github.com/CSi-Studio/AirdPro
- name: Aird-SDK
  role: Java/C#/Python SDK for reading the resulting .aird chromatogram container
    for verification and secondary development
  repo: https://github.com/CSi-Studio/Aird-SDK
- name: FastPFor
  role: bit-level integer compression codec applied to the delta-coded RT integer
    arrays
- name: ZSTD
  role: general-purpose compressor applied on top of FastPFor-encoded RT blocks
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1186/s12859-024-05685-x
    title: 'MRMPro: a web-based tool to improve the speed of manual calibration for
      multiple reaction monitoring data analysis by mass spectrometry'
  source_content_hash: sha256:2426a2b1e8f0fa3ef9c42ed05ece9bf15a6e1df20fa8c1d895556d57c9d151e2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/CSi-Studio/MRMPro-Doc
  tools:
  - Aird-SDK
  - AirdPro
  - FastPFor
  - MRMPro
  - ZSTD
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# compression-ratio-benchmarking

## Summary

Benchmark the size of encoded mass-spectrometry chromatogram containers against vendor and mzML baselines by delta-quantizing retention times (precision 1e-5), FastPFor-encoding the RT deltas, ZSTD-compressing the result, and reporting vendor/Aird and mzML/Aird size ratios (MRMPro reports >60% and >80% reduction respectively).

## When to use

Use when converting vendor MRM chromatograms to a compressed format (e.g., Aird) and you must quantify storage savings versus vendor and mzML files, or when implementing RT delta + FastPFor + ZSTD encoding.

## When NOT to use

- Data are ion mobility or DIA/SWATH spectra rather than MRM chromatograms — the RT-delta chromatogram scheme targets MRM acquisition mode
- You need lossless raw retention times: RTs are quantized at 1e-5 precision, so finer-resolution RT information is not preserved
- The goal is faster search-time decoding of m/z arrays (use Aird-Slice/ZDPD-style m/z compression instead of the chromatogram RT pipeline)

## Inputs

- vendor MRM chromatogram file (e.g., SCIEX Wiff/Wiff2, Thermo raw)
- mzML-converted counterpart of the same MRM run (size baseline)
- retention time array of each MRM transition chromatogram
- intensity array of each MRM transition chromatogram

## Outputs

- encoded chromatogram container (.aird data file with . Aird index)
- compression_report.csv with encoded/vendor and encoded/mzML size ratios
- FastPFor-encoded, ZSTD-compressed RT delta blocks

## How to apply

Load the vendor MRM chromatogram and its mzML-converted counterpart as reference size baselines. Quantize each retention time to an integer at 1e-5 precision and store deltas between adjacent RT integers rather than raw values, since neighboring RTs are nearly identical so deltas are small and highly compressible. Encode the delta arrays with the FastPFor bit-level integer codec, then apply ZSTD to the FastPFor blocks; compress intensity arrays separately using Aird's intensity encoding method. Write the encoded chromatogram container (Aird .aird plus . index) to disk, compute file sizes of the encoded output versus the vendor and mzML files, and write the ratios to compression_report.csv. Correctness is judged by successful round-trip decoding and by size ratios consistent with the reported >60% vs vendor and >80% vs mzML reductions.

## Related tools

- **MRMPro** (MRM data analysis platform whose Aird-format MRM chromatogram storage implements the delta + FastPFor + ZSTD compression being benchmarked) — https://github.com/CSi-Studio/MRMPro-Doc
- **AirdPro** (C# GUI client that converts vendor files into Aird format, supporting MRM/SRM acquisition method conversion) — https://github.com/CSi-Studio/AirdPro
- **Aird-SDK** (Java/C#/Python SDK for reading the resulting .aird chromatogram container for verification and secondary development) — https://github.com/CSi-Studio/Aird-SDK
- **FastPFor** (bit-level integer compression codec applied to the delta-coded RT integer arrays)
- **ZSTD** (general-purpose compressor applied on top of FastPFor-encoded RT blocks)

## Examples

```
from aird_sdk import AirdSDK; sdk = AirdSDK(); sdk.load("sample.aird")  # verify round-trip RTs, then diff sample.aird+sample. size vs vendor/.mzML for compression_report.csv
```

## Evaluation signals

- compression_report.csv contains size ratios for encoded output vs vendor file and vs mzML file for every converted sample
- Encoded Aird file is more than 60% smaller than the vendor file and more than 80% smaller than the mzML file, matching the reported reductions
- Round-trip decode of the FastPFor+ZSTD RT delta blocks via AirdSDK reconstructs RTs matching the originals at 1e-5 quantization
- The Aird index (.) is stored in the same directory with the same base filename as the .aird data file so AirdScanUtil can pair them
- AirdInfo lists the correct compressors for the RT and intensity arrays and type=MRM with a populated chromatogramIndex

## Limitations

- RT values are lossily quantized at precision 1e-5; reconstruction fidelity is bounded by this quantization
- FastPFor is an integer codec, so floating-point RTs must first be delta-coded integers; intensity arrays require a separate intensity encoding method
- Reported 60%/80% reductions are aggregate claims; per-file ratios depend on chromatogram sampling density and may vary

## Evidence

- [other] MRMPro stores adjacent-RT delta values, encodes the delta data with FastPFor, and then compresses the encoded delta data using the ZSTD algorithm.: [evidence span withheld — anchor status: not_found]
- [abstract] reduced data volume by more than 60% and 80% respectively compared to vendor and mzML format: "which reduced data volume by more than 60% and 80% respectively compared to vendor and mzML format"
- [readme] General-Purpose Compressors: Brotli, Snappy, Zstd, and Zlib: [evidence span withheld — anchor status: external_doc]
- [readme] Aird Index File Suffix: . / Aird Data File Suffix: .aird / Aird Index File and Aird Data File show be stored in the same directory with the same file.: [evidence span withheld — anchor status: external_doc]
- [intro] An efficient compression algorithm to meet the needs of fast rendering of hundreds of chromatograms on the web side: "An efficient compression algorithm to meet the needs of fast rendering of hundreds of chromatograms on the web side"
