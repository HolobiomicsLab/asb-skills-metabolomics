---
name: json-to-xml-format-conversion
description: Use when quantifying the storage-size penalty of XML vs JSON for the
  same metadata content, e.g. to justify a JSON-based index format like Aird's AirdMetaData.json
  or to benchmark format verbosity.
when_to_use_negative:
- When comparing compressed binary data formats (mzML, mz5, .aird) rather than plain-text
  metadata documents — the skill measures tag verbosity of text formats only
- When the metadata content differs between the two formats (e.g. fields added or
  dropped during conversion), since the size comparison is only meaningful for identical
  semantics
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_3077
tools:
- name: Aird
  role: The MS data format whose JSON metadata storage is the subject of the size
    comparison; Aird uses JSON for metadata to avoid XML tag redundancy
  repo: https://github.com/CSi-Studio/Aird-SDK
- name: Aird-SDK
  role: Source of the AirdMetaData.json schema (version 1.1) used as the reference
    metadata content for both serialization targets
  repo: https://github.com/CSi-Studio/Aird-SDK
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1186/s12859-021-04490-0
    title: 'Aird: a computation-oriented mass spectrometry data format enables a higher
      compression ratio and less decoding time'
  source_content_hash: sha256:a1a085849be07198e3ec48f288362894f8092c6db4900998915cbc48a64cd5c0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/CSi-Studio/Aird-SDK
  tools:
  - Aird
  - Aird-SDK
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_3077
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# json-to-xml-format-conversion

## Summary

A method for measuring the metadata storage overhead of XML versus JSON by serializing identical Aird metadata semantics into both formats and comparing on-disk sizes. It quantifies the tag-redundancy penalty of XML that motivates Aird's choice of JSON for metadata storage.

## When to use

Use when quantifying the storage-size penalty of XML vs JSON for the same metadata content, e.g. to justify a JSON-based index format like Aird's AirdMetaData.json or to benchmark format verbosity.

## When NOT to use

- When comparing compressed binary data formats (mzML, mz5, .aird) rather than plain-text metadata documents — the skill measures tag verbosity of text formats only
- When the metadata content differs between the two formats (e.g. fields added or dropped during conversion), since the size comparison is only meaningful for identical semantics

## Inputs

- Aird metadata JSON schema document (AirdMetaData.json from https://github.com/CSi-Studio/Aird-SDK/blob/1.1/AirdMetaData.json)
- XML serialization of the identical metadata semantics (same fields as AirdMetaData.json)
- Vendor MS data context files (e.g. SCIEX TOF or Thermo QE raw files whose metadata is being compared)

## Outputs

- Byte sizes of the JSON and XML metadata files
- Relative size increase of XML over JSON in percent, computed as (xml_bytes - json_bytes) / json_bytes * 100
- Comparison table (json_xml_metadata_size_comparison.csv) recording sizes and percentage increase

## How to apply

Take the Aird metadata JSON schema (AirdMetaData.json, Aird-SDK version 1.1, from the CSi-Studio/Aird-SDK repository) as the reference content so the two files carry identical information. Serialize the exact same metadata semantics into XML, preserving every field so no information is lost or added, and write both files to disk. Measure the byte size of the JSON file and the XML file, then compute the relative size increase as (xml_bytes - json_bytes) / json_bytes * 100. The rationale, per the Aird paper (Fig. 3A), is that JSON eliminates redundant XML tag information; in the paper this overhead was about 65% for SCIEX TOF files and about 1.7x for Thermo QE files. Write the two byte sizes and the computed percentage to a comparison CSV (e.g. json_xml_metadata_size_comparison.csv).

## Related tools

- **Aird** (The MS data format whose JSON metadata storage is the subject of the size comparison; Aird uses JSON for metadata to avoid XML tag redundancy) — https://github.com/CSi-Studio/Aird-SDK
- **Aird-SDK** (Source of the AirdMetaData.json schema (version 1.1) used as the reference metadata content for both serialization targets) — https://github.com/CSi-Studio/Aird-SDK

## Examples

```
python compare_metadata_size.py --json AirdMetaData.json --xml AirdMetaData.xml --out json_xml_metadata_size_comparison.csv
```

## Evaluation signals

- The computed size increase of the XML file relative to the JSON file should fall around the reported ~65% margin (per Fig. 3A); accept values within approximately ±10 percentage points
- XML must be larger than JSON for the same semantics; flag as expert_review if XML is not larger
- The JSON and XML files must carry identical information content (every AirdMetaData.json field preserved in the XML)
- Byte sizes must be measured on the actual serialized files on disk, not estimated from character counts of different encodings

## Limitations

- Reported overheads are instrument-dependent: ~65% for SCIEX TOF files versus ~1.7x for Thermo QE files, so a single percentage should not be generalized
- The comparison assumes information-equivalent serialization; any field dropped or added between JSON and XML invalidates the ratio
- The paper's claims about compression benefits focus on binary m/z data (ZDPD vs Zlib); the JSON-vs-XML result applies only to the plain-text metadata/index document

## Evidence

- [abstract] Aird uses JSON for metadata storage: "Aird uses JavaScript Object Notation for metadata storage."
- [methods] Aird JSON schema location: "we also provide a JSON Aird Schema (https://github.com/CSi-Studio/Aird-SDK/blob/1.1/AirdMetaData.json)"
- [intro] JSON readable as text, motivating the format comparison: [evidence span withheld — anchor status: not_found]
- [readme] Aird index file suffix is JSON per README: "Aird Index File Suffix: .json"
