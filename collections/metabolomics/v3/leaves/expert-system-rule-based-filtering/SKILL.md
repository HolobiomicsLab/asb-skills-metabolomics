---
name: expert-system-rule-based-filtering
description: Use when you have m/z query results matched against a metabolomic database
  (e.g., HMDB) and need to filter and rank candidate metabolite annotations under
  a defined experimental setup (ionization mode, allowed adducts).
when_to_use_negative:
- You need MS2-based confirmation rather than MS1 candidate filtering — CMM 3.0's
  separate spectral-comparison and spectral quality-assessment services are the appropriate
  choice
- Input is already a ranked, scored annotation table; re-applying expert-system filtering
  would only re-rank existing results
- Candidate metabolites come from sources without defined adduct/ionization-mode metadata,
  since the rules depend on the experimental setup specification
edam_operation: http://edamontology.org/operation_3803
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_3520
tools:
- name: CEU Mass Mediator (CMM) 3.0
  role: Online metabolite annotation tool whose expert system performs the filtering
    and scoring of query results
  repo: https://github.com/albertogilf/ceuMassMediator
- name: CMM 2.0
  role: Version that unified compounds from multiple databases and introduced the
    expert-system filtering and scoring of query results
- name: HMDB (Human Metabolome Database)
  role: Source of monoisotopic compound masses used to generate database-search candidates;
    also consumes CMM's RESTful API
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1021/acs.jproteome.8b00720
    title: 'CEU Mass Mediator 3.0: A Metabolite Annotation Tool'
  source_content_hash: sha256:7bbe000fec889fc0ac4228ac8ab8f7925d6aff4123b0f19153e0c85d8b9fee5e
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
  - CMM 2.0
  - HMDB (Human Metabolome Database)
  edam_operation: http://edamontology.org/operation_3803
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
emails_redacted: true
evidence_trimmed: true
triage_status: critique
yes_rate: 0.45454545454545453
n_claims: 11
---

# expert-system-rule-based-filtering

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Applies CMM's expert-system rules — encoding knowledge of the experimental setup (ionization mode, adducts) and compound properties — to filter database-search candidate metabolites and assign rule-based plausibility scores, normalized to the 0–1 range, so annotations can be ranked.

## When to use

Use when you have m/z query results matched against a metabolomic database (e.g., HMDB) and need to filter and rank candidate metabolite annotations under a defined experimental setup (ionization mode, allowed adducts).

## When NOT to use

- You need MS2-based confirmation rather than MS1 candidate filtering — CMM 3.0's separate spectral-comparison and spectral quality-assessment services are the appropriate choice
- Input is already a ranked, scored annotation table; re-applying expert-system filtering would only re-rank existing results
- Candidate metabolites come from sources without defined adduct/ionization-mode metadata, since the rules depend on the experimental setup specification

## Inputs

- Query set of monoisotopic compound masses (e.g., downloaded from public HMDB)
- Experimental setup specification (ionization mode and allowed adducts)
- Database-search candidate list (query m/z matched against HMDB compound masses)

## Outputs

- scored_candidates.csv — one row per candidate with filter decision (accepted/rejected) and normalized plausibility score (0–1 range)

## How to apply

Load the fixed query set of monoisotopic compound masses (downloaded from public HMDB) together with the experimental setup specification (ionization mode and allowed adducts). Generate database-search candidates by matching each query m/z — the neutral mass plus the adduct mass shift — against the HMDB compound masses. Apply the expert-system filter rules, which encode knowledge about the experimental setup and the compounds' properties, to mark each candidate as accepted or rejected. Compute a rule-based plausibility score for each candidate and normalize all scores to the 0–1 range. Emit one row per candidate with its filter decision and normalized score so researchers can prioritize annotations; this is the mechanism CMM 2.0 introduced on top of simple database querying.

## Related tools

- **CEU Mass Mediator (CMM) 3.0** (Online metabolite annotation tool whose expert system performs the filtering and scoring of query results) — https://github.com/albertogilf/ceuMassMediator
- **CMM 2.0** (Version that unified compounds from multiple databases and introduced the expert-system filtering and scoring of query results)
- **HMDB (Human Metabolome Database)** (Source of monoisotopic compound masses used to generate database-search candidates; also consumes CMM's RESTful API)

## Evaluation signals

- Every candidate in scored_candidates.csv carries an explicit filter decision (accepted or rejected) traceable to a rule about the experimental setup or compound properties
- All plausibility scores fall within the 0–1 range after normalization
- The candidate m/z set matches neutral masses plus the adduct mass shifts specified in the experimental setup (e.g., ionization mode and allowed adducts)
- Rejected candidates can be rationalized by a rule grounded in compound properties, not arbitrary removal
- Output is ranked so researchers can prioritize annotations in descending score order

## Limitations

- The expert system depends on knowledge of the experimental setup (ionization mode, adducts); incorrect or missing setup specification will corrupt both filtering and scoring
- This skill filters and scores MS1 database-search candidates; it does not confirm identities, which requires CMM 3.0's separate MS2 spectral comparison and spectral quality-assessment services
- Deploying CMM locally requires access to an internal CEU database (requested via email) and a data-source configured on the app server, plus the javax.faces-2.2.12.jar library

## Evidence
