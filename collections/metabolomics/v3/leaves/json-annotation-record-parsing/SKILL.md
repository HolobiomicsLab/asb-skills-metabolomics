---
name: json-annotation-record-parsing
description: Use when JSON annotation responses from CMM 3.0 RESTful API queries must
  be parsed and filtered to HMDB-sourced records so identifier, formula, and exact
  mass can be cross-checked against public HMDB accessions.
when_to_use_negative:
- Records are already structured/validated annotation data and no JSON parsing or
  HMDB-source filtering is needed
- Target metabolites are absent from HMDB — filtering to HMDB-sourced annotations
  will yield an empty subset and cross-checking cannot proceed
- The analysis needs raw annotation across all CMM database sources rather than HMDB-verified
  records (filtering would discard non-HMDB annotations)
edam_operation: http://edamontology.org/operation_3282
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0091
tools:
- name: CEU Mass Mediator 3.0
  role: Source of the RESTful API whose JSON annotation responses are parsed and filtered
    to HMDB-sourced records
  repo: albertogilf/ceuMassMediator
- name: HMDB (Human Metabolome Database)
  role: Integrated consumer of the CMM RESTful API and the reference database against
    which identifiers, formulas, and exact masses are verified
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1021/acs.jproteome.8b00720
    title: 'CEU Mass Mediator 3.0: A Metabolite Annotation Tool'
  source_content_hash: sha256:59c7baaeb34ebf1637dfa58674d269f61e0cc6c44250efa5db658b174561cf5b
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/albertogilf/ceuMassMediator
  repo_url: https://github.com/albertogilf/ceuMassMediator
  tools:
  - CEU Mass Mediator 3.0
  - HMDB (Human Metabolome Database)
  edam_operation: http://edamontology.org/operation_3282
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
emails_redacted: true
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6
n_claims: 5
---

# json-annotation-record-parsing

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Parses JSON responses returned by the CEU Mass Mediator 3.0 RESTful API (ceumass.eps.uspceu.es) for batch metabolite annotation queries, extracting annotation records (identifier, formula, exact mass, adduct, score, database source) into a structured form and filtering to HMDB-sourced entries for verification against public HMDB records.

## When to use

Use when JSON annotation responses from CMM 3.0 RESTful API queries must be parsed and filtered to HMDB-sourced records so identifier, formula, and exact mass can be cross-checked against public HMDB accessions.

## When NOT to use

- Records are already structured/validated annotation data and no JSON parsing or HMDB-source filtering is needed
- Target metabolites are absent from HMDB — filtering to HMDB-sourced annotations will yield an empty subset and cross-checking cannot proceed
- The analysis needs raw annotation across all CMM database sources rather than HMDB-verified records (filtering would discard non-HMDB annotations)

## Inputs

- JSON response bodies from CMM 3.0 RESTful API batch annotation queries against ceumass.eps.uspceu.es
- Fixed query list of metabolites present in both CMM and HMDB, with their exact monoisotopic masses and tested adduct forms
- Public HMDB records fetched via accession for cross-checking

## Outputs

- Single structured collection of parsed annotation records (identifier, formula, exact mass, adduct, score, database source)
- Filtered subset of HMDB-sourced annotations
- cmm_hmdb_annotations.json containing validated annotation records and comparison results

## How to apply

Issue batch annotation queries to the CMM 3.0 RESTful API endpoint at ceumass.eps.uspceu.es, requesting all database sources so HMDB matches are returned. Parse each JSON response and extract the annotation fields — identifier, formula, exact mass, adduct, score, and database source — into a single uniform structure. Filter the parsed records to keep only annotations whose source database is HMDB; records from other CMM sources (including the 681,198 predicted metabolites) are outside this verification scope. Cross-check each retained HMDB annotation's identifier, formula, and exact mass against the corresponding public HMDB record fetched by its accession, since the CMM RESTful API was explicitly created for HMDB integration. Write the validated records plus per-record comparison results to cmm_hmdb_annotations.json. Rationale: CMM's expert system 'filters and scores the query results', so the returned score and database-source fields must be carried through parsing to make the downstream verification auditable.

## Related tools

- **CEU Mass Mediator 3.0** (Source of the RESTful API whose JSON annotation responses are parsed and filtered to HMDB-sourced records) — albertogilf/ceuMassMediator
- **HMDB (Human Metabolome Database)** (Integrated consumer of the CMM RESTful API and the reference database against which identifiers, formulas, and exact masses are verified)

## Examples

```
import json; recs=json.load(open('cmm_response.json')); hmdb=[r for r in recs if r['database']=='HMDB']; json.dump(hmdb, open('cmm_hmdb_annotations.json','w'))
```

## Evaluation signals

- Every parsed record contains the fields identifier, formula, exact mass, adduct, score, and database source before filtering
- Post-filter records all have database source equal to HMDB — no non-HMDB sources survive
- Each retained HMDB identifier resolves to a public HMDB accession whose formula and exact mass match the CMM annotation
- Final cmm_hmdb_annotations.json contains both the validated annotation records and the per-record comparison results
- JSON output parses cleanly (valid JSON) with a one-record-per-annotation schema

## Limitations

- Verification is restricted to HMDB-sourced annotations; CMM's 681,198 predicted metabolites and annotations from other database sources cannot be cross-checked this way
- CMM 3.0 covers 332,665 experimental metabolites — metabolites outside this coverage will return no annotations to parse
- Deployment/operation of the CMM backend requires access to an internal CEU database requested via email (<maintainer e-mail, see the repository README>) and a configured data-source on the app server with javax.faces-2.2.12.jar

## Evidence
