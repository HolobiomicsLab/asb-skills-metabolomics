---
name: mass-formula-consistency-checking
description: Use when validating metabolite annotations returned by CMM 3.0 (ceumass.eps.uspceu.es),
  e.g. checking HMDB identifier, formula, and exact-mass triples against public records
  before trusting an annotation.
when_to_use_negative:
- The compounds of interest come from CMM's in-house oxidized lipids library, which
  is 'not present in other sources', so no HMDB record exists to cross-check against
- The goal is MS2 spectral matching or spectrum reliability scoring — CMM 3.0 exposes
  separate spectral comparison and spectral quality-assessment services for that
- The annotations are already locally validated confirmed identifications and no external
  database reconciliation is needed
edam_operation: http://edamontology.org/operation_2422
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0092
tools:
- name: CEU Mass Mediator (CMM) 3.0
  role: RESTful metabolite-annotation API whose returned identifier/formula/exact-mass/adduct
    records are queried, filtered to HMDB, and cross-checked
  repo: https://github.com/albertogilf/ceuMassMediator
- name: HMDB (Human Metabolome Database)
  role: Public reference database supplying the authoritative accession, formula,
    and exact mass used to validate each CMM annotation
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1021/acs.jproteome.8b00720
    title: 'CEU Mass Mediator 3.0: A Metabolite Annotation Tool'
  source_content_hash: sha256:383fba61af39b21ed9f3a87e837cc3d9ee9f612e7b4c3b2389bd6a8fd1f9c816
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
  - CEU Mass Mediator 3.0
  - HMDB (Human Metabolome Database)
  edam_operation: http://edamontology.org/operation_2422
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
emails_redacted: true
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4
n_claims: 15
---

# mass-formula-consistency-checking

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Cross-checks metabolite annotations returned by CEU Mass Mediator 3.0 (identifier, molecular formula, exact monoisotopic mass, adduct) against the corresponding public HMDB records fetched by accession. This validates records drawn from CMM's compound space of 332,665 experimental and 681,198 predicted metabolites before they are reused downstream.

## When to use

Use when validating metabolite annotations returned by CMM 3.0 (ceumass.eps.uspceu.es), e.g. checking HMDB identifier, formula, and exact-mass triples against public records before trusting an annotation.

## When NOT to use

- The compounds of interest come from CMM's in-house oxidized lipids library, which is 'not present in other sources', so no HMDB record exists to cross-check against
- The goal is MS2 spectral matching or spectrum reliability scoring — CMM 3.0 exposes separate spectral comparison and spectral quality-assessment services for that
- The annotations are already locally validated confirmed identifications and no external database reconciliation is needed

## Inputs

- Fixed query list of metabolites present in both CMM and HMDB, with exact monoisotopic masses and tested adduct forms
- JSON annotation responses from the CMM 3.0 RESTful API (fields: identifier, formula, exact mass, adduct, score, database source)
- Public HMDB records retrieved by accession for cross-checking

## Outputs

- cmm_hmdb_annotations.json: validated HMDB annotation records with identifier/formula/exact-mass comparison results
- Per-annotation match/mismatch verdicts for identifier, formula, and exact mass

## How to apply

Build a fixed query list of metabolites present in both CMM and HMDB, recording each compound's exact monoisotopic mass and the adduct forms to test. Issue batch annotation queries against the CMM 3.0 RESTful API at ceumass.eps.uspceu.es requesting all database sources so HMDB matches are returned; because CMM unifies compounds from multiple databases and filters/scores them with its expert system, raw returned records are not authoritative and must be re-verified. Parse the JSON responses, extracting identifier, formula, exact mass, adduct, score, and database source per record, then filter to keep only annotations whose source database is HMDB. Fetch each matched HMDB record via its accession and cross-check identifier, formula, and exact mass, additionally recomputing the monoisotopic mass from the formula as an internal mass-formula consistency invariant (predicted metabolites, 681,198 of CMM 3.0's compounds, are the likeliest source of mismatched mass/formula pairs). Write the validated annotation records together with the per-field comparison results to cmm_hmdb_annotations.json so mismatches remain auditable.

## Related tools

- **CEU Mass Mediator (CMM) 3.0** (RESTful metabolite-annotation API whose returned identifier/formula/exact-mass/adduct records are queried, filtered to HMDB, and cross-checked) — https://github.com/albertogilf/ceuMassMediator
- **HMDB (Human Metabolome Database)** (Public reference database supplying the authoritative accession, formula, and exact mass used to validate each CMM annotation)

## Examples

```
valid = [a for a in cmm_annotations if a["database_source"] == "HMDB"]; json.dump(valid, open("cmm_hmdb_annotations.json", "w"))
```

## Evaluation signals

- After filtering, every retained record has database source equal to HMDB
- For each annotation, identifier, formula, and exact mass all match the public HMDB record fetched by its accession, with any mismatch recorded in the comparison results
- Exact mass is consistent with its molecular formula when recomputed as a monoisotopic mass (formula-mass invariant holds per record)
- cmm_hmdb_annotations.json parses as valid JSON and covers every metabolite/adduct pair in the fixed query list
- Queried monoisotopic masses and adduct forms match the values fixed in the initial query list, with no ad-hoc substitutions

## Limitations

- Self-hosting CMM requires access to an internal CEU database (requested via email), a configured app-server data-source, and the javax.faces-2.2.12.jar dependency
- 681,198 of CMM 3.0's compounds are predicted metabolites rather than experimental ones, so mass/formula records for these are more likely to fail cross-checks
- Verification is limited to metabolites also present in HMDB; CMM-specific entries (e.g., the in-house oxidized lipid library) cannot be reconciled against public records
- The README documents only deployment (database access, data-source, dependency jar) — no API parameter or response-schema documentation is provided

## Evidence
