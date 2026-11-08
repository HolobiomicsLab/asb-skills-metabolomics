---
name: mass-spectrometry-intensity-encoding
description: Use when converting vendor MRM/MS raw files to a compact, computation-oriented
  binary format and file size or fast web-side rendering of hundreds of chromatograms
  is the goal.
when_to_use_negative:
- Input is already in Aird (.aird + .) or another compressed interchange format —
  re-encoding adds no information
- The goal is raw-data fidelity for re-processing in vendor software rather than compact
  storage and fast web-side rendering of chromatograms
- Only a handful of spectra need conversion, where vendor or mzML size overhead is
  not a bottleneck
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: MRMPro
  role: MRM data analysis platform whose efficient compression algorithm for fast
    web rendering of hundreds of chromatograms motivates and applies this encoding
    scheme
  repo: github.com/CSi-Studio/MRMPro-Doc
- name: AirdPro
  role: GUI C# client that converts vendor files to Aird format, applying the combinable
    compressors (ZSTD, FastPFor-style integer packing) to intensity and RT arrays
  repo: https://github.com/CSi-Studio/AirdPro/releases/
- name: AirdSDK
  role: Java/C#/Python SDK for reading the encoded .aird arrays back at decoding speed
    suitable for data visualization and analysis
  repo: https://github.com/CSi-Studio/Aird-SDK
- name: FastPFor
  role: Bit-level integer compression codec applied to the delta-coded RT integer
    arrays
- name: ZSTD
  role: General-purpose compressor applied to the FastPFor-encoded blocks before writing
    to disk
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1186/s12859-024-05685-x
    title: 'MRMPro: a web-based tool to improve the speed of manual calibration for
      multiple reaction monitoring data analysis by mass spectrometry'
  source_content_hash: sha256:3f14bb33b722bc271491c3193e5fe8b0836a97882a7551a81f01541d76329fc9
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - AirdPro
  - AirdSDK
  - FastPFor
  - MRMPro
  - ZSTD
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# mass-spectrometry-intensity-encoding

## Summary

Encodes mass spectrometry intensity arrays (and paired retention-time arrays for MRM chromatograms) using Aird's integer delta + FastPFor encoding followed by ZSTD general-purpose compression, achieving >60% size reduction versus vendor formats and >80% versus mzML.

## When to use

Use when converting vendor MRM/MS raw files to a compact, computation-oriented binary format and file size or fast web-side rendering of hundreds of chromatograms is the goal.

## When NOT to use

- Input is already in Aird (.aird + .) or another compressed interchange format — re-encoding adds no information
- The goal is raw-data fidelity for re-processing in vendor software rather than compact storage and fast web-side rendering of chromatograms
- Only a handful of spectra need conversion, where vendor or mzML size overhead is not a bottleneck

## Inputs

- Vendor MRM chromatogram raw file (e.g. SCIEX wiff2, Thermo Xcalibur output)
- mzML-converted counterpart used as a compression baseline
- Retention-time and intensity arrays extracted from each MRM chromatogram

## Outputs

- .aird binary data file with FastPFor/ZSTD-encoded RT and intensity arrays
- . Aird index file (same directory, same file name) containing compressor metadata for the m/z, intensity and mobility arrays
- compression_report.csv with encoded output sizes and size ratios versus vendor and mzML files

## How to apply

Load the vendor chromatogram and its mzML-converted counterpart as size-comparison baselines. For retention-time arrays, quantize each RT to the fifth decimal place (precision 1e-5), convert to integers, and store deltas between adjacent RT integers instead of raw values; delta coding exploits the smoothness of chromatographic acquisition to shrink the value range before bit-level encoding. Encode the delta-coded RT integer arrays with the FastPFor bit-level integer compressor, and separately compress the intensity arrays with Aird's intensity encoding method (Aird's combinable compressor family includes Integer-Purpose compressors such as Variable Byte and Binary Packing). Finish by compressing the FastPFor-encoded blocks with ZSTD (one of Aird's General-Purpose Compressors alongside Brotli, Snappy, and Zlib), then write the encoded chromatogram container (.aird plus . index, stored in the same directory with the same file name). Rationale: stacking delta coding, bit-level packing, and entropy compression is what lets Aird reduce data volume by more than 60% and 80% compared to vendor and mzML formats while keeping decoding fast for web rendering.

## Related tools

- **MRMPro** (MRM data analysis platform whose efficient compression algorithm for fast web rendering of hundreds of chromatograms motivates and applies this encoding scheme) — github.com/CSi-Studio/MRMPro-Doc
- **AirdPro** (GUI C# client that converts vendor files to Aird format, applying the combinable compressors (ZSTD, FastPFor-style integer packing) to intensity and RT arrays) — https://github.com/CSi-Studio/AirdPro/releases/
- **AirdSDK** (Java/C#/Python SDK for reading the encoded .aird arrays back at decoding speed suitable for data visualization and analysis) — https://github.com/CSi-Studio/Aird-SDK
- **FastPFor** (Bit-level integer compression codec applied to the delta-coded RT integer arrays)
- **ZSTD** (General-purpose compressor applied to the FastPFor-encoded blocks before writing to disk)

## Examples

```
pip install AirdSDK
```

## Evaluation signals

- compression_report.csv shows the encoded output is >60% smaller than the vendor file and >80% smaller than the mzML file, per the paper's reported reduction
- The .aird file has a matching . index file with the same base name in the same directory, and the JSON index records the compressor strategy for the intensity array
- Decoding the .aird arrays with AirdSDK reproduces the original RT values within the 1e-5 quantization precision and exactly matches intensity values
- Round-trip check: FastPFor decoding of delta arrays followed by cumulative-sum reconstruction recovers the original RT integer sequence
- Chromatograms render quickly in the MRMPro web interface, consistent with the goal of fast rendering of hundreds of chromatograms

## Limitations

- Compression precision is controllable but lossy for RT arrays: values are quantized to the fifth decimal place (1e-5), so exact original floating-point RTs are not preserved
- The scheme is tailored to Aird's computation-oriented format; downstream tools must read via AirdSDK rather than standard mzML parsers
- Dynamic-decider best-combo selection and memory constraints on large-scale conversions mean encoder settings may need tuning per acquisition method (MRM/SRM support was added in AirdPro V5.1.0 / V4.4.X)

## Evidence

- [abstract] MRM data compression algorithm reduced data volume by more than 60% and 80% compared to vendor and mzML format: "which reduced data volume by more than 60% and 80% respectively compared to vendor and mzML format"
- [intro] Objective: efficient compression algorithm for fast web-side rendering of chromatograms: "An efficient compression algorithm to meet the needs of fast rendering of hundreds of chromatograms on the web side"
- [readme] Aird's compressor family includes general-purpose and integer-purpose compressors: [evidence span withheld — anchor status: external_doc]
- [readme] ZSTD is among Aird's general-purpose compressors: [evidence span withheld — anchor status: external_doc]
- [readme] Aird stores .aird data and . index files together: [evidence span withheld — anchor status: external_doc]
