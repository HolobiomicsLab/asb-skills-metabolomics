---
name: mibig-json-data-parsing
description: Use when a study needs MIBiG BGC entries as a structured reference set
  — e.g. assessing novelty of candidate BGCs, homology searches against known clusters,
  or pathway design inputs.
when_to_use_negative:
- When the goal is to deposit a new BGC — use the MIBiG submission workflow (request
  accession number, then extended entry form) instead of parsing existing entries
- When candidate BGCs are only needed at scale without known-function annotation —
  IMG-ABC or antiSMASH-DB store many thousands of candidate BGCs and may be more appropriate
edam_operation: http://edamontology.org/operation_0226
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: MIBiG Repository
  role: Source of the JSON-formatted BGC entries parsed by this skill
  repo: https://mibig.secondarymetabolites.org/
- name: mibig-api
  role: REST-like web API handling access to the underlying PostgreSQL database that
    backs the parsed data
  repo: https://github.com/mibig-secmet/mibig-api/
- name: JSON schema
  role: Validation technology used to check MIBiG entry structure and dependency rules
  repo: https://json-schema.org
- name: ClusterCAD
  role: Downstream tool that sources BGC data from parsed MIBiG entries for pathway
    design
- name: antiSMASH 5
  role: Customised module sideloads the MIBiG JSON annotation file to generate individual
    BGC pages
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/nar/gkz882
    title: 'MIBiG 2.0: a repository for biosynthetic gene clusters of known function'
  source_content_hash: sha256:00413a10d10f0f6a1f3f8dbcc4da9072235cc18ffb34c991ee05ee2e9606c116
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ClusterCAD
  - GitHub
  - JSON schema
  - MIBiG Repository
  - PostgreSQL
  - antiSMASH 5
  - mibig-api
  edam_operation: http://edamontology.org/operation_0226
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9333333333333333
n_claims: 15
---

# mibig-json-data-parsing

## Summary

Download and parse MIBiG 2.0 JSON-formatted BGC entries, extracting biosynthetic class, taxonomic identifier, and product annotation fields so the curated reference set can serve as a homology baseline in downstream analyses.

## When to use

Use when a study needs MIBiG BGC entries as a structured reference set — e.g. assessing novelty of candidate BGCs, homology searches against known clusters, or pathway design inputs.

## When NOT to use

- When the goal is to deposit a new BGC — use the MIBiG submission workflow (request accession number, then extended entry form) instead of parsing existing entries
- When candidate BGCs are only needed at scale without known-function annotation — IMG-ABC or antiSMASH-DB store many thousands of candidate BGCs and may be more appropriate

## Inputs

- MIBiG 2.0 JSON-formatted BGC entries (from https://github.com/mibig-secmet or the mibig-api REST endpoint)
- MIBiG JSON schema definition (https://json-schema.org) for validation
- Query BGCs from genome/metagenome mining (e.g. antiSMASH-detected clusters from uncultivated soil bacteria)

## Outputs

- Structured table (e.g. CSV) of parsed MIBiG entries: accession, biosynthetic class, taxonomic identifier, product annotation
- Homology-comparison metrics such as fraction of query BGCs lacking MIBiG homology (novelty fraction)
- Correlation values between antibacterial BGC abundance and antimicrobial resistance gene counts across soil samples

## How to apply

Fetch the MIBiG 2.0 data from the official GitHub repository (https://github.com/mibig-secmet) or query it through the REST-like web API, which handles access to the underlying PostgreSQL database. Parse each JSON-formatted BGC entry, extracting the biosynthetic class, taxonomic identifier, and product annotation fields; the JSON data standard conforms to the JSON schema description and validation technology adopted by MIBiG, so validate entries against that schema before use. Retain accession numbers and literature-backed annotation quality signals (MIBiG 2.0 added 851 new BGCs and performed extensive manual curation, expanding 1170 to 2021 curated BGCs). For downstream pathway design, confirm that tools like ClusterCAD source their BGC sequence and annotation data directly from this parsed MIBiG data rather than re-curation. Finally, collate the extracted fields (e.g. novelty fraction of query BGCs lacking MIBiG homology, correlation coefficients for antibacterial BGC abundance) into a structured CSV table for reproduction metrics.

## Related tools

- **MIBiG Repository** (Source of the JSON-formatted BGC entries parsed by this skill) — https://mibig.secondarymetabolites.org/
- **mibig-api** (REST-like web API handling access to the underlying PostgreSQL database that backs the parsed data) — https://github.com/mibig-secmet/mibig-api/
- **JSON schema** (Validation technology used to check MIBiG entry structure and dependency rules) — https://json-schema.org
- **ClusterCAD** (Downstream tool that sources BGC data from parsed MIBiG entries for pathway design)
- **antiSMASH 5** (Customised module sideloads the MIBiG JSON annotation file to generate individual BGC pages)

## Evaluation signals

- All parsed entries validate against the MIBiG JSON schema (no schema violations or missing required fields)
- Parsed entry count matches the expected MIBiG 2.0 release size (2021 curated BGCs, i.e. 1170 + 851 added)
- Extracted biosynthetic classes and taxonomic identifiers are non-empty for entries intended as homology baselines (no 'empty or mistyped information' carried over)
- Reproduced novelty fraction and correlation values fall within ranges reported in Crits-Cristoph et al. and Bahram et al. when compared against the MIBiG 2.0 reference set
- Duplicate entries are detected and removed per the curation step of 'removing duplicate entries'

## Limitations

- MIBiG is restricted to BGCs of known function/known chemical structure — it is a curated baseline, not a comprehensive candidate-BGC database
- The API server requires a recent version of Go (1.11.x) for development, so local API deployment has toolchain constraints
- Accession requests depend on approval by MIBiG staff before extended entry forms can be completed — data freshness depends on curation turnaround

## Evidence

- [intro] REST API handles access to PostgreSQL: "A REST-like web API (https://github.com/mibig-secmet/mibig-api/) handles access to the underlying PostgreSQL"
- [intro] JSON schema validation adopted: "we also adopted the JSON schema description and validation technology (https://json-schema.org)"
- [intro] antiSMASH 5 sideloads MIBiG annotation file: "The individual BGC pages are generated using a customised antiSMASH 5 module that sideloads a MIBiG annotation file"
- [abstract] 851 new BGCs added with manual curation: "Over the past five years, 851 new BGCs have been added. Additionally, we performed extensive manual data curation of all entries to improve the annotation quality"
- [readme] README: API server purpose: "The WebAPI server driving the [Minimum Information about a Biosynthetic Gene cluster (MIBiG)](http://mibig.secondarymetabolites.org/) repository."
