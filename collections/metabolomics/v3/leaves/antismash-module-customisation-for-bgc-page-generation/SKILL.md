---
name: antismash-module-customisation-for-bgc-page-generation
description: Use when BGC detail pages must be rendered for curated BGC entries (e.g.
  MIBiG annotations stored as JSON) rather than recomputed from scratch, using antiSMASH
  5 sideloading instead of full de novo genome-mining detection.
when_to_use_negative:
- Input is a raw genome/metagenome assembly with no curated MIBiG JSON annotation
  — sideloading requires pre-curated annotations; run antiSMASH 5 de novo detection
  instead
- The goal is querying or filtering the repository rather than rendering BGC pages
  — that is handled by the PostgreSQL-backed REST-like mibig-api and AngularJS front
  end, not this module
edam_operation: http://edamontology.org/operation_0338
tools:
- name: antiSMASH 5
  role: customised module that sideloads the MIBiG JSON annotation file to generate
    individual BGC detail pages
- name: JSON schema
  role: description and validation technology ensuring the sideloaded MIBiG annotation
    file conforms to the JSON-formatted data standard before page generation
  repo: https://json-schema.org
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/nar/gkz882
    title: 'MIBiG 2.0: a repository for biosynthetic gene clusters of known function'
  source_content_hash: sha256:67c2a4e3a42d1d40d6414002cb51bb1ee663caf62938a90158f9007f618c82dd
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - JSON schema
  - PostgreSQL
  - antiSMASH
  - antiSMASH 5
  edam_operation: http://edamontology.org/operation_0338
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# antismash-module-customisation-for-bgc-page-generation

## Summary

Generates individual biosynthetic gene cluster (BGC) detail pages for the MIBiG repository by running a customised antiSMASH 5 module that sideloads a MIBiG annotation file in JSON format, validated against the JSON schema standard.

## When to use

Use when BGC detail pages must be rendered for curated BGC entries (e.g. MIBiG annotations stored as JSON) rather than recomputed from scratch, using antiSMASH 5 sideloading instead of full de novo genome-mining detection.

## When NOT to use

- Input is a raw genome/metagenome assembly with no curated MIBiG JSON annotation — sideloading requires pre-curated annotations; run antiSMASH 5 de novo detection instead
- The goal is querying or filtering the repository rather than rendering BGC pages — that is handled by the PostgreSQL-backed REST-like mibig-api and AngularJS front end, not this module

## Inputs

- MIBiG annotation file for a BGC entry, in JSON format and validated against the JSON schema data standard (compound name, biosynthetic class, taxonomy, BGC sequence/Genbank coordinates, literature references)
- customised antiSMASH 5 module configuration (sideload mode)

## Outputs

- individual BGC detail pages (HTML) rendered from the sideloaded MIBiG annotation
- page content wired into the AngularJS single-page repository UI for link-out from repository rows

## How to apply

Take the curated MIBiG annotation file for a BGC entry, which must conform to the JSON-formatted MIBiG data standard validated with the JSON schema description and validation technology (json-schema.org). Instead of running antiSMASH 5 de novo on raw sequence, configure a customised antiSMASH 5 module that sideloads this MIBiG JSON annotation file so the module consumes the curated annotations (compound names, biosynthetic class, taxonomy, structure data) rather than re-predicting them. The module then renders the individual BGC detail pages that the AngularJS single-page front end links out to, alongside the chemical structure database link-outs introduced in MIBiG 2.0. Rationale: sideloading preserves manually curated annotation quality from Annotathons and classroom curation, which would be lost if pages were generated from fresh antiSMASH predictions. Validate by confirming each generated BGC page reflects the sideloaded MIBiG annotations and is reachable from the repository rows in the AngularJS UI.

## Related tools

- **antiSMASH 5** (customised module that sideloads the MIBiG JSON annotation file to generate individual BGC detail pages)
- **JSON schema** (description and validation technology ensuring the sideloaded MIBiG annotation file conforms to the JSON-formatted data standard before page generation) — https://json-schema.org

## Evaluation signals

- Each generated BGC detail page renders the fields present in the sideloaded MIBiG JSON annotation (compound names, biosynthetic class, taxonomy, chemical structure links)
- Sideloaded annotation files validate against the MIBiG JSON schema before page generation
- BGC pages are linked out from the corresponding rows in the AngularJS repository UI and are reachable from the REST-like mibig-api front end
- Re-annotated/curated entries (post-Annotathon curation) display updated annotations rather than de novo antiSMASH predictions
- Chemical structure database link-outs added in MIBiG 2.0 appear on the generated BGC pages

## Limitations

- Page generation depends on the quality of the curated MIBiG JSON annotation; empty or mistyped information in the data propagates to the rendered pages unless fixed during re-annotation
- The customised module is tied to the antiSMASH 5 version and the MIBiG 2.0 JSON data standard; schema changes require regenerating pages
- The skill produces detail pages only — it does not itself provide browsing, filtering or Boolean querying, which live in the PostgreSQL/REST-like API/AngularJS stack

## Evidence

- [intro] BGC pages generated by sideload module: "The individual BGC pages are generated using a customised antiSMASH 5 module that sideloads a MIBiG annotation file"
- [intro] JSON schema validation of annotations: "we also adopted the JSON schema description and validation technology (https://json-schema.org)"
- [supplementary] source code and data standard on GitHub: "The source code components, JSON-formatted data standard, and SQL schema for the MIBiG Repository are available on GitHub"
