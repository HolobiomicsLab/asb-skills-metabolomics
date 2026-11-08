---
name: ceumass-restful-querying
description: Use when a task needs compound records from CMM 3.0 via its RESTful API,
  or when verifying that newly added taxonomy and ontology metadata fields are present
  and populated for a set of queried metabolites.
when_to_use_negative:
- The analysis goal is MS2 spectral comparison or spectral quality assessment — CMM
  3.0 exposes these as separate services rather than metadata-field queries.
- The compounds of interest are oxidized lipids requiring the CMM in-house oxidized-lipid
  library or the oxidized glycerophosphocholine annotation service, not general metadata
  coverage auditing.
- You only need bulk database downloads rather than per-compound API queries with
  metadata field checks.
edam_operation: http://edamontology.org/operation_2422
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: CEU Mass Mediator (CMM) 3.0
  role: RESTful API queried per metabolite to retrieve compound records and inspect
    newly added taxonomy and ontology metadata fields
  repo: https://github.com/albertogilf/ceuMassMediator
- name: HMDB (Human Metabolome Database)
  role: External integration showing the RESTful API's intended programmatic use;
    context for API-based querying
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1021/acs.jproteome.8b00720
    title: 'CEU Mass Mediator 3.0: A Metabolite Annotation Tool'
  source_content_hash: sha256:96ee75aaa1d27c42de6555e1d75fbe15d8caa54d11e042a7564b7eb684ec26e3
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
  - CEU Mass Mediator (CMM) 3.0
  - HMDB (Human Metabolome Database)
  edam_operation: http://edamontology.org/operation_2422
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
emails_redacted: true
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# ceumass-restful-querying

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Query the CEU Mass Mediator (CMM) 3.0 RESTful API to retrieve compound records enriched with new taxonomy and ontology metadata, and audit field coverage across a sampled set of metabolites.

## When to use

Use when a task needs compound records from CMM 3.0 via its RESTful API, or when verifying that newly added taxonomy and ontology metadata fields are present and populated for a set of queried metabolites.

## When NOT to use

- The analysis goal is MS2 spectral comparison or spectral quality assessment — CMM 3.0 exposes these as separate services rather than metadata-field queries.
- The compounds of interest are oxidized lipids requiring the CMM in-house oxidized-lipid library or the oxidized glycerophosphocholine annotation service, not general metadata coverage auditing.
- You only need bulk database downloads rather than per-compound API queries with metadata field checks.

## Inputs

- Fixed sample of metabolite names or identifiers queryable via the CMM 3.0 service (e.g., 20 metabolites)
- CMM 3.0 RESTful API endpoint

## Outputs

- metadata_coverage.csv — one row per sampled metabolite with binary presence/absence flags for each taxonomy and ontology metadata field
- Parsed CMM 3.0 compound records including taxonomy/ontology metadata (e.g., structural class, ontology terms)

## How to apply

Select a fixed sample of metabolites that are queryable through the CEU Mass Mediator 3.0 service. Issue one query per metabolite against the CMM RESTful API, which the paper notes was created specifically for programmatic integration (e.g., into HMDB). For each returned compound record, parse the response and detect the presence or absence of the newly added taxonomy and ontology metadata fields (e.g., structural class and ontology terms). Assemble a per-metabolite table with binary field-presence flags, one row per compound, and write it to metadata_coverage.csv. Judge success by confirming the API returns records for all sampled metabolites and by inspecting which metadata fields are populated versus missing across the sample.

## Related tools

- **CEU Mass Mediator (CMM) 3.0** (RESTful API queried per metabolite to retrieve compound records and inspect newly added taxonomy and ontology metadata fields) — https://github.com/albertogilf/ceuMassMediator
- **HMDB (Human Metabolome Database)** (External integration showing the RESTful API's intended programmatic use; context for API-based querying)

## Examples

```
for m in metabolites.txt: r = requests.get('http://ceumass.eps.uspceu.es/api/compound', params={'query': m}); flags = {f: (f in r.json()) for f in TAXONOMY_FIELDS}
```

## Evaluation signals

- The CMM RESTful API returns a parsed record for every metabolite in the fixed sample (e.g., 20/20 successful queries).
- metadata_coverage.csv has exactly one row per sampled metabolite with binary presence/absence flags per metadata field.
- Each row's flags cover the newly added taxonomy and ontology fields (e.g., structural class, ontology terms) described for CMM 3.0.
- Field-population results are consistent across repeated queries of the same metabolite sample.
- Queried metabolites resolve against CMM 3.0's experimental library (332,665 experimental metabolites) rather than only predicted entries.

## Limitations

- Deployment of the underlying service requires access to an internal CEU database (requested via email to <maintainer e-mail, see the repository README>) and a data-source configured on the app server with provided credentials, so self-hosting is not turnkey.
- The app server requires the javax.faces-2.2.12.jar library dependency.
- Coverage of the new taxonomy/ontology metadata fields may be incomplete across compounds — the audit is precisely what quantifies gaps, so missing flags are an expected finding, not necessarily an error.

## Evidence

- [abstract] new taxonomy and ontology metadata have been included: "new taxonomy and ontology metadata have been included"
- [abstract] a RESTful API has been created, and it has already been integrated into the HMDB (Human Metabolome Database): "a RESTful API has been created, and it has already been integrated into the HMDB (Human Metabolome Database)"
- [abstract] has increased the number of experimental metabolites up 332,665 and the number of predicted metabolites to 681,198: "has increased the number of experimental metabolites up 332,665 and the number of predicted metabolites to 681,198"
- [readme] It is necessary to access an internal database of CEU. Request access to <maintainer e-mail, see the repository README>.: "It is necessary to access an internal database of CEU. Request access to <maintainer e-mail, see the repository README>."
