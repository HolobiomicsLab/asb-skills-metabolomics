---
name: file-size-comparison-benchmarking
description: Use when comparing storage efficiency of two text serialization formats
  (e.g. JSON vs XML) holding the same structured metadata, or when benchmarking a
  format's size claim such as Aird's JSON-based metadata versus XML equivalents.
when_to_use_negative:
- Comparing compressed binary spectra payloads (m/z or intensity arrays) across formats
  — that requires the Aird ZDPD vs Zlib compression benchmark, not a text serialization
  size test
- Input files do not contain equivalent semantic content — a byte-size ratio is meaningless
  if fields were dropped or added during conversion
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: Aird
  role: The MS data format whose JSON metadata storage is the subject of the size
    comparison
  repo: https://github.com/CSi-Studio/Aird-SDK
- name: Aird-SDK
  role: Source of the AirdMetaData.json schema (version 1.1) used as the reference
    JSON metadata document
  repo: https://github.com/CSi-Studio/Aird-SDK
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1186/s12859-021-04490-0
    title: 'Aird: a computation-oriented mass spectrometry data format enables a higher
      compression ratio and less decoding time'
  source_content_hash: sha256:353065be1cf58d2885700ecd760dd6b2f8d23e14e258b010798bfb7adfa36de9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/CSi-Studio/Aird-SDK
  tools:
  - Aird
  - Aird-SDK
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# file-size-comparison-benchmarking

## Summary

Quantifies the storage overhead of verbose text formats by serializing identical semantic content in two formats and comparing on-disk byte sizes, e.g. Aird metadata stored as JSON versus XML where XML tag redundancy inflates size by ~65%.

## When to use

Use when comparing storage efficiency of two text serialization formats (e.g. JSON vs XML) holding the same structured metadata, or when benchmarking a format's size claim such as Aird's JSON-based metadata versus XML equivalents.

## When NOT to use

- Comparing compressed binary spectra payloads (m/z or intensity arrays) across formats — that requires the Aird ZDPD vs Zlib compression benchmark, not a text serialization size test
- Input files do not contain equivalent semantic content — a byte-size ratio is meaningless if fields were dropped or added during conversion

## Inputs

- AirdMetaData.json schema document (Aird-SDK repo, version 1.1, https://github.com/CSi-Studio/Aird-SDK/blob/1.1/AirdMetaData.json)
- XML serialization of the same metadata semantics with all fields preserved

## Outputs

- json_xml_metadata_size_comparison.csv containing JSON bytes, XML bytes, and the relative size-increase percentage

## How to apply

Start from the reference JSON document — for the Aird case, the AirdMetaData.json schema at version 1.1 of the Aird-SDK repository. Serialize the identical metadata semantics into the competing format (XML), preserving every field so both files carry the same information content. Measure the byte size of each file on disk, then compute the relative size increase as (xml_bytes - json_bytes) / json_bytes * 100. Write sizes and the percentage to a results CSV (e.g. json_xml_metadata_size_comparison.csv). The rationale is that JSON, which Aird uses for metadata storage, eliminates redundant XML tag information, so XML should be substantially larger — roughly 65% larger for SCIEX TOF files and about 1.7x for Thermo QE files.

## Related tools

- **Aird** (The MS data format whose JSON metadata storage is the subject of the size comparison) — https://github.com/CSi-Studio/Aird-SDK
- **Aird-SDK** (Source of the AirdMetaData.json schema (version 1.1) used as the reference JSON metadata document) — https://github.com/CSi-Studio/Aird-SDK

## Examples

```
import os; j=os.path.getsize('AirdMetaData.json'); x=os.path.getsize('AirdMetaData.xml'); print((x-j)/j*100)
```

## Evaluation signals

- XML file is strictly larger than the JSON file on disk; flag as expert_review if XML is not larger than JSON
- Computed relative size increase falls near the reported ~65% margin for SCIEX TOF files, accepting values within approximately ±10 percentage points
- For Thermo QE files the ratio should be about 1.7x, consistent with the paper's Fig. 3A finding that JSON uses much less text than XML for the same semantics
- Field-by-field diff confirms the XML serialization preserves every field of the AirdMetaData.json schema (both files carry identical information content)
- Both measured byte counts and the percentage formula result are present in json_xml_metadata_size_comparison.csv

## Limitations

- The reported ~65% / 1.7x figures are file-type dependent (SCIEX TOF vs Thermo QE); results for other instrument metadata will vary and the ±10 percentage point tolerance is calibrated only to the reported figures
- Aird's claimed 49% m/z size reduction and 3x faster decoding versus Zlib lack benchmark scripts, so analogous size claims for binary data cannot be reproduced from published artifacts
- The comparison holds only when the two serializations carry the same information content; any lossy field mapping invalidates the ratio

## Evidence

- [abstract] Aird uses JavaScript Object Notation for metadata storage.: "Aird uses JavaScript Object Notation for metadata storage."
- [methods] we also provide a JSON Aird Schema (https://github.com/CSi-Studio/Aird-SDK/blob/1.1/AirdMetaData.json): "we also provide a JSON Aird Schema (https://github.com/CSi-Studio/Aird-SDK/blob/1.1/AirdMetaData.json)"
- [other] the computed size increase of the XML file relative to the JSON file should fall around the reported ~65% margin (per Fig. 3A, JSON uses much less text than XML for the same semantics): [evidence span withheld — anchor status: not_found]
- [results] Storing the same Aird metadata content in XML instead of JSON increases file size by about 65% for SCIEX TOF files and 1.7 times for Thermo QE files, because JSON eliminates redundant XML tag information.: [evidence span withheld — anchor status: not_found]
