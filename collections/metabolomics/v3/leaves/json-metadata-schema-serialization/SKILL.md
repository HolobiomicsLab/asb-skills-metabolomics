---
name: json-metadata-schema-serialization
description: Use when converting vendor MS data to Aird format and you must persist
  spectrum indexes (retention time, MS1-MS2 DDA groups, DIA/SWATH precursor m/z windows)
  as a JSON metadata file for fast random access.
when_to_use_negative:
- The data is not in Aird format and no Aird metadata/index layer is being built (e.g.
  you are working directly with vendor files or mzML/mgf/mz5 outputs from MSConvert)
- The acquisition mode is not yet supported by Aird's index layer (e.g. ScanningSWATH
  or Ion Mobility formats, which were under development at the time of the paper)
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_3520
tools:
- name: AirdPro
  role: GUI conversion client that reads vendor files via ProteoWizard SDK, reorders
    spectra by acquisition mode, and writes the .aird file plus the JSON index file
  repo: https://github.com/CSi-Studio/AirdPro/releases/
- name: Aird-SDK
  role: Java (also C#, Python) SDK for reading the JSON metadata and spectrum data
    from the Aird pair during analysis
  repo: https://github.com/CSi-Studio/Aird-SDK
- name: MSConvert
  role: ProteoWizard converter used to produce the mzML, mgf, and mz5 comparison formats
    against which the Aird metadata/index approach was evaluated
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1186/s12859-021-04490-0
    title: 'Aird: a computation-oriented mass spectrometry data format enables a higher
      compression ratio and less decoding time'
  source_content_hash: sha256:7beaf94b3c2ab9efd40be07d49368235837c2e710e43977c97e624100946918b
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
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# json-metadata-schema-serialization

## Summary

Serialize Aird's metadata — acquisition parameters plus the correlation indexes (RT-MS, MS1-MS2 group, precursor m/z range-MS2 group) — into a .json index file conforming to the AirdMetaData.json schema, stored alongside the .aird data file so programs can read MS data more quickly during analysis.

## When to use

Use when converting vendor MS data to Aird format and you must persist spectrum indexes (retention time, MS1-MS2 DDA groups, DIA/SWATH precursor m/z windows) as a JSON metadata file for fast random access.

## When NOT to use

- The data is not in Aird format and no Aird metadata/index layer is being built (e.g. you are working directly with vendor files or mzML/mgf/mz5 outputs from MSConvert)
- The acquisition mode is not yet supported by Aird's index layer (e.g. ScanningSWATH or Ion Mobility formats, which were under development at the time of the paper)

## Inputs

- Converted vendor spectrum data (.aird binary data file) from AirdPro conversion (DDA, DIA/SWATH, PRM acquisition modes)
- Spectrum start positions and retention times from the compressed binary blocks
- DIA/SWATH precursor m/z window ranges (rangeList)

## Outputs

- AirdMetaData.json index file with identical base name in the same directory as the .aird file, containing RT-MS index, MS1-MS2 group index, and precursor m/z range-MS2 group index entries per the AirdMetaData.json schema

## How to apply

After classifying spectra by acquisition mode and reordering them for better random access performance, build the correlation indexes in the metadata: for RT-MS, arrange MS data linearly by retention time and store each spectrum's retention time and start position as index entries; for DDA, group MS1 spectra into contiguous blocks with their MS2 groups adjacent and record the start positions of each MS1 group and its MS2 group; for DIA/SWATH, assign each MS2 spectrum to its specified precursor m/z SWATH window and group MS2 data per window. Serialize these index structures together with acquisition metadata as JSON conforming to the AirdMetaData.json schema (version, compressors, instruments, rangeList, indexList, totalCount, msType). Write the .json index file in the same directory as the .aird data file with an identical base name, and verify the JSON parses against the AirdMetaData schema.

## Related tools

- **AirdPro** (GUI conversion client that reads vendor files via ProteoWizard SDK, reorders spectra by acquisition mode, and writes the .aird file plus the JSON index file) — https://github.com/CSi-Studio/AirdPro/releases/
- **Aird-SDK** (Java (also C#, Python) SDK for reading the JSON metadata and spectrum data from the Aird pair during analysis) — https://github.com/CSi-Studio/Aird-SDK
- **MSConvert** (ProteoWizard converter used to produce the mzML, mgf, and mz5 comparison formats against which the Aird metadata/index approach was evaluated)

## Examples

```
pip install AirdSDK; python -c "from AirdSDK import AirdFile; aird = AirdFile('sample.aird'); idx = aird.airdInfo['indexList']"  # read JSON-persisted RT-MS/SWATH indexes
```

## Evaluation signals

- The .json index file parses as valid JSON and validates against the AirdMetaData.json schema (https://github.com/CSi-Studio/Aird-SDK/blob/1.1/AirdMetaData.json)
- The .json and .aird files share the same base name and reside in the same directory, as required for AirdScanUtil to find both
- Required AirdInfo fields (version, indexList, totalCount, msType, type, fileSize) are populated; type is one of DIA, DDA, PRM, DIA_PASEF, DDA_PASEF, MRM, MSI_MALDI, COMMON
- For DDA data, each BlockIndex records the start positions of the contiguous MS1 groups and their adjacent MS2 groups; for DIA/SWATH, MS2 entries are grouped per precursor m/z SWATH window
- Programmatic random access via Aird-SDK using the JSON index (e.g. RT-based or SWATH-window-based lookup) retrieves the expected spectra faster than sequential Zlib-only decoding

## Limitations

- At the time of the paper Aird supported only DDA, DIA/SWATH, and PRM acquisition modes; ScanningSWATH and Ion Mobility index formats were still under development
- The .json index and .aird data files must be kept together with identical base names in the same directory; separating them breaks the file-pair convention
- No benchmark scripts are provided to reproduce the reported size reduction and decoding speed claims, so index-efficiency expectations rely on the paper's reported figures

## Evidence

- [methods] Aird builds additional correlation indexes in the metadata so that the program can read the data more quickly during analysis: "Aird builds additional correlation indexes in the metadata so that the program can read the data more quickly during analysis"
- [methods] we also provide a JSON Aird Schema: "we also provide a JSON Aird Schema (https://github.com/CSi-Studio/Aird-SDK/blob/1.1/AirdMetaData.json)"
- [abstract] Aird uses JavaScript Object Notation for metadata storage: "Aird uses JavaScript Object Notation for metadata storage."
- [readme] Aird Index File and Aird Data File show be stored in the same directory with the same file: "Aird Index File and Aird Data File show be stored in the same directory with the same file."
- [methods] According to the MS file's acquisition mode, AirdPro will reorder the spectrums for better random access performance: "According to the MS file's acquisition mode, AirdPro will reorder the spectrums for better random access performance"
