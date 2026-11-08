---
name: fastpfor-bit-level-integer-compression
description: Use when compressing large numeric arrays from mass-spectrometry chromatograms
  (e.g., MRM retention times) where delta coding plus bit-level integer encoding (FastPFor)
  followed by ZSTD must shrink file size for fast web-side rendering.
when_to_use_negative:
- Input arrays are floating-point raw m/z or intensity values that need Aird's Zlib-Diff-PforDelta
  (ZDPD)/ComboComp m/z pipeline rather than the RT delta + FastPFor scheme
- Data is not mass-spectrometry chromatogram/spectrum arrays — FastPFor integer coding
  requires quantized integers, not arbitrary binary blobs
- A full mzML is required downstream and Aird decoding via AirdSDK is unavailable
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_3314
tools:
- name: MRMPro
  role: Web-based MRM data analysis platform whose compression algorithm delta-codes
    RT arrays and applies FastPFor + ZSTD within the Aird format
  repo: github.com/CSi-Studio/MRMPro-Doc
- name: AirdPro
  role: GUI client (C#, ProteoWizard-based) that converts vendor MRM files to Aird
    format with the FastPFor/ZSTD compressor strategy
  repo: https://github.com/CSi-Studio/AirdPro/releases/
- name: AirdSDK
  role: Java/C#/Python SDK for reading the FastPFor+ZSTD-compressed Aird chromatogram
    data for secondary development and visualization
  repo: https://github.com/CSi-Studio/Aird-SDK
- name: ZSTD
  role: General-purpose compressor applied on top of the FastPFor-encoded RT delta
    blocks
- name: FastPFor
  role: Bit-level integer compression codec used to encode the delta-coded RT integer
    arrays
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1186/s12859-024-05685-x
    title: 'MRMPro: a web-based tool to improve the speed of manual calibration for
      multiple reaction monitoring data analysis by mass spectrometry'
  source_content_hash: sha256:a93d1b67033572a1badf8acccb236920d76f1b4ee370c566cee000db502d2be1
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - AirdPro
  - AirdSDK
  - FastPFor
  - MRMPro
  - ZSTD
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_3314
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6923076923076923
n_claims: 13
---

# FastPFor bit-level integer compression

## Summary

Applies the FastPFor bit-level integer codec to delta-coded retention-time integers in MRMPro's Aird-format MRM chromatograms, then wraps the encoded blocks with ZSTD to achieve the >60%/80% size reductions versus vendor and mzML formats.

## When to use

Use when compressing large numeric arrays from mass-spectrometry chromatograms (e.g., MRM retention times) where delta coding plus bit-level integer encoding (FastPFor) followed by ZSTD must shrink file size for fast web-side rendering.

## When NOT to use

- Input arrays are floating-point raw m/z or intensity values that need Aird's Zlib-Diff-PforDelta (ZDPD)/ComboComp m/z pipeline rather than the RT delta + FastPFor scheme
- Data is not mass-spectrometry chromatogram/spectrum arrays — FastPFor integer coding requires quantized integers, not arbitrary binary blobs
- A full mzML is required downstream and Aird decoding via AirdSDK is unavailable

## Inputs

- Vendor MRM chromatogram file (e.g., Thermo Xcalibur or SCIEX wiff2 raw data)
- mzML-converted counterpart used as a size-comparison baseline
- Delta-coded retention time integer arrays (quantized at 1e-5 precision)

## Outputs

- FastPFor-encoded, ZSTD-compressed retention time blocks in an Aird (.aird) data file with . index
- Separately encoded intensity arrays using Aird's intensity encoding method
- compression_report.csv with file-size ratios of encoded output vs vendor and mzML files

## How to apply

Load the vendor MRM chromatogram and its mzML-converted counterpart as reference baselines for size comparison. Quantize each retention time to an integer at precision 1e-5 (fifth decimal place) and store deltas between adjacent RT integers rather than raw values, since adjacent deltas are small and compress well. Feed the delta-coded integer array to the FastPFor bit-level compression codec, then compress the FastPFor-encoded blocks with ZSTD; intensity arrays are compressed separately using Aird's own intensity encoding method. Write the encoded chromatogram container (Aird data file plus . index) to disk via AirdPro/AirdSDK, and report compression ratios (Aird vs vendor vs mzML) into compression_report.csv; MRMPro reports reductions of more than 60% and 80% respectively.

## Related tools

- **MRMPro** (Web-based MRM data analysis platform whose compression algorithm delta-codes RT arrays and applies FastPFor + ZSTD within the Aird format) — github.com/CSi-Studio/MRMPro-Doc
- **AirdPro** (GUI client (C#, ProteoWizard-based) that converts vendor MRM files to Aird format with the FastPFor/ZSTD compressor strategy) — https://github.com/CSi-Studio/AirdPro/releases/
- **AirdSDK** (Java/C#/Python SDK for reading the FastPFor+ZSTD-compressed Aird chromatogram data for secondary development and visualization) — https://github.com/CSi-Studio/Aird-SDK
- **ZSTD** (General-purpose compressor applied on top of the FastPFor-encoded RT delta blocks)
- **FastPFor** (Bit-level integer compression codec used to encode the delta-coded RT integer arrays)

## Examples

```
airdpro --convert sample.wiff2 --acquisition MRM --compressor rt=FastPFor+Zstd,intensity=AirIntensity --report compression_report.csv
```

## Evaluation signals

- Aird file size is at least 60% smaller than the vendor file and 80% smaller than the mzML counterpart
- compression_report.csv contains Aird-vs-vendor and Aird-vs-mzML size ratios for every converted file
- RT integers are recoverable by cumulative summation of decoded deltas at 1e-5 precision (decoded chromatograms match the mzML baseline)
- The . index lists FastPFor and ZSTD as the compressors for the retention time array in the chromatogramIndex
- Intensity arrays were encoded with Aird's intensity encoding method, distinct from the RT pipeline

## Limitations

- Precision is fixed by quantization to the fifth decimal place (1e-5); finer RT resolution than 1e-5 is lost
- Codec choice is validated for MRM chromatograms (V5.1.0+ added MRM/SRM support in AirdPro); other acquisition modes use different compressor strategies (e.g., ZDPD/Stack-ZDPD for m/z)
- Large-scale conversions previously suffered memory overflow issues; use recent AirdPro V6+ builds for stability

## Evidence

- [other] MRMPro stores adjacent-RT delta values, encodes the delta data with FastPFor, and then compresses the encoded delta data using the ZSTD algorithm.: [evidence span withheld — anchor status: not_found]
- [abstract] which reduced data volume by more than 60% and 80% respectively compared to vendor and mzML format: "which reduced data volume by more than 60% and 80% respectively compared to vendor and mzML format"
- [intro] An efficient compression algorithm to meet the needs of fast rendering of hundreds of chromatograms on the web side: "An efficient compression algorithm to meet the needs of fast rendering of hundreds of chromatograms on the web side"
- [readme] Integer-Purpose Compressors: Variable Byte, Binary Packing; General-Purpose Compressors: Brotli, Snappy, Zstd, and Zlib: "Integer-Purpose Compressors: Variable Byte, Binary Packing
- [New Feature] General-Purpose Compressors: Brotli, Snappy, Zstd, and Zlib"
- [readme] V5.1.0 [New Feature] Supporting MRM acquisition method; [New Feature] Supporting Wiff2 format: "### V5.1.0
- [New Feature] New File Selection UI
- [New Feature] Supporting MRM acquisition method
- [New Feature] Supporting Wiff2 format"
