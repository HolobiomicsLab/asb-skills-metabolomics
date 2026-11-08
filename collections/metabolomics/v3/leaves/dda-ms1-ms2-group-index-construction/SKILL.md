---
name: dda-ms1-ms2-group-index-construction
description: Use when converting DDA vendor MS data to the Aird format and you need
  to construct the metadata correlation index that groups MS1 spectra into contiguous
  blocks with their associated MS2 spectra for fast random access.
when_to_use_negative:
- Data were acquired in DIA/SWATH mode — build the precursor m/z range-MS2 group index
  instead of the MS1-MS2 group index
- The acquisition mode is PRM or a format Aird does not yet support (e.g. ScanningSWATH
  or Ion Mobility are under development)
- The goal is to read an existing Aird file rather than produce the index — use AirdSDK's
  reader instead of rebuilding the index
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: AirdPro
  role: C# GUI client that reads vendor files via ProteoWizard, reorders DDA spectra
    into MS1 blocks with adjacent MS2 groups, and writes the .aird file and JSON index
  repo: https://github.com/CSi-Studio/AirdPro/releases/
- name: Aird-SDK
  role: Java SDK providing the AirdMetaData.json schema against which the serialized
    index structures are validated
  repo: https://github.com/CSi-Studio/Aird-SDK
- name: MSConvert
  role: ProteoWizard converter used for converting raw data into mzML, mgf, and mz5
    comparison formats
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1186/s12859-021-04490-0
    title: 'Aird: a computation-oriented mass spectrometry data format enables a higher
      compression ratio and less decoding time'
  source_content_hash: sha256:236abac40d0240504615b56820c6af3ca9b1f62db0ea13f84e08554144448503
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
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 15
---

# dda-ms1-ms2-group-index-construction

## Summary

Build the MS1-MS2 group index that Aird stores in its JSON metadata for DDA data, reordering spectra so each MS1 block sits adjacent to its corresponding MS2 group with recorded start positions, enabling faster block reads during analysis.

## When to use

Use when converting DDA vendor MS data to the Aird format and you need to construct the metadata correlation index that groups MS1 spectra into contiguous blocks with their associated MS2 spectra for fast random access.

## When NOT to use

- Data were acquired in DIA/SWATH mode — build the precursor m/z range-MS2 group index instead of the MS1-MS2 group index
- The acquisition mode is PRM or a format Aird does not yet support (e.g. ScanningSWATH or Ion Mobility are under development)
- The goal is to read an existing Aird file rather than produce the index — use AirdSDK's reader instead of rebuilding the index

## Inputs

- Vendor raw MS files acquired in DDA mode (converted via AirdPro using ProteoWizard pwiz_bindings_cli.dll)
- Spectra classified by acquisition mode with m/z and intensity arrays already compressed per the chosen precision

## Outputs

- AirdMetaData.json index file containing the MS1-MS2 group index (start positions of each MS1 block and its MS2 group) plus RT-MS index entries
- Paired .aird data file with reordered spectra, stored in the same directory with the same base name as the .json index

## How to apply

After loading vendor spectrum data via the ProteoWizard SDK in AirdPro and determining the acquisition mode is DDA, reorder the spectra so MS1 data are grouped into contiguous MS1 blocks with their corresponding MS2 groups adjacent (each MS1 spectrum corresponds to zero or multiple MS2 data). Record the start position of each MS1 group and of its MS2 group as index entries, alongside the RT-MS index entries that store each spectrum's retention time and start position. Serialize the index structures together with acquisition metadata as JSON conforming to the AirdMetaData.json schema, writing the .json index file in the same directory as the .aird data file with an identical base name. The rationale is that contiguous blocks let analysis programs read an MS1 spectrum and its fragmenting MS2 spectra with minimal file seeks.

## Related tools

- **AirdPro** (C# GUI client that reads vendor files via ProteoWizard, reorders DDA spectra into MS1 blocks with adjacent MS2 groups, and writes the .aird file and JSON index) — https://github.com/CSi-Studio/AirdPro/releases/
- **Aird-SDK** (Java SDK providing the AirdMetaData.json schema against which the serialized index structures are validated) — https://github.com/CSi-Studio/Aird-SDK
- **MSConvert** (ProteoWizard converter used for converting raw data into mzML, mgf, and mz5 comparison formats)

## Evaluation signals

- The .json index file parses against the AirdMetaData schema (https://github.com/CSi-Studio/Aird-SDK/blob/1.1/AirdMetaData.json)
- The .json and .aird files share the same base name and reside in the same directory, so AirdScanUtil can pair them
- Each MS1 group's recorded start position matches the byte offset of a contiguous MS1 block whose adjacent MS2 group start position is also recorded
- Reordered spectrum sequence preserves each MS1 spectrum followed by its zero or multiple corresponding MS2 spectra

## Limitations

- Aird currently supports only DDA, DIA/SWATH, and PRM acquisition modes; ScanningSWATH and Ion Mobility formats are under development
- No benchmark scripts are provided to independently reproduce compression or decoding-speed claims, so index benefit must be verified empirically by the user
- When processing DIA/SWATH spectra via AirdSDK, the README advises processing SWATH windows one by one to control memory (analogous care applies to block reads)

## Evidence

- [methods] MS1-MS2 Group index: "MS1-MS2 Group index: Considering DDA, each MS1 spectrum will correspond to zero or multiple MS2 data"
- [methods] Reorder spectrums based on acquisition mode for better random access performance: "According to the MS file's acquisition mode, AirdPro will reorder the spectrums for better random access performance"
- [methods] Aird builds additional correlation indexes in the metadata so that the program can read the data more quickly during analysis: "Aird builds additional correlation indexes in the metadata so that the program can read the data more quickly during analysis"
- [readme] Aird Index File and Aird Data File should be stored in the same directory with the same file name: "Aird Index File and Aird Data File show be stored in the same directory with the same file name but with different suffix"
