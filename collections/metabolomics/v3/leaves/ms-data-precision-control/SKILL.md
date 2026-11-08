---
name: ms-data-precision-control
description: Use when storing, converting, or compressing MS m/z arrays and you must
  fix a controlled-precision loss level (ppm target) before quantization, e.g. converting
  vendor/mzML data to an integer-based format like Aird or evaluating ZDPD-style compression.
when_to_use_negative:
- Lossless m/z fidelity is required (e.g. downstream centroiding or exact mass recalibration
  that cannot tolerate the chosen ppm rounding)
- The input is intensity data rather than m/z — Aird applies different strategies
  to the intensity array (e.g. the log10 compressor, ignoring zero-intensity points)
- The acquisition mode is unsupported by Aird (the paper states only DDA, DIA/SWATH,
  and PRM are supported; ScanningSWATH and Ion Mobility were under development)
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: Aird-SDK
  role: Java/C#/Python SDK for reading and writing Aird files whose m/z arrays are
    stored as precision-controlled integers
  repo: https://github.com/CSi-Studio/Aird-SDK
- name: AirdPro
  role: GUI conversion client that reads vendor files via ProteoWizard and compresses
    m/z arrays 'according to the compression parameters and the user's targeted precision'
  repo: https://github.com/CSi-Studio/AirdPro/releases/
- name: MS-Numpress
  role: 'Prior art: a set of compressors providing algorithms with controllable precision,
    cited as the conceptual predecessor of Aird''s precision-controlled m/z storage'
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1186/s12859-021-04490-0
    title: 'Aird: a computation-oriented mass spectrometry data format enables a higher
      compression ratio and less decoding time'
  source_content_hash: sha256:01b2b9e7455eac139201164840fd4037c33807f3fd066d16d5ec60292ff821f7
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/CSi-Studio/Aird-SDK
  tools:
  - Aird-SDK
  - AirdPro
  - MS-Numpress
  - Zlib
  - Zlib-Diff-PforDelta (ZDPD)
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 14
---

# ms-data-precision-control

## Summary

Method for controlling precision of mass spectrometry m/z data storage, following Aird's approach of determining the required mass accuracy (e.g. 10 ppm accuracy → 3 decimal places, 0.1 ppm → 5 dp), then converting the floating-point m/z array into an integer array so that downstream compressors (FastPfor delta coding, Zlib) operate on small integers.

## When to use

Use when storing, converting, or compressing MS m/z arrays and you must fix a controlled-precision loss level (ppm target) before quantization, e.g. converting vendor/mzML data to an integer-based format like Aird or evaluating ZDPD-style compression.

## When NOT to use

- Lossless m/z fidelity is required (e.g. downstream centroiding or exact mass recalibration that cannot tolerate the chosen ppm rounding)
- The input is intensity data rather than m/z — Aird applies different strategies to the intensity array (e.g. the log10 compressor, ignoring zero-intensity points)
- The acquisition mode is unsupported by Aird (the paper states only DDA, DIA/SWATH, and PRM are supported; ScanningSWATH and Ion Mobility were under development)

## Inputs

- Sorted floating-point m/z array from an MS spectrum (from vendor files, mzML, mzXML, or raw scans)
- Target mass accuracy in ppm (e.g. 10 ppm → 3 dp; 0.1 ppm → 5 dp)
- Compression parameters / targeted precision as set in the AirdPro conversion config

## Outputs

- Integer m/z array quantized at the chosen decimal precision
- Delta array of adjacent integer m/z values (optional, for ZDPD-style coding)
- Compressed m/z block (FastPfor + Zlib) plus Aird metadata (.json index) recording the precision and compressors used

## How to apply

First decide the target precision from the required mass accuracy: 10 ppm accuracy corresponds to 3 decimal places (dp) and 0.1 ppm corresponds to 5 dp. Scale the sorted floating-point m/z values by 10^dp and round to integers, mirroring how Aird 'chooses the integer type as the storage type for the m/z array'. Optionally compute deltas of adjacent integers, since it is effective to compress relatively small deltas of adjacent integers instead of the large integers themselves. Feed the integer (or delta) array to FastPfor differential coding with SIMD, then wrap with Zlib (DotNetZip, level 6) as in the ZDPD compressor. Validate that the quantized-then-reconstructed m/z values stay within the chosen ppm tolerance and that the compressed output is smaller than Zlib-only compression of the same integer array.

## Related tools

- **Aird-SDK** (Java/C#/Python SDK for reading and writing Aird files whose m/z arrays are stored as precision-controlled integers) — https://github.com/CSi-Studio/Aird-SDK
- **AirdPro** (GUI conversion client that reads vendor files via ProteoWizard and compresses m/z arrays 'according to the compression parameters and the user's targeted precision') — https://github.com/CSi-Studio/AirdPro/releases/
- **MS-Numpress** (Prior art: a set of compressors providing algorithms with controllable precision, cited as the conceptual predecessor of Aird's precision-controlled m/z storage)

## Examples

```
import numpy as np; dp = 3; mz_int = np.rint(mz_sorted * 10**dp).astype(np.int64); deltas = np.diff(mz_int)  # 10 ppm -> 3 dp, then adjacent-integer deltas for ZDPD coding
```

## Evaluation signals

- Reconstructing floats from the integer array reproduces original m/z values within the chosen tolerance (10 ppm for 3 dp, 0.1 ppm for 5 dp)
- The delta array contains only small near-constant integers — large deltas indicate wrong decimal-precision choice or unsorted m/z input
- ZDPD (FastPfor + Zlib) output must be smaller than Zlib-only compression of the same integer array; the paper reports m/z data size about 55% lower than Zlib on average
- Decoding throughput should approach the reported ~400 MB/s m/z array decompression, i.e. Aird takes roughly 33% of Zlib's decoding time
- The .json Aird index records the compressor list and precision so the quantization is reproducible

## Limitations

- Precision is fixed at conversion time; no automatic tuner guarantees the ppm tolerance across heterogeneous instruments (SCIEX, Thermo, Agilent datasets were tested, but the choice is user-driven)
- The paper provides no benchmark scripts to reproduce the reported 49% size reduction or 3x faster decoding claims, so expected gains are paper-reported averages rather than reproducible artifacts
- Newer Aird kernels (Stack-ZDPD, ZDVB) supersede plain ZDPD with 5-20% and 10-20% further improvements, so results tied to the ZDPD kernel may understate current performance

## Evidence

- [intro] After determining the required precision, Aird converts the floating-point m/z array to an integer array.: "After determining the required precision, Aird converts the floating-point m/z array to an integer array."
- [methods] integer storage type: "Aird chooses the integer type as the storage type for the m/z array"
- [intro] delta compression rationale: "it is an effective way to compress relatively small deltas of adjacent integers instead of the large integers themselves"
- [methods] compression per targeted precision: "AirdPro starts to compress the m/z and intensity arrays according to the compression parameters and the user's targeted precision"
- [readme] controllable precision framing: "an opensource and computation-oriented format with controllable precision, flexible indexing strategies, and high compression rate"
- [full_text] ppm-to-decimal-places mapping: [evidence span withheld — anchor status: not_found]
