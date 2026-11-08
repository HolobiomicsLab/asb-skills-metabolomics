---
name: tabular-data-structuring
description: Use when you need to turn a collection of compound records returned by
  PeakForest REST endpoints into a structured tabular file, and to verify its completeness
  against a known collection size (e.g. the 96-compound demonstrator).
when_to_use_negative:
- The goal is to store raw spectral data — PeakForest is designed for metabolite annotation
  report data rather than storage of raw spectral data.
- The input data still needs cleaning or extraction — PeakForest does not include
  data clean-up or extraction tools and requires metadata to be manually entered following
  the team's SOPs.
- The compound records are already assembled in a validated tabular file — re-structuring
  would add no value.
edam_operation: http://edamontology.org/operation_2422
edam_topics:
- http://edamontology.org/topic_3366
- http://edamontology.org/topic_3391
tools:
- name: PeakForest
  role: Source REST Web service (OpenAPI v3.0) whose chemical compound library endpoints
    are queried and filtered to collect the compound records that are structured into
    the table
  repo: https://github.com/peakforest/peakforest-webapp
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1007/s11306-022-01899-3
    title: 'PeakForest: a multi-platform digital infrastructure for interoperable
      metabolite spectral data and metadata management'
  source_content_hash: sha256:e91b9862eba4eae20f626f12b08f967e4fca0ab210e32e14e29558f2a6cf9d2a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/peakforest/peakforest-webapp
  tools:
  - PeakForest
  edam_operation: http://edamontology.org/operation_2422
  edam_topics:
  - http://edamontology.org/topic_3366
  - http://edamontology.org/topic_3391
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# tabular-data-structuring

## Summary

Assemble records retrieved from the PeakForest chemical compound library REST Web services into a single flat table (e.g. tab-separated) with one row per compound, so that the MetaboHUB demonstrator dataset can be verified, shared, and reused.

## When to use

Use when you need to turn a collection of compound records returned by PeakForest REST endpoints into a structured tabular file, and to verify its completeness against a known collection size (e.g. the 96-compound demonstrator).

## When NOT to use

- The goal is to store raw spectral data — PeakForest is designed for metabolite annotation report data rather than storage of raw spectral data.
- The input data still needs cleaning or extraction — PeakForest does not include data clean-up or extraction tools and requires metadata to be manually entered following the team's SOPs.
- The compound records are already assembled in a validated tabular file — re-structuring would add no value.

## Inputs

- PeakForest REST API responses from the chemical compound library endpoints (JSON records of compounds)
- OpenAPI (v3.0) specification of the PeakForest demonstrator instance
- Filter criteria (e.g. column brand/type/name/length/diameter, AND/OR logic-gate combinations)

## Outputs

- Tab-separated table with one row per chemical compound (96 rows for the demonstrator dataset)
- Per-compound metadata columns: compound identifiers, names, structural/spectral metadata

## How to apply

Generate a Python client from the PeakForest OpenAPI (v3.0) specification that documents its REST Web services, then call the chemical compound library endpoints, using the query/filter capabilities of the Web service — column characteristics as filters and the logic-gate 'AND'/'OR' filter system — to list all compounds. Parse each returned compound record and extract the key metadata fields (compound identifiers, names, and associated structural/spectral metadata). Assemble the records into one tab-separated table with one row per compound, keeping one consistent column per metadata field. Validate the result by checking the row count matches the expected collection size (96 compounds in the demonstrator instance) and that required metadata fields are present in every row; on failure, re-query with relaxed filters rather than padding rows manually.

## Related tools

- **PeakForest** (Source REST Web service (OpenAPI v3.0) whose chemical compound library endpoints are queried and filtered to collect the compound records that are structured into the table) — https://github.com/peakforest/peakforest-webapp

## Examples

```
import requests; compounds = requests.get("https://peakforest.org/rest/compounds").(); assert len(compounds) == 96
```

## Evaluation signals

- The assembled table contains exactly 96 compound rows, matching the reported size of the demonstrator compound collection
- Required metadata fields (compound identifiers, names, associated structural/spectral metadata) are present in every row
- Row count is stable when the compound library is re-listed with the AND/OR filter system reset to no filters
- Column-specific filters (e.g. chromatography column brand, type, name, length, diameter) return subsets whose sizes are consistent with the unfiltered total
- The output is a single tab-separated file with one row per compound and no duplicate compound identifiers

## Limitations

- A minimum number of compounds and spectra must be imported before local PeakForest databases are fully functional, so smaller instances may return fewer records than expected
- PeakForest requires metadata to be manually entered in the database and does not include data clean-up or extraction tools, so record quality depends on upstream curation
- Filter availability and query ranges depend on the spectral type stored in the queried instance

## Evidence

- [other] The PeakForest demonstrator instance contains a dataset of 96 chemical compounds: [evidence span withheld — anchor status: not_found]
- [results] Filter system with AND/OR logic gates: "A filter system, integrating basic logic gates (‘AND’, ‘OR’) allows users to query the database, and a range of queries are possible depending on spectral type"
- [results] Column characteristics as Web service filters: "The column characteristics can be used as filter in Web service"
- [methods] REST documentation hosted on Tomcat: "The Web application and the REST documentation are hosted on a Tomcat server (version 7)"
- [methods] Compound import and exploitation via Web services: "Users can import compounds and spectral data manually extracted from raw data, curate information and exploit this knowledge via data export or PeakForest Web services connected tools"
