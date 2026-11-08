---
name: mass-spectrometry-data-compression
description: Use when converting vendor MS files (SCIEX, Thermo, Agilent; DDA, DIA/SWATH,
  PRM, PASEF) into a computation-oriented format where decoding speed and m/z storage
  size matter, or when benchmarking ZDPD against Zlib-only compression.
when_to_use_negative:
- The data must remain strictly lossless in the original floating-point domain — ZDPD
  quantizes m/z floats to integers at a controllable precision
- The required acquisition mode is ScanningSWATH or Ion Mobility, which were under
  development at publication time
- The goal is lossless m/z-only compression of existing mzXML files without format
  conversion — MassComp targets that case instead
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: Aird
  role: Computation-oriented MS data format with controllable precision, flexible
    indexing, and the ZDPD compressor for m/z data
  repo: https://github.com/CSi-Studio/Aird-SDK
- name: Aird-SDK
  role: Java/C#/Python SDK used to load ordered m/z arrays and read Aird files for
    compression and benchmarking
  repo: https://github.com/CSi-Studio/Aird-SDK
- name: AirdPro
  role: C# GUI client that reads vendor files via ProteoWizard and writes .aird/.json
    files with chosen compressor strategies and precision
  repo: https://github.com/CSi-Studio/AirdPro/releases/
- name: Zlib-Diff-PforDelta (ZDPD)
  role: 'm/z compressor: delta-code quantized integers, FastPfor SIMD encoding, wrapped
    with Zlib level 6'
- name: Zlib
  role: Baseline general-purpose compressor (DotNetZip) used for comparison and as
    the outer wrapper of ZDPD
  repo: http://dotnetzip.codeplex.com
- name: MSConvert
  role: Converts the raw data into mzML, mgf, and mz5 comparison formats
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1186/s12859-021-04490-0
    title: 'Aird: a computation-oriented mass spectrometry data format enables a higher
      compression ratio and less decoding time'
  source_content_hash: sha256:03b4f94d8fae8f58c9db8e6dddf0ffd293f98dfebd6b93db7764ff32e1382682
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/CSi-Studio/Aird-SDK
  tools:
  - Aird
  - Aird-SDK
  - AirdPro
  - MSConvert
  - Zlib
  - Zlib-Diff-PforDelta (ZDPD)
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8260869565217391
n_claims: 23
---

# mass-spectrometry-data-compression

## Summary

Compress mass spectrometry m/z arrays using Aird's Zlib-Diff-PforDelta (ZDPD) scheme — quantize floats to integers, delta-code, then SIMD-accelerated FastPfor encoding wrapped in Zlib — to reach ~55% smaller m/z data than Zlib-only and ~3x faster decoding. This addresses the bottleneck that vendor formats and mzML/mgf/mz5 are slow to decode in computational workflows.

## When to use

Use when converting vendor MS files (SCIEX, Thermo, Agilent; DDA, DIA/SWATH, PRM, PASEF) into a computation-oriented format where decoding speed and m/z storage size matter, or when benchmarking ZDPD against Zlib-only compression.

## When NOT to use

- The data must remain strictly lossless in the original floating-point domain — ZDPD quantizes m/z floats to integers at a controllable precision
- The required acquisition mode is ScanningSWATH or Ion Mobility, which were under development at publication time
- The goal is lossless m/z-only compression of existing mzXML files without format conversion — MassComp targets that case instead

## Inputs

- Raw vendor MS files from SCIEX, Thermo, or Agilent instruments (DDA, DIA/SWATH, PRM, DIA_PASEF, DDA_PASEF, MRM acquisition modes)
- Ordered floating-point m/z arrays extracted via Aird-SDK or ProteoWizard SDK
- Targeted precision setting (e.g., 3 decimal places for ten ppm, 5 for 0.1 ppm)

## Outputs

- A .aird binary data file plus a .json Aird index file with BlockIndex entries, compressors, and rangeList
- Compressed integer m/z arrays encoded with ZDPD (delta coding + FastPfor + Zlib)
- Compressed-size and decoding-time comparison metrics vs Zlib-only baseline

## How to apply

Read vendor file data via the ProteoWizard SDK (AirdPro), then determine the targeted precision (ten ppm requires 3 decimal places; 0.1 ppm requires 5 decimal places) and quantize each floating-point m/z array to an integer array. Delta-code adjacent integers and encode the deltas with the FastPfor library, which uses SIMD technology, then wrap with Zlib (DotNetZip, compression level 6) to form the ZDPD compressor. Optionally let the Dynamic-Decider pick the best ComboComp; compress intensity arrays separately (e.g., Log10 compressor, Ignore Zero Intensity Point) and store spectra reordered by acquisition mode with JSON metadata. Benchmark by comparing compressed byte size and decoding wall-clock time of ZDPD against Zlib-alone at level 6 on the same m/z arrays; the paper reports ZDPD m/z size at ~55% of Zlib-only and 33% of Zlib decoding time (49% reduction and 3x faster decoding in the discussion).

## Related tools

- **Aird** (Computation-oriented MS data format with controllable precision, flexible indexing, and the ZDPD compressor for m/z data) — https://github.com/CSi-Studio/Aird-SDK
- **Aird-SDK** (Java/C#/Python SDK used to load ordered m/z arrays and read Aird files for compression and benchmarking) — https://github.com/CSi-Studio/Aird-SDK
- **AirdPro** (C# GUI client that reads vendor files via ProteoWizard and writes .aird/.json files with chosen compressor strategies and precision) — https://github.com/CSi-Studio/AirdPro/releases/
- **Zlib-Diff-PforDelta (ZDPD)** (m/z compressor: delta-code quantized integers, FastPfor SIMD encoding, wrapped with Zlib level 6)
- **Zlib** (Baseline general-purpose compressor (DotNetZip) used for comparison and as the outer wrapper of ZDPD) — http://dotnetzip.codeplex.com
- **MSConvert** (Converts the raw data into mzML, mgf, and mz5 comparison formats)

## Examples

```
pip install AirdSDK; # then load spectra and benchmark ZDPD vs Zlib per the Aird paper protocol
```

## Evaluation signals

- Compressed m/z byte size with ZDPD is about 55% of (i.e., ~49% lower than) Zlib-only compression averaged across datasets
- ZDPD decoding wall-clock time is about 33% of Zlib decoding time (roughly 3x faster), aided by SIMD
- Round-trip decoded m/z values match the original within the chosen precision (3 decimals for ten ppm, 5 decimals for 0.1 ppm)
- The .aird and .json index files share the same file name in the same directory, and the JSON metadata parses and previewable as text
- Decoding throughput on the order of 400 MB/s per m/z array as claimed for Aird

## Limitations

- At publication, Aird supported only DDA, DIA/SWATH, and PRM acquisition modes; ScanningSWATH and Ion Mobility formats were under development
- The paper provides no benchmark scripts or reproducibility artifact to reproduce the 49% size-reduction or 3x faster-decoding claims
- No limitation section discusses when ZDPD may underperform; later AirdPro versions note Stack-ZDPD (5-20% better) and ZDVB (10-20% better, faster decoding) supersede the ZDPD kernel
- The README cites ~65% lower m/z size vs Zlib for the ComboComp format, differing from the paper's 55% figure, so expected ratios vary with compressor configuration

## Evidence

- [abstract] ZDPD quantizes floats: "Aird provides a novel compressor called Zlib-Diff-PforDelta (ZDPD) for m/z data"
- [abstract] ZDPD vs Zlib metrics: [evidence span withheld — anchor status: not_found]
- [intro] Precision and delta coding rationale: "it is an effective way to compress relatively small deltas of adjacent integers instead of the large integers themselves"
- [intro] FastPfor SIMD: "Aird uses the FastPfor library for differential coding of integer arrays [8]. The FastPfor library uses the SIMD technology"
- [methods] Zlib wrapper details: "Zlib library is DotNetZip from http://dotnetzip.codeplex.com"
- [results] Supported modes: "At present, Aird's supporting formats are DDA, DIA/SWATH, PRM."
- [discussion] Missing benchmark artifact: [evidence span withheld — anchor status: not_found]
- [readme] README file pairing: "Aird Index File and Aird Data File show be stored in the same directory with the same file."
