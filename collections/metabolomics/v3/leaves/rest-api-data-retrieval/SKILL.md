---
name: rest-api-data-retrieval
description: Use when a scientific database exposes REST Web services with an OpenAPI
  (v3.0) spec and you need programmatic, filterable retrieval of records (compounds,
  spectra) into a tabular dataset.
when_to_use_negative:
- You need raw spectral data storage or retrieval — PeakForest is designed for metabolite
  annotation report data rather than storage of raw spectral data
- The data need cleaning or extraction before retrieval — PeakForest does not include
  data clean-up or extraction tools and requires metadata to be manually entered/curated
  upstream
- A one-off interactive browse through the Web interface suffices; scripted REST retrieval
  adds unnecessary setup
edam_operation: http://edamontology.org/operation_2422
tools:
- name: PeakForest
  role: REST Web service whose chemical compound library and spectra library endpoints
    are queried; OpenAPI (v3.0) spec enables Python client generation
  repo: https://github.com/peakforest
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1007/s11306-022-01899-3
    title: 'PeakForest: a multi-platform digital infrastructure for interoperable
      metabolite spectral data and metadata management'
  source_content_hash: sha256:369c060168c8d76b1b08aecd8596a46087d8938c81b817476b34df299adecaa2
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - PeakForest
  edam_operation: http://edamontology.org/operation_2422
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# rest-api-data-retrieval

## Summary

Retrieve structured records (e.g. the 96-compound PeakForest demonstrator collection) from a scientific Web service over REST, using its OpenAPI (v3.0) specification to generate a typed client and its built-in filter system (column characteristics, AND/OR logic gates) to scope queries.

## When to use

Use when a scientific database exposes REST Web services with an OpenAPI (v3.0) spec and you need programmatic, filterable retrieval of records (compounds, spectra) into a tabular dataset.

## When NOT to use

- You need raw spectral data storage or retrieval — PeakForest is designed for metabolite annotation report data rather than storage of raw spectral data
- The data need cleaning or extraction before retrieval — PeakForest does not include data clean-up or extraction tools and requires metadata to be manually entered/curated upstream
- A one-off interactive browse through the Web interface suffices; scripted REST retrieval adds unnecessary setup

## Inputs

- PeakForest REST Web service endpoint with OpenAPI (v3.0) specification
- Filter criteria (column characteristics for LC/GC/ion chromatography, AND/OR logic-gate query terms)
- List of metadata fields to extract (compound identifiers, names, structural/spectral metadata)

## Outputs

- Tab-separated table with one row per compound record
- Parsed compound/spectra metadata records (identifiers, names, structural and spectral metadata)

## How to apply

Access the service's REST endpoints, using the OpenAPI (v3.0) specification to generate a client in a common language such as Python rather than hand-crafting HTTP calls. Apply the Web service's query/filter capabilities — e.g. chromatography column characteristics (brand, type, name, length, diameter for LC/GC/ion chromatography data) and the logic-gate 'AND'/'OR' filter system — to list or narrow the target record set. Parse each returned record and extract key metadata fields (identifiers, names, structural/spectral metadata), then assemble them into a single table with one row per record. Validate by checking row counts against a known reference size (e.g. exactly 96 compound rows for the PeakForest demonstrator) and confirming required metadata fields are present in every row.

## Related tools

- **PeakForest** (REST Web service whose chemical compound library and spectra library endpoints are queried; OpenAPI (v3.0) spec enables Python client generation) — https://github.com/peakforest

## Examples

```
openapi-generator-cli generate -i peakforest-openapi-v3.json -g python -o pf_client; then call the generated compound-library endpoints to list all compounds
```

## Evaluation signals

- The assembled table contains exactly 96 compound rows, matching the reported size of the PeakForest demonstrator compound collection
- Every row has the required metadata fields populated (compound identifiers, names, associated structural/spectral metadata)
- Generated client is produced successfully from the OpenAPI (v3.0) specification (e.g. a Python client compiles/imports)
- Applied filters behave per the documented logic-gate system: AND/OR combinations return the expected narrowed record sets
- Extracted fields are consistent with the service's data model (column characteristics such as brand, type, name, length, diameter where chromatography data is queried)

## Limitations

- A minimum number of compounds and spectra must be imported before local PeakForest databases are fully functional, so queries against a sparsely populated instance may return incomplete data
- Metadata must be manually entered/curated by users; the service does not include data clean-up or extraction tools
- Query scope depends on spectral type — the range of possible queries varies depending on spectral type (MS vs NMR)

## Evidence

- [results] REST filters on column characteristics: "The column characteristics can be used as filter in Web service"
- [results] AND/OR logic-gate filter system: "A filter system, integrating basic logic gates (‘AND’, ‘OR’) allows users to query the database, and a range of queries are possible depending on spectral type"
- [methods] Exploitation of imported data via Web services: "Users can import compounds and spectral data manually extracted from raw data, curate information and exploit this knowledge via data export or PeakForest Web services connected tools"
- [abstract] REST/automatic programming interfaces: "We developed the PeakForest infrastructure, an open-source Java tool with automatic programming interfaces that can be deployed locally"
- [readme] Compound library service endpoint: "chemical compound library manager"
