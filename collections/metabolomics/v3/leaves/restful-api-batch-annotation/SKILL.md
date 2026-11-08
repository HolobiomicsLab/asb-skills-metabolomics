---
name: restful-api-batch-annotation
description: Use when a fixed list of metabolites with known exact monoisotopic masses
  and adduct forms must be annotated programmatically via the CMM 3.0 RESTful API
  and the HMDB-sourced results verified against public HMDB records.
when_to_use_negative:
- When the annotation must come from a source other than HMDB — this skill retains
  only annotations whose database source is HMDB and cross-checks against public HMDB
  records
- When MS2-level identification is needed (spectral comparison or spectral quality
  assessment services), rather than mass/adduct batch annotation over the RESTful
  API
- When the goal is to self-host or deploy CMM source code, which instead requires
  internal CEU database access and app-server configuration
edam_operation: http://edamontology.org/operation_2422
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: CEU Mass Mediator 3.0
  role: RESTful API endpoint (ceumass.eps.uspceu.es) queried in batch mode with exact
    monoisotopic masses and adduct forms, requesting all database sources to obtain
    HMDB annotations
  repo: http://ceumass.eps.uspceu.es/
- name: HMDB (Human Metabolome Database)
  role: Reference database into which the CMM RESTful API is integrated; public records
    fetched by accession are used to verify returned identifier, formula, and exact
    mass
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1021/acs.jproteome.8b00720
    title: 'CEU Mass Mediator 3.0: A Metabolite Annotation Tool'
  source_content_hash: sha256:145e7368b41326b00e6bcb34076935277e9f1ad58a2d21b6bde59f1325f2a118
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/albertogilf/ceuMassMediator
  tools:
  - CEU Mass Mediator 3.0
  - HMDB (Human Metabolome Database)
  edam_operation: http://edamontology.org/operation_2422
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# restful-api-batch-annotation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Programmatically retrieves metabolite annotations from CEU Mass Mediator 3.0 via its RESTful API (ceumass.eps.uspceu.es), which the paper reports has already been integrated into HMDB, and validates returned records (identifier, formula, exact mass, adduct, score, database source) against public HMDB entries.

## When to use

Use when a fixed list of metabolites with known exact monoisotopic masses and adduct forms must be annotated programmatically via the CMM 3.0 RESTful API and the HMDB-sourced results verified against public HMDB records.

## When NOT to use

- When the annotation must come from a source other than HMDB — this skill retains only annotations whose database source is HMDB and cross-checks against public HMDB records
- When MS2-level identification is needed (spectral comparison or spectral quality assessment services), rather than mass/adduct batch annotation over the RESTful API
- When the goal is to self-host or deploy CMM source code, which instead requires internal CEU database access and app-server configuration

## Inputs

- Fixed query list of metabolites present in both CMM and HMDB, with exact monoisotopic masses
- Tested adduct forms for each metabolite
- HMDB accession identifiers of the expected target compounds

## Outputs

- Parsed JSON annotation records (identifier, formula, exact mass, adduct, score, database source)
- Validated HMDB-only annotation set with cross-check results, written to cmm_hmdb_annotations.json

## How to apply

First build a fixed query list of metabolites present in both CMM and HMDB, recording each compound's exact monoisotopic mass and the adduct forms to be tested. For each metabolite, issue a batch annotation query to the CMM RESTful API endpoint (ceumass.eps.uspceu.es), requesting all database sources so that HMDB matches are included among the returned annotations. Parse the JSON responses and extract each annotation record's identifier, formula, exact mass, adduct, score, and database source into a single structure. Filter the parsed records to keep only annotations whose source database is HMDB, then fetch the corresponding public HMDB record by its accession and cross-check the identifier, formula, and exact mass. Write the validated annotation records together with the comparison results to cmm_hmdb_annotations.json. Rationale: CMM 3.0's expert system filters and scores query results based on the experimental setup and compound properties, so submitting controlled masses/adducts and requesting all sources makes the HMDB subset reproducible and independently verifiable.

## Related tools

- **CEU Mass Mediator 3.0** (RESTful API endpoint (ceumass.eps.uspceu.es) queried in batch mode with exact monoisotopic masses and adduct forms, requesting all database sources to obtain HMDB annotations) — http://ceumass.eps.uspceu.es/
- **HMDB (Human Metabolome Database)** (Reference database into which the CMM RESTful API is integrated; public records fetched by accession are used to verify returned identifier, formula, and exact mass)

## Evaluation signals

- Every annotation retained after filtering has database source equal to HMDB
- Each retained record's identifier, formula, and exact mass match the corresponding public HMDB record fetched via its accession
- All parsed records contain the fields identifier, formula, exact mass, adduct, score, and database source
- API responses are valid JSON and parse without errors across the full batch query list
- Validated records plus comparison results are written to cmm_hmdb_annotations.json

## Limitations

- Cross-checking in this workflow is limited to HMDB; the paper reports the RESTful API has been integrated into HMDB specifically
- Self-hosting the CMM source code requires access to an internal CEU database (requested via email) and a data-source configured on the app server with javax.faces-2.2.12.jar
- No discussion/changelog content was available, so deployment or API-version caveats beyond the abstract and README are not documented

## Evidence
