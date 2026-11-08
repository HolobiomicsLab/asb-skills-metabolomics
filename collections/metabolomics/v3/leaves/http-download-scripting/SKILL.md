---
name: http-download-scripting
description: Use when a task needs to download Metabolomics Workbench study/analysis
  data programmatically over the MW REST interface — e.g. given STUDY_ID/ANALYSIS_ID
  key-value pairs, construct and fetch REST request URLs with mwtab's mwrest module.
when_to_use_negative:
- Input is already a local mwTab or JSONized mwTab file — use mwtab's file-reading
  and conversion machinery instead of issuing REST requests
- The target data source is not the Metabolomics Workbench REST API — GenericMWURL
  validates only MW REST parameter key-value pairs
- Only one-shot manual browsing of a study is needed — the mwrest scripting layer
  adds validation and parsing overhead unnecessary for interactive retrieval
edam_operation: http://edamontology.org/operation_0226
edam_topics:
- http://edamontology.org/topic_3174
tools:
- name: mwtab 1.0.1
  role: Provides the mwrest module (GenericMWURL, MWRESTFile, read_mwrest) used to
    build, validate, and fetch MW REST interface URLs
  repo: https://github.com/MoseleyBioinformaticsLab/mwtab
- name: Python
  role: Runtime for the mwtab package, which runs under Python 3.6+
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.3390/metabo11030163
    title: The mwtab Python Library for RESTful Access and Enhanced Quality Control,
      Deposition, and Curation of the Metabolomics Workbench Data Repository
  source_content_hash: sha256:f8ed0e84dbf94ffd49c59badfca817f47bb241c5c284d176fc129115e43c2ae3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/MoseleyBioinformaticsLab/mwtab
  tools:
  - Python
  - mwtab 1.0.1
  edam_operation: http://edamontology.org/operation_0226
  edam_topics:
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# http-download-scripting

## Summary

Programmatically fetch Metabolomics Workbench data over its REST interface using the mwtab 1.0.1 mwrest module, which constructs validated REST URLs (GenericMWURL with base https://www.metabolomicsworkbench.org/rest/) and downloads responses either as mwTab-format files or raw csv/json/text payloads.

## When to use

Use when a task needs to download Metabolomics Workbench study/analysis data programmatically over the MW REST interface — e.g. given STUDY_ID/ANALYSIS_ID key-value pairs, construct and fetch REST request URLs with mwtab's mwrest module.

## When NOT to use

- Input is already a local mwTab or JSONized mwTab file — use mwtab's file-reading and conversion machinery instead of issuing REST requests
- The target data source is not the Metabolomics Workbench REST API — GenericMWURL validates only MW REST parameter key-value pairs
- Only one-shot manual browsing of a study is needed — the mwrest scripting layer adds validation and parsing overhead unnecessary for interactive retrieval

## Inputs

- REST parameter key-value dictionary (e.g. study ID, analysis ID) for the MW REST interface
- Base URL string (default https://www.metabolomicsworkbench.org/rest/)
- Metabolomics Workbench study/analysis identifiers

## Outputs

- Validated MW REST request URLs
- Parsed MWTabFile instances for responses representable in mwTab format
- MWRESTFile instances holding raw csv/json/text payloads not representable in mwTab format
- Results file containing constructed URLs and downloaded payloads

## How to apply

Load the mwtab package's mwrest components (GenericMWURL, MWRESTFile, read_mwrest) under Python 3.6+. Instantiate GenericMWURL with the base URL (default https://www.metabolomicsworkbench.org/rest/) and a dictionary of REST parameter key-value pairs (e.g. study ID and analysis ID); the class validates that the parameters are valid REST keys and combines them with the base URL to build a specific request URL. Invoke read_mwrest() to download the response: if the payload is representable in mwTab format it is parsed with the standard MWTabFile machinery, otherwise (csv/json/text) it is retained as a MWRESTFile instance of raw content. Write the constructed URLs and both the mwTab-format and non-mwTab payloads to a results file. This is the mechanism mwtab 1.0.1 introduced for programmatic interaction with MW's REST interface, complementing the library's conversion and validation features.

## Related tools

- **mwtab 1.0.1** (Provides the mwrest module (GenericMWURL, MWRESTFile, read_mwrest) used to build, validate, and fetch MW REST interface URLs) — https://github.com/MoseleyBioinformaticsLab/mwtab
- **Python** (Runtime for the mwtab package, which runs under Python 3.6+)

## Examples

```
from mwtab.mwrest import GenericMWURL, read_mwrest; url = GenericMWURL("https://www.metabolomicsworkbench.org/rest/", {"study_id": "ST000001", "analysis_id": "AN000001"}); data = read_mwrest(url)
```

## Evaluation signals

- Each constructed URL passes GenericMWURL's key-value validation of REST parameters
- Both mwTab-format and non-mwTab (csv/json/text) downloads succeed over the MW REST interface
- The mwTab-format payload parses cleanly with the standard MWTabFile machinery
- The non-mwTab payload is retained as raw csv/json/text content in a MWRESTFile instance rather than being rejected
- Constructed URLs and both downloaded payloads are present in the single results file

## Limitations

- Responses that are not representable in mwTab format (csv/json/text) can only be held as raw MWRESTFile content, not parsed into structured blocks
- Requires Python 3.6+ for the mwtab package
- Parameter keys must pass GenericMWURL validation; invalid REST key-value pairs will not produce a usable URL

## Evidence

- [results] GenericMWURL validation of REST parameters: [evidence span withheld — anchor status: not_found]
- [methods] mwrest module components and Python version: "The :mod:`mwtab` package runs under Python 3.6+."
- [abstract] REST interface enhancements in 1.0.1: "many new enhancements including methods for interacting with the MW REST interface, enhanced format validation features"
- [readme] README usage of read_files with ANALYSIS_ID: ">>> # Here we use ANALYSIS_ID of file to fetch data from URL
>>> for mwfile in mwtab.read_files("1", "2"):"
