---
name: zstd-block-compression
description: Use when converting vendor MRM chromatogram data (e.g. Xcalibur/SCIEX
  OS raw files) into the Aird format and you need compact, fast-decoding RT and intensity
  arrays for web rendering of hundreds of chromatograms.
when_to_use_negative:
- The input is not MRM chromatogram data (e.g. DDA/DIA spectra), where the Aird m/z
  compression strategies such as ZDPD/Stack-ZDPD rather than the RT-delta scheme apply
- The target output must be a standard exchange format like mzML or mzXML rather than
  the Aird format
- You only need lossless byte-level compression of already-encoded data with no RT
  delta coding step (plain Zstd applies, but the RT quantization to 1e-5 would be
  skipped)
tools:
- name: MRMPro
  role: MRM data analysis platform whose Aird-based compression pipeline applies the
    RT-delta + FastPFor + ZSTD scheme to MRM chromatograms
  repo: github.com/CSi-Studio/MRMPro-Doc
- name: AirdPro
  role: GUI C# client that converts vendor files into Aird files, including MRM/SRM
    acquisition support
  repo: https://github.com/CSi-Studio/AirdPro/releases/
- name: Aird-SDK
  role: Java/C#/Python SDK for reading the resulting .aird files for secondary development
    and visualization
  repo: https://github.com/CSi-Studio/Aird-SDK
- name: FastPFor
  role: Bit-level integer compression codec applied to the delta-coded RT integer
    arrays before ZSTD
- name: ZSTD
  role: General-purpose block compressor applied to the FastPFor-encoded RT blocks
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1186/s12859-024-05685-x
    title: 'MRMPro: a web-based tool to improve the speed of manual calibration for
      multiple reaction monitoring data analysis by mass spectrometry'
  source_content_hash: sha256:86c9a0185a025d1e2fdd0780e58a4f3e33ec6f4b9f1a46fdb6a8ab46e0738670
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Aird-SDK
  - AirdPro
  - FastPFor
  - MRMPro
  - ZSTD
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# zstd-block-compression

## Summary

Compresses MRM chromatogram retention-time and intensity arrays for the Aird format by delta-coding RTs quantized to 1e-5 precision, encoding deltas with the FastPFor bit-level integer codec, and then compressing the encoded blocks with ZSTD, achieving >60% and >80% size reduction versus vendor and mzML formats respectively.

## When to use

Use when converting vendor MRM chromatogram data (e.g. Xcalibur/SCIEX OS raw files) into the Aird format and you need compact, fast-decoding RT and intensity arrays for web rendering of hundreds of chromatograms.

## When NOT to use

- The input is not MRM chromatogram data (e.g. DDA/DIA spectra), where the Aird m/z compression strategies such as ZDPD/Stack-ZDPD rather than the RT-delta scheme apply
- The target output must be a standard exchange format like mzML or mzXML rather than the Aird format
- You only need lossless byte-level compression of already-encoded data with no RT delta coding step (plain Zstd applies, but the RT quantization to 1e-5 would be skipped)

## Inputs

- Vendor MRM chromatogram raw file (e.g. Thermo Xcalibur or SCIEX OS output)
- mzML-converted counterpart of the vendor file (size baseline)
- Retention time array of each MRM chromatogram (float RT values)

## Outputs

- .aird binary data file containing ZSTD-compressed FastPFor-encoded RT deltas and encoded intensity arrays
- . Aird index file with the same base name in the same directory
- compression_report.csv with file-size ratios of encoded output versus vendor and mzML files

## How to apply

Load the vendor MRM chromatogram and its mzML-converted counterpart as size baselines. Convert each retention time to an integer by quantizing to the fifth decimal place (precision 1e-5) and store deltas between adjacent RT integers instead of raw values. Encode the delta-coded RT integer arrays with the FastPFor bit-level integer compression codec, then compress the FastPFor-encoded blocks with ZSTD; compress intensity arrays separately using Aird's intensity encoding method. Write the encoded chromatogram container to disk as a .aird data file paired with a . index file sharing the same base name, and record the size ratios of encoded output versus vendor and mzML files in compression_report.csv. Correct application is judged by the compression report: Aird files should be more than 60% smaller than vendor and more than 80% smaller than mzML, while remaining decodable via AirdSDK.

## Related tools

- **MRMPro** (MRM data analysis platform whose Aird-based compression pipeline applies the RT-delta + FastPFor + ZSTD scheme to MRM chromatograms) — github.com/CSi-Studio/MRMPro-Doc
- **AirdPro** (GUI C# client that converts vendor files into Aird files, including MRM/SRM acquisition support) — https://github.com/CSi-Studio/AirdPro/releases/
- **Aird-SDK** (Java/C#/Python SDK for reading the resulting .aird files for secondary development and visualization) — https://github.com/CSi-Studio/Aird-SDK
- **FastPFor** (Bit-level integer compression codec applied to the delta-coded RT integer arrays before ZSTD)
- **ZSTD** (General-purpose block compressor applied to the FastPFor-encoded RT blocks)

## Examples

```
pip install AirdSDK
```

## Evaluation signals

- compression_report.csv shows the Aird output is reduced by more than 60% versus the vendor file and more than 80% versus the mzML file
- The .aird data file and . index file share the same base name and reside in the same directory so AirdScanUtil can scan both
- Decoding the RT array via AirdSDK reproduces the original retention times within the 1e-5 quantization precision
- The AirdInfo index records the compressor strategies applied to the chromatogram arrays, confirming FastPFor + ZSTD was actually used
- ZSTD-compressed FastPFor blocks are smaller than Zstd-only compressed raw RT values, confirming the delta + FastPFor pre-encoding step contributed compression gain

## Limitations

- Quantization to 1e-5 precision makes the RT representation lossy below the fifth decimal place, so exact original RT values are not preserved
- ZSTD compression was tested in the context of Aird's MRM chromatogram scheme; the README notes ZDVB/Stack-ZDPD variants were preferred for m/z arrays, so this RT scheme is not the optimal kernel for m/z data
- The scheme is specific to the Aird format ecosystem (AirdPro conversion, AirdSDK reading); consumers must use Aird-aware tooling rather than generic mzML readers

## Evidence

- [abstract] MRM data compression algorithm reduced data volume by more than 60% and 80% compared to vendor and mzML format: "which reduced data volume by more than 60% and 80% respectively compared to vendor and mzML format"
- [intro] Objective: efficient compression for fast web-side chromatogram rendering: "An efficient compression algorithm to meet the needs of fast rendering of hundreds of chromatograms on the web side"
- [readme] AirdPro lists Zstd among its general-purpose compressors: [evidence span withheld — anchor status: external_doc]
- [readme] Aird JSON index records compression strategies per array type including chromatograms for MRM: [evidence span withheld — anchor status: external_doc]
- [readme] Aird chromatogram index stores MRM acquisition mode chromatograms: [evidence span withheld — anchor status: not_found]
