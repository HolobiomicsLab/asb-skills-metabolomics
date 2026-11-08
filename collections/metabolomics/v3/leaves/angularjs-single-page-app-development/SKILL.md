---
name: angularjs-single-page-app-development
description: 'Use when migrating static HTML data pages into a dynamic repository:
  a relational backend (PostgreSQL) and REST-like API (mibig-api) already exist, and
  users need in-browser filtering, sorting and query building over annotation metadata.'
when_to_use_negative:
- 'The data volume is small and static: static HTML pages remain adequate, as in pre-2.0
  MIBiG, and a dynamic API + SPA stack adds avoidable complexity'
- The backend is not PostgreSQL served through the mibig-api REST-like interface —
  the AngularJS front end depends on that specific API contract
- The task is generating individual BGC detail pages themselves; that is done by a
  customised antiSMASH 5 module sideloading a MIBiG JSON annotation file, not by the
  AngularJS application
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3361
tools:
- name: AngularJS
  role: JavaScript framework used to write the single-page web application that consumes
    the REST-like API and implements filtering/sorting of repository rows
  repo: https://angularjs.org/
- name: PostgreSQL
  role: Relational database backend holding the migrated MIBiG data, accessed by the
    SPA via the API
  repo: https://www.postgresql.org/
- name: mibig-api
  role: REST-like web API server (Go, Apache-2.0) that handles access to the underlying
    PostgreSQL database on behalf of the front end
  repo: https://github.com/mibig-secmet/mibig-api/
- name: JSON schema
  role: Description and validation technology adopted for the MIBiG annotation files
    served to and linked from the application
  repo: https://json-schema.org
- name: antiSMASH 5
  role: Customised module generating the individual BGC detail pages that the SPA
    links out to, sideloading a MIBiG annotation file
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/nar/gkz882
    title: 'MIBiG 2.0: a repository for biosynthetic gene clusters of known function'
  source_content_hash: sha256:feedea23d92b28bbf52e93dea1403d01741ec810cfb5daa2fdd4608a4e798b23
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - AngularJS
  - JSON schema
  - PostgreSQL
  - antiSMASH 5
  - mibig-api
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3361
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# angularjs-single-page-app-development

## Summary

Build a single-page AngularJS web application that consumes a REST-like web API backed by a PostgreSQL relational database, replacing static HTML pages with dynamic browsing, filtering, sorting and Boolean querying of repository rows (as done for the MIBiG 2.0 repository).

## When to use

Use when migrating static HTML data pages into a dynamic repository: a relational backend (PostgreSQL) and REST-like API (mibig-api) already exist, and users need in-browser filtering, sorting and query building over annotation metadata.

## When NOT to use

- The data volume is small and static: static HTML pages remain adequate, as in pre-2.0 MIBiG, and a dynamic API + SPA stack adds avoidable complexity
- The backend is not PostgreSQL served through the mibig-api REST-like interface — the AngularJS front end depends on that specific API contract
- The task is generating individual BGC detail pages themselves; that is done by a customised antiSMASH 5 module sideloading a MIBiG JSON annotation file, not by the AngularJS application

## Inputs

- REST-like web API exposing MIBiG annotation metadata from a PostgreSQL relational database (mibig-api)
- MIBiG JSON-formatted annotation files validated against the JSON schema standard
- User query parameters: compound names, taxonomic identifiers, biosynthetic classes, completeness and BGC class filters

## Outputs

- Single-page AngularJS web application with a filterable, sortable 'Repository' page and Boolean query builder
- Statistics page summarising repository contents
- Link-outs from repository rows to antiSMASH 5-generated BGC detail pages and to chemical/analytical databases (e.g. Natural Products Atlas, GNPS)

## How to apply

First confirm the server side is in place: the REST-like web API (mibig-api, Apache-2.0 licensed, requiring a recent Go 1.11.x toolchain per its README) handles access to the underlying PostgreSQL database. Then implement the front end as a single-page application written in AngularJS (https://angularjs.org/) that consumes the API endpoints rather than fetching static pages. Wire the 'Repository' page so its rows can be filtered and sorted based on annotation metadata such as compound names, taxonomic identifiers or biosynthetic classes, and expose Boolean query building (e.g. 'find all complete RiPP BGCs from the genus Streptomyces'). Provide fast client-side filtering plus a statistics page, and link each entry out to BGC detail pages generated by a customised antiSMASH 5 module that sideloads a MIBiG JSON annotation file, as well as to chemical-structure databases such as the Natural Products Atlas. The rationale for the SPA + API split is that annotation metadata changes after curation rounds (e.g. MIBiG 2.0's re-annotation of all entries) must be reflected immediately in browse/filter views without regenerating static HTML.

## Related tools

- **AngularJS** (JavaScript framework used to write the single-page web application that consumes the REST-like API and implements filtering/sorting of repository rows) — https://angularjs.org/
- **PostgreSQL** (Relational database backend holding the migrated MIBiG data, accessed by the SPA via the API) — https://www.postgresql.org/
- **mibig-api** (REST-like web API server (Go, Apache-2.0) that handles access to the underlying PostgreSQL database on behalf of the front end) — https://github.com/mibig-secmet/mibig-api/
- **JSON schema** (Description and validation technology adopted for the MIBiG annotation files served to and linked from the application) — https://json-schema.org
- **antiSMASH 5** (Customised module generating the individual BGC detail pages that the SPA links out to, sideloading a MIBiG annotation file)

## Evaluation signals

- The 'Repository' page rows can be filtered and sorted based on annotation metadata (e.g. species names, biosynthetic classes) as described in the article
- A Boolean query such as 'find all complete RiPP BGCs from the genus Streptomyces' returns the correct subset of repository entries
- Each repository entry links out to a BGC detail page generated by the customised antiSMASH 5 sideload module
- Front end reads all data through the mibig-api REST-like interface backed by PostgreSQL — no reliance on static HTML pages remains
- Statistics page renders counts consistent with the database contents (MIBiG 2.0: 2021 curated BGCs, 851 new entries over five years)

## Limitations

- The mibig-api server requires a recent Go version (1.11.x) for dependency handling, so contributors must match that toolchain
- The AngularJS front end is tightly coupled to the mibig-api REST contract and PostgreSQL schema published in the MIBiG Repository GitHub organisation
- The repository README lacks a changelog; documented ('No changelog found'), so behaviour must be verified against the live deployment rather than version history

## Evidence

- [intro] SPA consuming REST API: "A REST-like web API (https://github.com/mibig-secmet/mibig-api/) handles access to the underlying PostgreSQL"
- [intro] AngularJS SPA: "A single-page web application written in AngularJS (https://angularjs.org/)"
- [intro] filtering and sorting: "Rows in the ‘Repository’ page can now be filtered and sorted based on annotation metadata"
- [intro] Boolean query example: "‘find all complete RiPP BGCs from the genus Streptomyces’"
- [intro] antiSMASH 5 sideload: "The individual BGC pages are generated using a customised antiSMASH 5 module that sideloads a MIBiG annotation file"
- [intro] migration from static pages: "the MIBiG data has now been migrated into a relational database"
- [readme] README Go toolchain requirement: "Make sure you have a recent version of Go (1.11.x), as the dependency handling requires that."
