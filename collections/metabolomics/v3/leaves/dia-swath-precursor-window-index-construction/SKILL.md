---
name: dia-swath-precursor-window-index-construction
description: Use when converting DIA/SWATH vendor MS data to the Aird format and you
  need metadata indexes that let analysis programs read an m/z window's MS2 spectra
  as a contiguous block.
when_to_use_negative:
- The data are DDA — use the MS1-MS2 group index instead, which reorders spectra into
  contiguous MS1 blocks with adjacent MS2 groups
- The acquisition mode is MRM (use chromatogramIndex), or ScanningSWATH / Ion Mobility
  formats that are still under development
- The output target is mzML, mgf, or mz5 — those formats converted with MSConvert
  do not carry Aird's precursor-window correlation indexes
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: AirdPro
  role: GUI client that reads vendor file data via the ProteoWizard SDK, reorders
    DIA/SWATH spectra by precursor m/z window, and writes the JSON index next to the
    .aird file
  repo: https://github.com/CSi-Studio/AirdPro/releases/
- name: Aird-SDK
  role: Java/C#/Python SDK providing the AirdMetaData.json schema and readers that
    consume the precursor m/z range-MS2 group index for fast windowed retrieval
  repo: https://github.com/CSi-Studio/Aird-SDK
- name: MSConvert
  role: Used in the benchmark comparison to convert raw data into mzML, mgf, and mz5
    formats; Aird's window index is the alternative to those conversion targets
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1186/s12859-021-04490-0
    title: 'Aird: a computation-oriented mass spectrometry data format enables a higher
      compression ratio and less decoding time'
  source_content_hash: sha256:d5f29217f0c42ae9c5f484228c9f0c927f65fdf3b4c5ea8732f117f53667ef62
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Aird-SDK
  - AirdPro
  - MSConvert
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7333333333333333
n_claims: 15
---

# dia-swath-precursor-window-index-construction

## Summary

Construction of the precursor m/z range-MS2 group index that Aird builds for DIA/SWATH data, grouping MS2 spectra per precursor m/z SWATH window so programs can read a given m/z range with high block-read efficiency.

## When to use

Use when converting DIA/SWATH vendor MS data to the Aird format and you need metadata indexes that let analysis programs read an m/z window's MS2 spectra as a contiguous block.

## When NOT to use

- The data are DDA — use the MS1-MS2 group index instead, which reorders spectra into contiguous MS1 blocks with adjacent MS2 groups
- The acquisition mode is MRM (use chromatogramIndex), or ScanningSWATH / Ion Mobility formats that are still under development
- The output target is mzML, mgf, or mz5 — those formats converted with MSConvert do not carry Aird's precursor-window correlation indexes

## Inputs

- Vendor raw MS files acquired in DIA/SWATH mode (e.g. SCIEX WIFF, Thermo, Agilent) via AirdPro/ProteoWizard
- Converted spectrum arrays (m/z, intensity) with compression parameters and user-targeted precision
- Precursor m/z SWATH window ranges from the acquisition method

## Outputs

- Precursor m/z range-MS2 group index entries in AirdMetaData.json
- JSON index file (.json) co-located with the .aird data file, sharing the same base name
- rangeList of WindowRange entries for the DIA precursor m/z windows

## How to apply

After loading vendor spectra via the ProteoWizard SDK in AirdPro and classifying them by acquisition mode, identify DIA/SWATH MS2 spectra and assign each to its specified precursor m/z SWATH window. Group the MS2 data per window so that spectra sharing a precursor m/z range are stored contiguously, improving the block read rate when a caller requests a given m/z range. Record the window ranges (e.g. in rangeList for DIA/PRM types) and serialize the index structures together with the acquisition metadata as JSON conforming to the AirdMetaData.json schema (https://github.com/CSi-Studio/Aird-SDK/blob/1.1/AirdMetaData.json). Write the .json index file in the same directory as the .aird data file with an identical base name so AirdScanUtil can pair them. This index is one of several correlation indexes Aird builds in metadata (alongside the RT-MS index and the DDA MS1-MS2 group index) so programs can read data more quickly during analysis.

## Related tools

- **AirdPro** (GUI client that reads vendor file data via the ProteoWizard SDK, reorders DIA/SWATH spectra by precursor m/z window, and writes the JSON index next to the .aird file) — https://github.com/CSi-Studio/AirdPro/releases/
- **Aird-SDK** (Java/C#/Python SDK providing the AirdMetaData.json schema and readers that consume the precursor m/z range-MS2 group index for fast windowed retrieval) — https://github.com/CSi-Studio/Aird-SDK
- **MSConvert** (Used in the benchmark comparison to convert raw data into mzML, mgf, and mz5 formats; Aird's window index is the alternative to those conversion targets)

## Examples

```
java -cp aird-sdk.jar SampleCode  # read DIA window data via AirdSDK; convert vendor files first with AirdPro.exe
```

## Evaluation signals

- The .json index file parses against the AirdMetaData schema (https://github.com/CSi-Studio/Aird-SDK/blob/1.1/AirdMetaData.json)
- The .json and .aird files sit in the same directory with identical base names, as required for AirdScanUtil pairing
- Every MS2 spectrum in a DIA/SWATH file maps to exactly one precursor m/z SWATH window and MS2 data are grouped per window
- The AirdInfo 'type' field is DIA and rangeList contains the adjusted precursor m/z WindowRange entries
- Reading spectra for a given m/z window touches a contiguous block, i.e. block read rate is improved versus ungrouped storage

## Limitations

- At present Aird's supported formats are DDA, DIA/SWATH, and PRM; ScanningSWATH and Ion Mobility formats are under development
- When dealing with spectra, AirdPro advises processing with SWATH windows one by one to control memory
- No benchmark scripts are provided to reproduce the compression/decoding speed claims, limiting verification of end-to-end read performance

## Evidence

- [methods] Precursor m/z range-MS2 Group index: Considering DIA/SWATH, each MS2 spectrum corresponds to a specified precursor m/z range: "Precursor m/z range-MS2 Group index: Considering DIA/SWATH, each MS2 spectrum corresponds to a specified precursor m/z range"
- [methods] group MS2 data per window to improve block read rate: [evidence span withheld — anchor status: not_found]
- [methods] Aird builds additional correlation indexes in the metadata so that the program can read the data more quickly during analysis: "Aird builds additional correlation indexes in the metadata so that the program can read the data more quickly during analysis"
- [readme] The rangeList ... targeted for DIA and PRM type format: [evidence span withheld — anchor status: external_doc]
- [readme] When dealing with Spectra, we advise that you should process with SWATH Window one by one so that we can control the Memory: "When dealing with Spectra, we advise that you should process with SWATH Window one by one so that we can control the Memory"
