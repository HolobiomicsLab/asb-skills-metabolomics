---
name: retention-time-integer-quantization
description: Use when converting vendor MRM chromatogram files (e.g. from Xcalibur
  or SCIEX OS) into the Aird format and storage size or web-side rendering speed for
  hundreds of chromatograms is the bottleneck.
when_to_use_negative:
- The target format is mzML or mzXML and byte-level size minimization is not required
  — mzML is used here only as a baseline for size comparison
- The data is DDA/DIA spectra (m/z arrays) rather than MRM chromatograms — m/z compression
  in Aird uses other combinable compressors such as ZDPD/Stack-ZDPD instead of this
  RT pipeline
- Lossless raw retention-time values are required with no quantization tolerance —
  this scheme fixes precision at 1e-5
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_3071
tools:
- name: MRMPro
  role: MRM data analysis platform whose Aird-format storage applies the RT delta
    + FastPFor + ZSTD compression scheme
  repo: github.com/CSi-Studio/MRMPro-Doc
- name: AirdPro
  role: GUI client for converting vendor MRM files to Aird files using the compression
    strategy
  repo: https://github.com/CSi-Studio/AirdPro/releases/
- name: Aird-SDK
  role: Java/C#/Python SDK for reading the Aird files produced by this compression
    scheme
  repo: https://github.com/CSi-Studio/Aird-SDK
- name: FastPFor
  role: Bit-level integer compression codec applied to the delta-coded RT integer
    arrays
- name: ZSTD
  role: General-purpose compressor applied on top of the FastPFor-encoded RT blocks
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1186/s12859-024-05685-x
    title: 'MRMPro: a web-based tool to improve the speed of manual calibration for
      multiple reaction monitoring data analysis by mass spectrometry'
  source_content_hash: sha256:151c0ca720552e2cb2d6016334ba76c96e70ed069f0677ec22d571a17564c99e
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Aird-SDK
  - AirdPro
  - FastPFor
  - MRMPro
  - ZSTD
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_3071
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# retention-time-integer-quantization

## Summary

Quantizes MRM chromatogram retention times to integers (precision 1e-5) and stores deltas between adjacent RT integers, then compresses the delta arrays with FastPFor followed by ZSTD as part of MRMPro's Aird-format storage for MRM data.

## When to use

Use when converting vendor MRM chromatogram files (e.g. from Xcalibur or SCIEX OS) into the Aird format and storage size or web-side rendering speed for hundreds of chromatograms is the bottleneck.

## When NOT to use

- The target format is mzML or mzXML and byte-level size minimization is not required — mzML is used here only as a baseline for size comparison
- The data is DDA/DIA spectra (m/z arrays) rather than MRM chromatograms — m/z compression in Aird uses other combinable compressors such as ZDPD/Stack-ZDPD instead of this RT pipeline
- Lossless raw retention-time values are required with no quantization tolerance — this scheme fixes precision at 1e-5

## Inputs

- Vendor MRM chromatogram file (e.g. Thermo Xcalibur or SCIEX OS raw data)
- mzML-converted counterpart of the same MRM run (baseline for size comparison)
- Chromatogram retention time array

## Outputs

- Delta-coded, FastPFor-encoded, ZSTD-compressed RT integer arrays in an Aird (.aird) data file with a . index
- compression_report.csv with file-size ratios of Aird output versus vendor file and mzML file

## How to apply

Load the vendor MRM chromatogram and its mzML-converted counterpart as size-comparison baselines. Quantize each retention time to the fifth decimal place (precision 1e-5) to obtain integers, then store only the deltas between adjacent RT integers instead of raw values. Encode the delta-coded RT integer arrays with the FastPFor bit-level integer compression codec, and compress the FastPFor-encoded blocks with ZSTD; intensity arrays are compressed separately using Aird's intensity encoding. Write the encoded chromatogram container to disk as an Aird file with its JSON index. The rationale is that adjacent MRM scan RTs differ by small, regular amounts, so delta coding at 1e-5 precision yields small integers that FastPFor packs efficiently and ZSTD further shrinks; the paper reports the MRM compression scheme reduced data volume by more than 60% and 80% versus vendor and mzML formats respectively.

## Related tools

- **MRMPro** (MRM data analysis platform whose Aird-format storage applies the RT delta + FastPFor + ZSTD compression scheme) — github.com/CSi-Studio/MRMPro-Doc
- **AirdPro** (GUI client for converting vendor MRM files to Aird files using the compression strategy) — https://github.com/CSi-Studio/AirdPro/releases/
- **Aird-SDK** (Java/C#/Python SDK for reading the Aird files produced by this compression scheme) — https://github.com/CSi-Studio/Aird-SDK
- **FastPFor** (Bit-level integer compression codec applied to the delta-coded RT integer arrays)
- **ZSTD** (General-purpose compressor applied on top of the FastPFor-encoded RT blocks)

## Evaluation signals

- Decoded RT values, after inverse delta transformation and dequantization, reproduce the original chromatogram RTs within the 1e-5 quantization precision
- Encoded Aird output is more than 60% smaller than the vendor file and more than 80% smaller than the mzML baseline
- compression_report.csv is produced with size ratios against both the vendor file and the mzML reference
- The Aird JSON index file and .aird data file share the same directory and base file name as required by the format
- Aird-SDK can decode the compressed RT arrays and render chromatograms without errors

## Limitations

- Precision is fixed at 1e-5; retention times requiring finer resolution are not losslessly representable
- The scheme targets MRM chromatograms; other acquisition types (DIA/SWATH, DDA, PRM, PASEF) use different compressor strategies in Aird
- The MRMPro paper notes it relies on classical algorithms such as these compression schemes, with more advanced analytical algorithms only planned as future work

## Evidence

- [other] FastPFor then ZSTD pipeline: [evidence span withheld — anchor status: not_found]
- [other] Quantization and delta workflow: [evidence span withheld — anchor status: not_found]
- [abstract] Compression result: "which reduced data volume by more than 60% and 80% respectively compared to vendor and mzML format"
- [readme] AirdPro compressor options: [evidence span withheld — anchor status: external_doc]
- [readme] MRM support in AirdPro: [evidence span withheld — anchor status: external_doc]
