---
name: aird-metadata-schema-handling
description: Use when reading, validating, generating, or comparing the metadata/index
  side of an Aird file pair (.json index + .aird data), or when deciding between JSON
  and XML for mass spec metadata storage.
when_to_use_negative:
- The task concerns compression or decoding of the binary spectral arrays (m/z via
  ZDPD, intensity) inside the .aird data file, not the metadata/index document.
- The input is a vendor file or mzML/mgf/mz5 file that has not yet been converted
  with AirdPro — no Aird JSON index exists yet.
- The analysis requires PSI CV-annotated XML metadata semantics by mandate; Aird's
  JSON index is a text-previewable alternative, not an mzML replacement.
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: Aird-SDK
  role: Java (also C#, Python) SDK for reading the Aird JSON index and spectrum data;
    hosts the AirdMetaData.json schema used to validate the metadata document.
  repo: https://github.com/CSi-Studio/Aird-SDK
- name: AirdPro
  role: GUI conversion client that generates the .aird + .json index file pair from
    vendor files; the JSON index it emits follows the AirdInfo schema.
  repo: https://github.com/CSi-Studio/AirdPro/releases/
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1186/s12859-021-04490-0
    title: 'Aird: a computation-oriented mass spectrometry data format enables a higher
      compression ratio and less decoding time'
  source_content_hash: sha256:1e1e94e72465956330ddee76ff65b24c2c895593e7dd54f4b4bd6f5f72b9369a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/CSi-Studio/Aird-SDK
  tools:
  - Aird
  - Aird-SDK
  - AirdPro
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6923076923076923
n_claims: 13
---

# aird-metadata-schema-handling

## Summary

Handles the JSON metadata (index) component of Aird mass spectrometry files using the AirdMetaData.json schema — Aird deliberately stores metadata as JSON rather than XML so the index is smaller (about 65% less text for SCIEX TOF files) and human-previewable.

## When to use

Use when reading, validating, generating, or comparing the metadata/index side of an Aird file pair (.json index + .aird data), or when deciding between JSON and XML for mass spec metadata storage.

## When NOT to use

- The task concerns compression or decoding of the binary spectral arrays (m/z via ZDPD, intensity) inside the .aird data file, not the metadata/index document.
- The input is a vendor file or mzML/mgf/mz5 file that has not yet been converted with AirdPro — no Aird JSON index exists yet.
- The analysis requires PSI CV-annotated XML metadata semantics by mandate; Aird's JSON index is a text-previewable alternative, not an mzML replacement.

## Inputs

- Aird index file (*.json) following the AirdMetaData.json / AirdInfo schema
- Aird data file (*.aird) paired with the index file
- AirdMetaData.json schema document (Aird-SDK, version 1.1)
- Equivalent metadata serialized as XML (for JSON-vs-XML size comparison)

## Outputs

- Validated/parsed AirdInfo metadata object (version, compressors, indexList, type, totalCount, etc.)
- BlockIndex entries and correlation indexes (RT-MS, MS1-MS2 Group, Precursor m/z range-MS2 Group) used for random access
- Measured byte sizes and relative size increase of XML vs JSON metadata, e.g. json_xml_metadata_size_comparison.csv

## How to apply

Treat an Aird file as a pair: the index file with suffix .json and the data file with suffix .aird, stored in the same directory under the same base name. Parse the .json index against the AirdMetaData.json schema (v1.1 from the Aird-SDK repository), which defines the AirdInfo structure: required fields include version, versionCode, engine (0: Row Compression, 1: Column Compression), compressors, instruments, indexList, type (DIA, DDA, PRM, DIA_PASEF, DDA_PASEF, MRM, MSI_MALDI, COMMON), fileSize, totalCount, and msType; for version code >= 7, compressed binary index data is located via indexStartPtr/indexEndPtr into the .aird file. When authoring or comparing metadata representations, serialize identical semantics in JSON and XML and measure byte sizes — the paper reports XML is about 65% larger for SCIEX TOF files and 1.7x larger for Thermo QE files because JSON eliminates redundant tag information. Rationale: JSON keeps the index compact and text-previewable, supporting Aird's computation-oriented design.

## Related tools

- **Aird-SDK** (Java (also C#, Python) SDK for reading the Aird JSON index and spectrum data; hosts the AirdMetaData.json schema used to validate the metadata document.) — https://github.com/CSi-Studio/Aird-SDK
- **AirdPro** (GUI conversion client that generates the .aird + .json index file pair from vendor files; the JSON index it emits follows the AirdInfo schema.) — https://github.com/CSi-Studio/AirdPro/releases/

## Examples

```
pip install AirdSDK
```

## Evaluation signals

- The .json index parses as valid JSON and contains the required AirdInfo fields (version, versionCode, engine, compressors, instruments, indexList, type, fileSize, totalCount, msType).
- The .json index and .aird data file share the same base name and live in the same directory, as required so AirdScanUtil can scan the pair.
- The fileSize field recorded in the metadata matches the on-disk size of the .aird and .json files.
- For a JSON-vs-XML size comparison on identical metadata semantics, the XML size increase falls around the reported ~65% for SCIEX TOF content (accept within roughly ±10 percentage points); XML not being larger than JSON indicates an error.
- For version code >= 7, indexStartPtr and indexEndPtr delimit a region of compressed binary index data within the .aird file.

## Limitations

- The schema is versioned and evolving (e.g. version code >= 7 introduced indexStartPtr/indexEndPtr; AirdPro V5/V6 add MRM and ion-mobility improvements), so validation should pin the schema version such as AirdMetaData.json at the 1.1 tag.
- Aird (and its metadata index semantics) only supports certain acquisition modes — the paper states DDA, DIA/SWATH, and PRM at publication time, while ScanningSWATH and Ion Mobility formats were under development.
- No dedicated schema-validation tooling or benchmark scripts are provided in the paper; the JSON-vs-XML comparison must be re-derived by the practitioner.

## Evidence

- [abstract] Aird uses JSON for metadata storage: "Aird uses JavaScript Object Notation for metadata storage."
- [methods] JSON Aird Schema provided in Aird-SDK: "we also provide a JSON Aird Schema (https://github.com/CSi-Studio/Aird-SDK/blob/1.1/AirdMetaData.json)"
- [readme] Aird file pair naming convention: "Aird Index File Suffix: .json"
- [readme] AirdInfo required fields: [evidence span withheld — anchor status: external_doc]
