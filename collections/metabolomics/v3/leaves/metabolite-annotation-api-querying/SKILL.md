---
name: metabolite-annotation-api-querying
description: Use when a downstream application (e.g. HMDB or a custom pipeline) needs
  to fetch compound annotations programmatically from CMM 3.0 — i.e. given a list
  of metabolite exact masses and adduct forms, you must query ceumass.eps.uspceu.
when_to_use_negative:
- When the goal is interactive, exploratory annotation through the CMM web interface
  (the web UI at ceumass.eps.uspceu.es is the appropriate route, not the API)
- When the metabolites are oxidized glycerophosphocholines or require MS2 spectral
  comparison/quality assessment — those are separate CMM 3.0 services, not the generic
  annotation API query flow
- 'When self-hosting CMM: deploying the source code requires access to an internal
  CEU database requested via <maintainer e-mail, see the repository README> plus a
  server-side data-source, which is out of scope for simply consuming the hosted API'
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_3337
- http://edamontology.org/topic_0154
tools:
- name: CEU Mass Mediator 3.0
  role: The annotation service whose RESTful API is queried to retrieve compound annotations
    across all database sources
  repo: https://github.com/albertogilf/ceuMassMediator
- name: HMDB (Human Metabolome Database)
  role: Reference database used to cross-check returned annotation identifiers, formulas,
    and exact masses; also the platform into which CMM's API has been integrated
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1021/acs.jproteome.8b00720
    title: 'CEU Mass Mediator 3.0: A Metabolite Annotation Tool'
  source_content_hash: sha256:ad0ed2a0b71efaad39ba90216d3c95cc79c45324238dbda003aa6de6969efd3d
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
  edam_operation: http://edamontology.org/operation_3646
  edam_topics:
  - http://edamontology.org/topic_3337
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
emails_redacted: true
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# metabolite-annotation-api-querying

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A skill for programmatically retrieving metabolite annotations through the CEU Mass Mediator 3.0 RESTful API and cross-checking returned records (identifiers, formulas, exact masses) against an external reference database such as HMDB, so annotation results can be validated outside the web interface.

## When to use

Use when a downstream application (e.g. HMDB or a custom pipeline) needs to fetch compound annotations programmatically from CMM 3.0 — i.e. given a list of metabolite exact masses and adduct forms, you must query ceumass.eps.uspceu.es over HTTP rather than the web UI, and returned records must be verifiable against a public reference database.

## When NOT to use

- When the goal is interactive, exploratory annotation through the CMM web interface (the web UI at ceumass.eps.uspceu.es is the appropriate route, not the API)
- When the metabolites are oxidized glycerophosphocholines or require MS2 spectral comparison/quality assessment — those are separate CMM 3.0 services, not the generic annotation API query flow
- When self-hosting CMM: deploying the source code requires access to an internal CEU database requested via <maintainer e-mail, see the repository README> plus a server-side data-source, which is out of scope for simply consuming the hosted API

## Inputs

- Fixed query list of metabolite exact monoisotopic masses (e.g. those present in both CMM 3.0 and HMDB)
- Tested adduct forms for each metabolite
- CMM 3.0 RESTful API endpoint URL (ceumass.eps.uspceu.es)
- Public reference database records (e.g. HMDB accession-based fetches) for cross-checking

## Outputs

- Parsed annotation records (identifier, formula, exact mass, adduct, score, database source) extracted from API JSON responses
- Filtered subset of annotations retaining only records from the target source database (e.g. HMDB)
- Validated annotation records with cross-check results against the reference database
- Single consolidated JSON output file (e.g. cmm_hmdb_annotations.json)

## How to apply

First build a fixed query list of metabolite monoisotopic masses and the adduct forms to test (CMM 3.0 covers 332,665 experimental and 681,198 predicted metabolites, so restrict the query space before issuing requests). For each metabolite, submit a batch annotation query to the CMM RESTful API endpoint at ceumass.eps.uspceu.es, requesting all database sources so that records from HMDB, and other integrated sources, are returned in a single call; parse the JSON responses and extract the annotation record fields — identifier, formula, exact mass, adduct, score, and database source. Filter the parsed records to keep only annotations whose source database is the reference of interest (e.g. HMDB). For each retained annotation, cross-check the identifier, formula, and exact mass against the corresponding public reference record, then write the validated annotations and comparison results to a single structured JSON file (e.g. cmm_hmdb_annotations.json). The rationale is that CMM 3.0 exposes a RESTful API precisely so external platforms can integrate its annotation capability — as demonstrated by its integration into HMDB — and validation against a reference database turns raw API output into trusted annotation records.

## Related tools

- **CEU Mass Mediator 3.0** (The annotation service whose RESTful API is queried to retrieve compound annotations across all database sources) — https://github.com/albertogilf/ceuMassMediator
- **HMDB (Human Metabolome Database)** (Reference database used to cross-check returned annotation identifiers, formulas, and exact masses; also the platform into which CMM's API has been integrated)

## Examples

```
curl -s 'https://ceumass.eps.uspceu.es/api/annotation?mass=180.0633&adducts=%5BM%2BH%5D&databases=all' | python -c "import sys,json; [print(r['identifier'],r['formula'],r['exact_mass'],r['database_source']) for r in json.load(sys.stdin) if r['database_source']=='HMDB']"
```

## Evaluation signals

- Every retained annotation record contains a non-empty identifier, formula, exact mass, adduct, and database source field after JSON parsing
- Only records whose source database equals the target reference (e.g. HMDB) remain after filtering — records from other sources are excluded
- For each validated annotation, the identifier, formula, and exact mass match the corresponding public HMDB record fetched by accession
- Output is a single structured JSON file (e.g. cmm_hmdb_annotations.json) combining the annotation records and their comparison results
- The set of queried metabolites matches the fixed input list in both count and adduct coverage (no silently dropped queries)

## Limitations

- Deployment of the underlying CMM source code requires access to an internal CEU database (request via <maintainer e-mail, see the repository README>) and configuration of a data-source on the app server with provided credentials — the API is only usable as-is against the hosted instance
- The server needs the javax.faces-2.2.12.jar library for the JSF app, so locally rebuilt instances may fail without it
- CMM 3.0's annotation is expert-system-filtered based on the experimental setup and compound properties, so API results depend on how well the query mass/adduct reflects the actual experiment
- The article does not document request rate limits, pagination, or error-handling behaviour of the RESTful API, so robust production use requires defensive parsing of JSON responses

## Evidence
