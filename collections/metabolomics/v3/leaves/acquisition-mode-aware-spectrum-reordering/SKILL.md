---
name: acquisition-mode-aware-spectrum-reordering
description: Use when converting vendor MS files (SCIEX wiff, Thermo, Agilent) into
  a computation-oriented format and you need fast random access by retention time,
  DDA MS1-MS2 grouping, or DIA/SWATH precursor m/z windows.
when_to_use_negative:
- Input data uses ScanningSWATH or Ion Mobility acquisition, which are formats under
  development and not supported by the Aird indexing described in this paper
- Acquisition mode is something other than DDA, DIA/SWATH, or PRM (the Aird supporting
  formats reported in this paper), so no mode-specific group index can be constructed
- You only need lossless archival storage of vendor files without fast computational
  random access — vendor files are already ideal for long-term storage due to their
  high compression rate
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_0121
tools:
- name: AirdPro
  role: GUI client that reads vendor file data via the ProteoWizard SDK, reorders
    spectrums by acquisition mode, and builds the correlation indexes into the JSON
    metadata during conversion
  repo: https://github.com/CSi-Studio/AirdPro/releases/
- name: Aird-SDK
  role: Java SDK for programs to read the reordered spectra and the JSON-built correlation
    indexes (RT-MS, MS1-MS2 group, precursor m/z range-MS2 group) during analysis
  repo: https://github.com/CSi-Studio/Aird-SDK
- name: MSConvert
  role: Used to convert raw vendor data into mzML, mgf, and mz5 formats for comparison
    benchmarks against the Aird indexed format
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1186/s12859-021-04490-0
    title: 'Aird: a computation-oriented mass spectrometry data format enables a higher
      compression ratio and less decoding time'
  source_content_hash: sha256:438144a1a9d1e0a1c4a7db9af07bbe85841f9a0f9f444ee6de4d4f7e241706d3
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Aird-SDK
  - AirdPro
  - MSConvert
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 16
---

# acquisition-mode-aware-spectrum-reordering

## Summary

AirdPro reorders spectra and builds acquisition-mode-specific correlation indexes (RT-MS, MS1-MS2 group for DDA, precursor m/z range-MS2 group for DIA/SWATH) so downstream programs can random-access MS data more quickly during analysis.

## When to use

Use when converting vendor MS files (SCIEX wiff, Thermo, Agilent) into a computation-oriented format and you need fast random access by retention time, DDA MS1-MS2 grouping, or DIA/SWATH precursor m/z windows.

## When NOT to use

- Input data uses ScanningSWATH or Ion Mobility acquisition, which are formats under development and not supported by the Aird indexing described in this paper
- Acquisition mode is something other than DDA, DIA/SWATH, or PRM (the Aird supporting formats reported in this paper), so no mode-specific group index can be constructed
- You only need lossless archival storage of vendor files without fast computational random access — vendor files are already ideal for long-term storage due to their high compression rate

## Inputs

- Vendor raw MS files (SCIEX, Thermo, Agilent instruments)
- Vendor spectrum data converted via MSConvert/ProteoWizard
- Acquisition mode classification of spectra (DDA, DIA/SWATH, PRM)

## Outputs

- .aird data file with reordered spectra
- AirdMetaData.json index file (RT-MS index, MS1-MS2 group index, precursor m/z range-MS2 group index)
- JSON index validated against the AirdMetaData.json schema

## How to apply

Read vendor file data using the integrated ProteoWizard SDK in AirdPro as the first step of conversion, then classify spectra by acquisition mode (DDA, DIA/SWATH, PRM). Reorder the spectrums according to the MS file's acquisition mode for better random access performance: for DDA, group MS1 spectra into contiguous MS1 blocks with their corresponding MS2 groups adjacent and record the start positions of each MS1 group and its MS2 group; for DIA/SWATH, assign each MS2 spectrum to its specified precursor m/z SWATH window and group MS2 data per window to improve block read rate for a given m/z range. Independently build the RT-MS index by arranging MS data linearly by retention time and storing each spectrum's retention time and start position as index entries. Serialize the index structures together with acquisition metadata as JSON conforming to the AirdMetaData.json schema, writing the .json index file in the same directory as the .aird data file with an identical base name.

## Related tools

- **AirdPro** (GUI client that reads vendor file data via the ProteoWizard SDK, reorders spectrums by acquisition mode, and builds the correlation indexes into the JSON metadata during conversion) — https://github.com/CSi-Studio/AirdPro/releases/
- **Aird-SDK** (Java SDK for programs to read the reordered spectra and the JSON-built correlation indexes (RT-MS, MS1-MS2 group, precursor m/z range-MS2 group) during analysis) — https://github.com/CSi-Studio/Aird-SDK
- **MSConvert** (Used to convert raw vendor data into mzML, mgf, and mz5 formats for comparison benchmarks against the Aird indexed format)

## Evaluation signals

- The .json index file is written in the same directory as the .aird data file with an identical base name, so both files can be scanned together
- The JSON metadata parses against the AirdMetaData.json schema (https://github.com/CSi-Studio/Aird-SDK/blob/1.1/AirdMetaData.json)
- For DDA data, spectra are grouped into contiguous MS1 blocks with corresponding MS2 groups adjacent, with start positions of each MS1 group and its MS2 group recorded
- For DIA/SWATH data, every MS2 spectrum is assigned to a specified precursor m/z SWATH window and grouped per window, improving block read rate for a given m/z range
- The RT-MS index entries contain each spectrum's retention time and start position, arranged linearly by retention time

## Limitations

- Aird currently supports only DDA, DIA/SWATH, and PRM acquisition modes in this paper
- ScanningSWATH and Ion Mobility formats are under development and not yet supported
- When dealing with spectra, the README advises processing SWATH windows one by one so that memory can be controlled

## Evidence

- [methods] Reorder spectrums based on acquisition mode for better random access performance: "According to the MS file's acquisition mode, AirdPro will reorder the spectrums for better random access performance"
- [methods] Build correlation indexes in metadata per acquisition mode: "Aird builds additional correlation indexes in the metadata so that the program can read the data more quickly during analysis"
- [methods] Precursor m/z range-MS2 Group index definition: "Precursor m/z range-MS2 Group index: Considering DIA/SWATH, each MS2 spectrum corresponds to a specified precursor m/z range"
- [abstract] JSON metadata storage: "Aird uses JavaScript Object Notation for metadata storage."
- [readme] Index/data file co-location convention: "Aird Index File and Aird Data File show be stored in the same directory with the same file."
- [results] Supported acquisition modes limitation: "At present, Aird's supporting formats are DDA, DIA/SWATH, PRM."
