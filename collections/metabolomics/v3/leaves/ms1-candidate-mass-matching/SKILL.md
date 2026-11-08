---
name: ms1-candidate-mass-matching
description: 'Use when an untargeted metabolomics workflow needs candidate metabolite
  annotations for MS1 peaks: given a list of monoisotopic m/z values, an ionization
  mode, and a set of allowed adducts, to produce a ranked candidate list.'
when_to_use_negative:
- 'When the goal is MS2-level confirmation: use CMM 3.0''s spectral comparison service
  for MS2 data or its spectral quality-assessment service instead of MS1 candidate
  matching.'
- When the compound of interest lies outside the queried databases' scope, e.g., oxidized
  lipids only present in CMM's in-house library of oxidized glycerophosphocholines
  and not in HMDB.
- When candidate ranking is already available from another annotation tool and no
  re-scoring against the experimental setup is needed.
edam_operation: http://edamontology.org/operation_3803
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: CEU Mass Mediator (CMM) 3.0
  role: Online metabolite annotation tool that performs the database queries, expert-system
    filtering, and scoring of MS1 candidates
  repo: https://github.com/albertogilf/ceuMassMediator
- name: CMM 2.0
  role: Version that unified compounds from multiple databases and introduced the
    expert-system filtering and scoring of query results
  repo: https://github.com/albertogilf/ceuMassMediator
- name: HMDB (Human Metabolome Database)
  role: Source of monoisotopic compound masses queried for candidate matching; CMM's
    RESTful API is integrated into HMDB
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1021/acs.jproteome.8b00720
    title: 'CEU Mass Mediator 3.0: A Metabolite Annotation Tool'
  source_content_hash: sha256:7942aa926de3a4db50114c4f721cf2da0ba924ab153d80b8a0d99e4107f64cd2
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
  - CEU Mass Mediator
  - CEU Mass Mediator (CMM) 3.0
  - CMM 2.0
  - HMDB (Human Metabolome Database)
  edam_operation: http://edamontology.org/operation_3803
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
emails_redacted: true
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# ms1-candidate-mass-matching

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Generates and prioritizes MS1 metabolite annotations by matching query m/z values (neutral mass plus adduct mass shifts) against compound databases such as HMDB, then applying CMM's expert-system filter and scoring rules to rank candidates.

## When to use

Use when an untargeted metabolomics workflow needs candidate metabolite annotations for MS1 peaks: given a list of monoisotopic m/z values, an ionization mode, and a set of allowed adducts, to produce a ranked candidate list.

## When NOT to use

- When the goal is MS2-level confirmation: use CMM 3.0's spectral comparison service for MS2 data or its spectral quality-assessment service instead of MS1 candidate matching.
- When the compound of interest lies outside the queried databases' scope, e.g., oxidized lipids only present in CMM's in-house library of oxidized glycerophosphocholines and not in HMDB.
- When candidate ranking is already available from another annotation tool and no re-scoring against the experimental setup is needed.

## Inputs

- List of query monoisotopic m/z values from MS1 peaks
- Experimental setup specification: ionization mode and allowed adduct list
- Monoisotopic compound masses from a metabolomic database (e.g., HMDB)

## Outputs

- One row per candidate with filter decision (accepted/rejected) and normalized plausibility score in the 0–1 range, e.g., scored_candidates.csv
- Ranked list of candidate metabolite annotations per query m/z

## How to apply

Start from the fixed query set of monoisotopic compound masses (e.g., compounds downloaded from public HMDB) together with the experimental setup specification — ionization mode and allowed adducts. Generate database-search candidates by matching each query m/z (neutral mass plus the adduct mass shift) against the compound masses in the queried metabolomic databases. CMM 2.0/3.0 then applies an expert system that encodes knowledge about the experimental setup (ionization mode, adducts) and compound properties to filter the retrieved compounds, marking each candidate as accepted or rejected. Compute a rule-based plausibility score for each surviving candidate and normalize all scores to the 0–1 range so candidates can be prioritized. Write one row per candidate with its filter decision and normalized score, yielding a ranked candidate-annotation table (e.g., scored_candidates.csv).

## Related tools

- **CEU Mass Mediator (CMM) 3.0** (Online metabolite annotation tool that performs the database queries, expert-system filtering, and scoring of MS1 candidates) — https://github.com/albertogilf/ceuMassMediator
- **CMM 2.0** (Version that unified compounds from multiple databases and introduced the expert-system filtering and scoring of query results) — https://github.com/albertogilf/ceuMassMediator
- **HMDB (Human Metabolome Database)** (Source of monoisotopic compound masses queried for candidate matching; CMM's RESTful API is integrated into HMDB)

## Evaluation signals

- Every candidate row carries an explicit filter decision (accepted or rejected) traceable to an expert-system rule about the ionization mode, adduct, or compound properties.
- All plausibility scores fall within the normalized 0–1 range.
- Each query m/z yields candidates consistent with the declared adduct mass shifts (neutral mass + adduct shift matches the query m/z within the search tolerance).
- Candidates rejected by the expert system are excluded from the top-ranked output, and accepted candidates are ordered by normalized score.
- Candidate counts are consistent with the database scope queried (e.g., CMM 3.0 covers 332,665 experimental and 681,198 predicted metabolites).

## Limitations

- Deployment of the CMM source requires access to an internal CEU database, which must be requested via email, plus configuration of a data-source on the app server with provided credentials.
- Filtering and scoring quality depends on the correctness of the declared experimental setup (ionization mode and adducts); incorrect adduct lists will wrongly accept or reject candidates.
- The scope is limited to the metabolites in the queried databases; compounds absent from public sources (e.g., certain oxidized lipids) require the CMM in-house library.

## Evidence
