---
name: metabolite-candidate-score-normalization
description: Use when you have a list of monoisotopic m/z candidates matched against
  a metabolite database (e.g., HMDB) plus an experimental setup (ionization mode,
  adduct list) and need to filter, score, and rank the candidate metabolites before
  annotation.
when_to_use_negative:
- You already have MS2 spectra and need spectral comparison or spectral quality assessment
  rather than MS1 candidate filtering/scoring (CMM 3.0 exposes those as separate services)
- Your input is raw spectral data and no database search candidates have been generated
  yet — candidate m/z matching against compound masses must precede this skill
- You need to annotate oxidized glycerophosphocholines from the CMM in-house oxidized
  lipid library rather than score general HMDB query candidates
edam_operation: http://edamontology.org/operation_3214
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: CEU Mass Mediator (CMM) 3.0
  role: Online metabolite annotation tool that unifies compounds from metabolomic
    databases and, via its expert system, filters and scores query results; hosts
    the workflow this skill reproduces
  repo: http://ceumass.eps.uspceu.es/
- name: CMM 2.0
  role: Version whose expert system with knowledge about the experimental setup and
    compound properties performs the filtering and scoring of query results
- name: HMDB (Human Metabolome Database)
  role: Source of the monoisotopic compound masses queried; CMM's RESTful API is integrated
    into HMDB
- name: ceuMassMediator
  role: Source code of the CEU Mass Mediator metabolite annotation tool; deployment
    requires CEU internal database access and a data-source configured with provided
    credentials
  repo: https://github.com/albertogilf/ceuMassMediator
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1021/acs.jproteome.8b00720
    title: 'CEU Mass Mediator 3.0: A Metabolite Annotation Tool'
  source_content_hash: sha256:29563ac42be7c04b4f01790eaf7f55dd48877eb94ac7bd4db5029a2671a7ee69
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/albertogilf/ceuMassMediator
  tools:
  - CEU Mass Mediator (CMM) 3.0
  - CMM 2.0
  - HMDB (Human Metabolome Database)
  - ceuMassMediator
  edam_operation: http://edamontology.org/operation_3214
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
emails_redacted: true
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# metabolite-candidate-score-normalization

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Applies CMM-style expert-system knowledge (ionization mode, allowed adducts, compound properties) to filter metabolite candidates returned from database queries such as HMDB, and computes rule-based plausibility scores normalized to the 0–1 range so researchers can prioritize annotations.

## When to use

Use when you have a list of monoisotopic m/z candidates matched against a metabolite database (e.g., HMDB) plus an experimental setup (ionization mode, adduct list) and need to filter, score, and rank the candidate metabolites before annotation.

## When NOT to use

- You already have MS2 spectra and need spectral comparison or spectral quality assessment rather than MS1 candidate filtering/scoring (CMM 3.0 exposes those as separate services)
- Your input is raw spectral data and no database search candidates have been generated yet — candidate m/z matching against compound masses must precede this skill
- You need to annotate oxidized glycerophosphocholines from the CMM in-house oxidized lipid library rather than score general HMDB query candidates

## Inputs

- Set of monoisotopic compound masses downloaded from HMDB (Human Metabolome Database)
- Experimental setup specification: ionization mode and allowed adducts
- Database-search candidate list (query m/z matched against compound masses)

## Outputs

- scored_candidates.csv with one row per candidate, its filter decision (accepted/rejected), and a plausibility score normalized to the 0–1 range
- Ranked list of candidate metabolite annotations

## How to apply

Start from the fixed query set of monoisotopic compound masses (e.g., downloaded from HMDB) and the experimental setup specification: the ionization mode and the allowed adducts. Generate candidates by matching each query m/z (neutral mass plus its adduct mass shift) against the database compound masses, mirroring how CMM 2.0 unifies compounds from multiple metabolomic databases. Then apply expert-system filter rules that encode knowledge about the experimental setup and compound properties to mark each candidate as accepted or rejected. Compute a rule-based plausibility score for every surviving candidate and normalize all scores into the 0–1 range. Finally, emit one row per candidate with its filter decision and its normalized score (e.g., to scored_candidates.csv) so that ranked annotations can be prioritized.

## Related tools

- **CEU Mass Mediator (CMM) 3.0** (Online metabolite annotation tool that unifies compounds from metabolomic databases and, via its expert system, filters and scores query results; hosts the workflow this skill reproduces) — http://ceumass.eps.uspceu.es/
- **CMM 2.0** (Version whose expert system with knowledge about the experimental setup and compound properties performs the filtering and scoring of query results)
- **HMDB (Human Metabolome Database)** (Source of the monoisotopic compound masses queried; CMM's RESTful API is integrated into HMDB)
- **ceuMassMediator** (Source code of the CEU Mass Mediator metabolite annotation tool; deployment requires CEU internal database access and a data-source configured with provided credentials) — https://github.com/albertogilf/ceuMassMediator

## Evaluation signals

- Every candidate row in scored_candidates.csv carries an explicit filter decision (accepted or rejected) plus a plausibility score
- All scores fall within the normalized 0–1 range
- Rejected candidates are consistent with the encoded expert-system rules about the experimental setup (ionization mode) and allowed adducts
- Ranking by normalized score is monotonic: higher-plausibility candidates precede lower ones in the output ordering
- Each query m/z maps to candidates via neutral mass plus the adduct mass shift, matching the query set from HMDB monoisotopic masses

## Limitations

- Deploying the CMM source code requires access to an internal CEU database (requested via email) and configuration of a data-source on the app server with provided credentials
- The app server requires the javax.faces-2.2.12.jar library
- Filtering and scoring depend on the quality of the input experimental setup specification (ionization mode and adduct list); incorrect setup yields invalid filter decisions
- The skill covers MS1 candidate filtering/scoring only; MS2 spectral comparison and spectrum reliability assessment are separate CMM 3.0 services

## Evidence
