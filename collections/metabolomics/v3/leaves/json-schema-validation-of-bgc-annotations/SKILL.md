---
name: json-schema-validation-of-bgc-annotations
description: Use when submitting, curating, or consuming MIBiG 2.0 BGC entries in
  JSON annotation format, or before sideloading a MIBiG annotation file into the customised
  antiSMASH 5 BGC-page module.
when_to_use_negative:
- BGC annotations destined for databases like IMG-ABC or antiSMASH-DB that store large
  numbers of candidate BGCs without MIBiG's known-function curation standard
- Depositions that have not yet been assigned a MIBiG accession number after submitting
  product names and Genbank coordinates — the extended entry form workflow comes first
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_0219
- http://edamontology.org/topic_0780
tools:
- name: JSON schema
  role: Description and validation technology used to embed validation and dependency
    rules into the MIBiG data schema
  repo: https://json-schema.org
- name: antiSMASH 5
  role: Customised module sideloads a MIBiG JSON annotation file (expected to be schema-conformant)
    to render individual BGC detail pages
- name: PostgreSQL
  role: Relational database that stores the validated MIBiG entries via the REST-like
    mibig-api
  repo: https://www.postgresql.org/
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/nar/gkz882
    title: 'MIBiG 2.0: a repository for biosynthetic gene clusters of known function'
  source_content_hash: sha256:2cdfd3a7131355fe0456a676a8f3acd2cfb657fd0e806ed8d2246c3d47d4535f
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - JSON schema
  - PostgreSQL
  - antiSMASH 5
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_0219
  - http://edamontology.org/topic_0780
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# json-schema-validation-of-bgc-annotations

## Summary

Validate MIBiG biosynthetic gene cluster (BGC) annotation records against the JSON schema standard adopted in MIBiG 2.0, embedding validation and dependency rules so that sideloaded annotation files conform before being used to generate BGC detail pages.

## When to use

Use when submitting, curating, or consuming MIBiG 2.0 BGC entries in JSON annotation format, or before sideloading a MIBiG annotation file into the customised antiSMASH 5 BGC-page module.

## When NOT to use

- BGC annotations destined for databases like IMG-ABC or antiSMASH-DB that store large numbers of candidate BGCs without MIBiG's known-function curation standard
- Depositions that have not yet been assigned a MIBiG accession number after submitting product names and Genbank coordinates — the extended entry form workflow comes first

## Inputs

- MIBiG BGC annotation file in JSON format (validated against the JSON schema description)
- MIBiG JSON schema specification with embedded validation and dependency rules (from https://github.com/mibig-secmet)

## Outputs

- Schema-validated MIBiG JSON annotation records eligible for loading into the PostgreSQL relational database
- Validation pass/fail report per entry identifying schema violations (empty/mistyped fields, dependency-rule breaches)

## How to apply

MIBiG 2.0 adopted the JSON schema description and validation technology (https://json-schema.org) as the data standard, so each BGC annotation file is checked against the published JSON schema rather than a bespoke validator. Embed both validation rules (type, required fields) and dependency rules (fields whose presence or values depend on other fields) directly into the schema, then validate entries loaded into the PostgreSQL relational database that backs the repository. Validation also applies to the annotation files sideloaded by the customised antiSMASH 5 module used to generate individual BGC pages, so reject any JSON that fails schema conformance before page generation. When curating (e.g. during Annotathons or re-annotation sweeps), treat mistyped or empty information and duplicate entries as data-quality failures to fix at the schema/entry level. The SQL schema and JSON-formatted data standard are available via the MIBiG Repository GitHub organisation (https://github.com/mibig-secmet).

## Related tools

- **JSON schema** (Description and validation technology used to embed validation and dependency rules into the MIBiG data schema) — https://json-schema.org
- **antiSMASH 5** (Customised module sideloads a MIBiG JSON annotation file (expected to be schema-conformant) to render individual BGC detail pages)
- **PostgreSQL** (Relational database that stores the validated MIBiG entries via the REST-like mibig-api) — https://www.postgresql.org/

## Evaluation signals

- Every MIBiG JSON annotation record validates against the published MIBiG 2.0 JSON schema (no empty or mistyped information in required fields).
- Dependency rules embedded in the schema are enforced — fields whose presence depends on others cannot appear inconsistently.
- Sideloaded annotation files accepted by the customised antiSMASH 5 module must be the JSON files that passed schema validation before BGC page generation.
- Schema-conformant entries loaded via mibig-api are retrievable and filterable in the AngularJS front end (e.g. by species name or biosynthetic class).
- Data-quality curation outcomes: duplicates removed and low-quality entries retired without breaking schema conformance of the remaining set.

## Limitations

- Validation is only as complete as the embedded validation and dependency rules; the schema redesign was intended 'to ensure the compliance of future annotations', so legacy entries required extensive manual re-annotation to reach quality.
- The README for mibig-api documents only the Go 1.11.x requirement and Apache 2.0 license; it provides no endpoint-level validation documentation, so validation logic must be taken from the schema itself.
- JSON schema conformance does not by itself guarantee biological correctness — curation steps (fixing mistyped info, removing duplicates, adding structures, retiring low-quality entries) operate beyond schema checks.

## Evidence

- [intro] JSON schema adopted as validation technology: "we also adopted the JSON schema description and validation technology (https://json-schema.org)"
- [intro] Validation and dependency rules embedded in schema: "embed validation and dependency rules into the schema"
- [intro] antiSMASH 5 sideloads MIBiG annotation file: "a customised antiSMASH 5 module that sideloads a MIBiG annotation file"
- [intro] REST-like API over PostgreSQL: "handles access to the underlying PostgreSQL (https://www.postgresql.org/) database"
- [supplementary] Schema source on GitHub: "The source code components, JSON-formatted data standard, and SQL schema for the MIBiG Repository are available on GitHub"
- [readme] API server README: "The WebAPI server driving the [Minimum Information about a Biosynthetic Gene cluster (MIBiG)](http://mibig.secondarymetabolites.org/) repository."
