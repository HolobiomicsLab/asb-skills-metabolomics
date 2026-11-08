---
name: web-api-record-count-aggregation
description: Use when a task asks to reproduce or verify repository-level statistics
  — e.g. total spectra, per-instrument-type counts, or contributing-group tallies
  — from a public database with a REST API or web endpoint.
when_to_use_negative:
- When the goal is similarity-based spectral search or compound identification scoring
  (weighted cosine correlation), not record enumeration and counting
- When analyzing individual peak lists or record file contents (MassBank record format
  with accession IDs and peaks) rather than aggregate repository statistics
- When the target database lacks a queryable API or record-count endpoint, so counts
  cannot be enumerated programmatically
edam_operation: http://edamontology.org/operation_0224
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: MassBank3
  role: Provides the REST API and Swagger UI used to enumerate and count MassBank
    records per data group and contributor
  repo: https://github.com/MassBank/MassBank3
- name: MassBank public website
  role: Live instance at https://massbank.eu (mirror at https://msbi.ipb-halle.de/MassBank)
    queried for record counts and group metadata
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1002/jms.1777
    title: 'MassBank: a public repository for sharing mass spectral data for life
      sciences'
  source_content_hash: sha256:a78612baf52880b6d4b83720145614700f5fcf3f072350bf17627f4ba0f9ad73
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/MassBank/MassBank3
  tools:
  - MassBank public website
  - MassBank3
  - MassBank3 web app
  edam_operation: http://edamontology.org/operation_0224
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4166666666666667
n_claims: 12
---

# web-api-record-count-aggregation

## Summary

Enumerate records from a public scientific repository's REST API (here MassBank at massbank.eu), count them per data group (EI-MS, FAB-MS, ESI-MSn, other-MS, ESI-MS2), contributor group, and compound set, and tabulate the breakdown into a CSV for validation against published statistics.

## When to use

Use when a task asks to reproduce or verify repository-level statistics — e.g. total spectra, per-instrument-type counts, or contributing-group tallies — from a public database with a REST API or web endpoint.

## When NOT to use

- When the goal is similarity-based spectral search or compound identification scoring (weighted cosine correlation), not record enumeration and counting
- When analyzing individual peak lists or record file contents (MassBank record format with accession IDs and peaks) rather than aggregate repository statistics
- When the target database lacks a queryable API or record-count endpoint, so counts cannot be enumerated programmatically

## Inputs

- MassBank REST API endpoints at https://massbank.eu (Swagger UI at https://massbank.eu/MassBank-api)
- Record count endpoint (e.g. /MassBank-api/records/count)
- Group/contributor metadata returned by the API
- Reference values from the article's Table 2 and abstract (605 EI-MS, 137 FAB-MS, 9276 ESI-MSn, 3045 ESI-MS2, 11545 EI-MS of volatiles)

## Outputs

- massbank_statistics.csv with one row per data group and columns for spectra and compound counts
- Aggregated per-research-group and per-institution record counts

## How to apply

Query the public MassBank service (https://massbank.eu, with a parallel instance at https://msbi.ipb-halle.de/MassBank) via its REST API to enumerate records and count spectra per data group: EI-MS (605), FAB-MS (137), ESI-MSn (9276), other-MS (834), and ESI-MS2 (3045). Aggregate record counts per research group using the group metadata returned by the API — the January 2010 snapshot covers 16 contributing groups in this distributed database where each group serves data from its own MassBank data servers. Compute totals (10294 spectra of 2337 authentic compounds, 679 synthetic drugs, 10286 volatile compounds) and write one row per data group with spectra and compound columns to massbank_statistics.csv. Validate by cross-checking each tabulated count against the article's Table 2 and abstract reference values; accept the output only when all counts match.

## Related tools

- **MassBank3** (Provides the REST API and Swagger UI used to enumerate and count MassBank records per data group and contributor) — https://github.com/MassBank/MassBank3
- **MassBank public website** (Live instance at https://massbank.eu (mirror at https://msbi.ipb-halle.de/MassBank) queried for record counts and group metadata)

## Examples

```
curl https://massbank.eu/MassBank-api/records/count
```

## Evaluation signals

- Retrieved counts match the published reference values exactly: 605 EI-MS, 137 FAB-MS, 9276 ESI-MSn, 3045 ESI-MS2, 834 other-MS, 11545 EI-MS of volatiles
- Total spectra reconciles to 10294 mass spectra (plus 3045 ESI-MS2 of 679 synthetic drugs) across 16 research groups
- Unique compound counts match: 2337 authentic compounds, 679 synthetic drugs, 10286 volatile compounds
- massbank_statistics.csv contains one row per data group with non-null spectra and compound columns
- Record count from the API endpoint (e.g. /MassBank-api/records/count) equals the sum of per-group tabulated counts

## Limitations

- Live API counts reflect the current database, not the January 2010 snapshot, so historical figures may not be reproducible without an archived or versioned data release
- MassBank is a distributed database — each research group serves data from its own servers — so aggregation depends on complete group metadata from the API
- Data merging of ESI-MS2 records under different collision-induced dissociation conditions means identical compounds can appear as multiple records, complicating unique-compound counting

## Evidence

- [abstract] abstract: "The database contains 605 electron-ionization mass spectrometry (EI-MS), 137 fast atom bombardment MS and 9276 electrospray ionization (ESI)-MS(n) data of 2337 authentic compounds"
- [readme] readme: [evidence span withheld — anchor status: not_found]
