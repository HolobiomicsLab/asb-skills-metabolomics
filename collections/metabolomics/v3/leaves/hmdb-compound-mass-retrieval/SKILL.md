---
name: hmdb-compound-mass-retrieval
description: Use when an MS1 metabolite-annotation workflow needs a reference library
  of monoisotopic compound masses retrieved from public HMDB to match query m/z values
  (neutral mass plus adduct shift) against.
when_to_use_negative:
- The task is MS2 spectral comparison or spectral quality assessment — CMM 3.0 provides
  dedicated services for those, distinct from MS1 mass-based candidate retrieval.
- 'Annotating oxidized glycerophosphocholines / oxidized lipids: CMM adds an in-house
  library of oxidized lipids ''not present in other sources'', so HMDB retrieval alone
  will miss them.'
- Input is already a scored/filtered candidate table — this skill only provisions
  the reference masses upstream of the expert-system filtering step.
edam_operation: http://edamontology.org/operation_2422
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3174
tools:
- name: HMDB (Human Metabolome Database)
  role: Public source of compound monoisotopic masses retrieved to build the reference
    mass list matched against query m/z values.
- name: CEU Mass Mediator (CMM) 3.0
  role: Runs the MS1 database search that consumes the HMDB-derived mass library together
    with ionization mode/adduct settings; exposes the RESTful API integrated into
    HMDB.
  repo: https://github.com/albertogilf/ceuMassMediator
- name: CMM 2.0
  role: Introduced the expert system that filters and scores the query results retrieved
    from the mass library using knowledge of the experimental setup and compound properties.
  repo: https://github.com/albertogilf/ceuMassMediator
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1021/acs.jproteome.8b00720
    title: 'CEU Mass Mediator 3.0: A Metabolite Annotation Tool'
  source_content_hash: sha256:3d5fbaff77bd8415090cf3df07a22fd5fea26629c2e42d1eb6ca94a322db3e79
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
  edam_operation: http://edamontology.org/operation_2422
  edam_topics:
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
emails_redacted: true
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4166666666666667
n_claims: 12
---

# hmdb-compound-mass-retrieval

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Retrieves monoisotopic compound masses from the public HMDB (Human Metabolome Database) to build the reference mass library that CEU Mass Mediator matches MS1 query m/z values (neutral mass plus adduct shift) against. This provisioning step feeds CMM's expert-system filtering and scoring of candidate metabolite annotations.

## When to use

Use when an MS1 metabolite-annotation workflow needs a reference library of monoisotopic compound masses retrieved from public HMDB to match query m/z values (neutral mass plus adduct shift) against.

## When NOT to use

- The task is MS2 spectral comparison or spectral quality assessment — CMM 3.0 provides dedicated services for those, distinct from MS1 mass-based candidate retrieval.
- Annotating oxidized glycerophosphocholines / oxidized lipids: CMM adds an in-house library of oxidized lipids 'not present in other sources', so HMDB retrieval alone will miss them.
- Input is already a scored/filtered candidate table — this skill only provisions the reference masses upstream of the expert-system filtering step.

## Inputs

- Public HMDB compound records with monoisotopic masses (reference compound mass list)
- Query m/z values from MS1 metabolomic runs
- Experimental setup specification: ionization mode and allowed adducts

## Outputs

- Reference library of monoisotopic compound masses consumed by CMM's database search
- Database-search candidate matches per query (compound, adduct), subsequently filtered and scored by CMM's expert system into normalized 0-1 plausibility scores

## How to apply

Retrieve monoisotopic compound masses from the public HMDB — CMM provides a RESTful API already integrated into HMDB, which is the natural retrieval channel — and assemble them as the fixed reference mass list for the database search. Pair the mass library with the experimental setup specification (ionization mode and the allowed adduct list), because CMM's expert system, introduced in version 2.0, encodes exactly this knowledge to filter and score the query results. Generate candidates by computing each query m/z as the neutral monoisotopic mass plus the adduct mass shift and matching it against the HMDB compound masses. Note that CMM 3.0 unifies sources up to 332,665 experimental and 681,198 predicted metabolites, so an HMDB-only mass list under-covers relative to CMM's integrated library, and compound classes such as the added oxidized lipids are absent from other sources. Validate retrieval by confirming each accepted candidate's mass reconciles with the declared ionization mode and adduct set before expert-system scoring assigns its normalized 0-1 plausibility score.

## Related tools

- **HMDB (Human Metabolome Database)** (Public source of compound monoisotopic masses retrieved to build the reference mass list matched against query m/z values.)
- **CEU Mass Mediator (CMM) 3.0** (Runs the MS1 database search that consumes the HMDB-derived mass library together with ionization mode/adduct settings; exposes the RESTful API integrated into HMDB.) — https://github.com/albertogilf/ceuMassMediator
- **CMM 2.0** (Introduced the expert system that filters and scores the query results retrieved from the mass library using knowledge of the experimental setup and compound properties.) — https://github.com/albertogilf/ceuMassMediator

## Evaluation signals

- Every retrieved HMDB entry contributes exactly one monoisotopic mass; the reference list size should reconcile with the HMDB release used (CMM 3.0 reports 332,665 experimental metabolites across its unified sources).
- For each candidate match, query m/z must equal the neutral monoisotopic mass plus the declared adduct's mass shift under the specified ionization mode.
- Expert-system filter decisions must stay consistent with the declared experimental setup: candidates implying adducts outside the allowed list should be rejected.
- Plausibility scores are normalized to the 0-1 range; no scored candidate should fall outside [0,1].
- A query repeated through the CMM RESTful API integrated into HMDB should return a candidate set consistent with the local HMDB-mass retrieval.

## Limitations

- The public source code cannot run standalone: deployment requires access to CEU's internal database (requested via <maintainer e-mail, see the repository README>), an app-server data-source with those credentials, and the javax.faces-2.2.12.jar library.
- HMDB coverage is incomplete for some compound classes — CMM added an in-house library of oxidized lipids 'not present in other sources', so retrieval from HMDB alone misses those.
- Coverage counts (332,665 experimental / 681,198 predicted metabolites) belong to CMM 3.0's unified library and differ from a raw HMDB-only mass list.

## Evidence
