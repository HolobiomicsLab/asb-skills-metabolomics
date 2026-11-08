---
name: adduct-aware-ion-annotation
description: Use when assigning metabolite identities to MS1 m/z features by querying
  metabolomic databases such as HMDB, and candidates must be filtered and ranked according
  to ionization mode, adduct knowledge, and compound properties.
when_to_use_negative:
- Input is already a curated annotation table with identities assigned — re-running
  database querying and expert-system filtering would be redundant
- The task is MS2-only spectral library matching without any MS1 precursor m/z query
  against compound databases
- The compound space of interest is absent from CMM's integrated databases (e.g.,
  specialized lipids outside its libraries), since filtering and scoring operate only
  on retrieved query results
edam_operation: http://edamontology.org/operation_3803
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_3520
tools:
- name: CEU Mass Mediator (CMM) 3.0
  role: Online annotation tool that queries metabolomic databases and applies the
    expert system to filter and score MS1 candidates; exposes the RESTful API used
    for annotation
  repo: http://ceumass.eps.uspceu.es/
- name: CMM 2.0
  role: Version that introduced unification of compounds across databases and the
    expert-system filtering and scoring of query results
- name: HMDB (Human Metabolome Database)
  role: Source of monoisotopic compound masses used as the query library; CMM is integrated
    into HMDB via a RESTful API
- name: albertogilf/ceuMassMediator
  role: Source code for self-hosting the CMM annotation tool (requires javax.faces-2.2.12.jar
    on the app server)
  repo: https://github.com/albertogilf/ceuMassMediator
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1021/acs.jproteome.8b00720
    title: 'CEU Mass Mediator 3.0: A Metabolite Annotation Tool'
  source_content_hash: sha256:575ae114dfb2d324522b1800b8328c9906f1203dd09971662d2f93d097e69a7e
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
  - albertogilf/ceuMassMediator
  edam_operation: http://edamontology.org/operation_3803
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
emails_redacted: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# adduct-aware-ion-annotation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Expert-system-based annotation of MS1 metabolite candidates in which an expert system uses knowledge of the experimental setup (ionization mode, allowed adducts) and compound properties to filter and score database-query results, producing ranked metabolite annotations.

## When to use

Use when assigning metabolite identities to MS1 m/z features by querying metabolomic databases such as HMDB, and candidates must be filtered and ranked according to ionization mode, adduct knowledge, and compound properties.

## When NOT to use

- Input is already a curated annotation table with identities assigned — re-running database querying and expert-system filtering would be redundant
- The task is MS2-only spectral library matching without any MS1 precursor m/z query against compound databases
- The compound space of interest is absent from CMM's integrated databases (e.g., specialized lipids outside its libraries), since filtering and scoring operate only on retrieved query results

## Inputs

- List of query m/z values (MS1 features, monoisotopic peak-derived)
- Experimental setup specification: ionization mode (positive/negative) and allowed adducts
- Fixed library of monoisotopic compound masses downloaded from HMDB (or CMM's integrated experimental/predicted metabolite databases)
- Optional MS2 spectra for spectral comparison refinement

## Outputs

- Ranked list of candidate metabolite annotations per query m/z, with filter decision (accepted/rejected) and normalized 0–1 plausibility score
- scored_candidates.csv (one row per candidate with filter decision and normalized score)

## How to apply

Load the fixed query set of monoisotopic compound masses (e.g., downloaded from HMDB) together with the experimental setup specification: ionization mode and the allowed adducts. Generate database-search candidates by matching each query m/z — the neutral mass plus the appropriate adduct mass shift — against the compound masses in the queried metabolomic databases. Apply the expert-system filter rules, which encode knowledge about the experimental setup and the compounds' properties, to mark each candidate as accepted or rejected. Compute a rule-based plausibility score for each accepted candidate and normalize all scores to the 0–1 range so candidates can be prioritized. Write one row per candidate with its filter decision and normalized score, producing a ranked candidate annotation list. In CMM 3.0 this core MS1 annotation can be followed by MS2 spectral comparison and spectral quality assessment to further support identification.

## Related tools

- **CEU Mass Mediator (CMM) 3.0** (Online annotation tool that queries metabolomic databases and applies the expert system to filter and score MS1 candidates; exposes the RESTful API used for annotation) — http://ceumass.eps.uspceu.es/
- **CMM 2.0** (Version that introduced unification of compounds across databases and the expert-system filtering and scoring of query results)
- **HMDB (Human Metabolome Database)** (Source of monoisotopic compound masses used as the query library; CMM is integrated into HMDB via a RESTful API)
- **albertogilf/ceuMassMediator** (Source code for self-hosting the CMM annotation tool (requires javax.faces-2.2.12.jar on the app server)) — https://github.com/albertogilf/ceuMassMediator

## Evaluation signals

- Each candidate row in scored_candidates.csv carries an explicit filter decision (accepted/rejected) consistent with the declared ionization mode and adduct set
- All candidate plausibility scores fall within the normalized 0–1 range
- Query m/z values match database candidates only via declared adduct mass shifts (neutral mass + adduct shift), not arbitrary deltas
- Candidate rankings are monotone with normalized scores (higher score = higher priority annotation)
- Deployment self-check: the app server resolves the CEU internal data-source and javax.faces-2.2.12.jar is present, so database queries return results

## Limitations

- Self-hosting requires access to an internal CEU database, which must be requested via email, plus app-server data-source configuration with provided credentials
- Expert-system filtering and scoring only operate on compounds returned by the queried metabolomic databases; coverage is bounded by those sources (332,665 experimental and 681,198 predicted metabolites in CMM 3.0)
- MS1 filtering/scoring alone yields candidate rankings, not definitive identifications; MS2 spectral comparison and spectral quality assessment are needed to strengthen assignments

## Evidence

- [abstract] CMM 2.0 expert system description: "unifies the compounds from these databases and, using an expert system with knowledge about the experimental setup and the compounds properties, filters and scores the query results (CMM 2.0)"
- [intro] CMM as an annotation tool: "metabolite annotation tool called CEU Mass Mediator. This tool is available at http://ceumass.eps.uspceu.es/"
- [abstract] CMM integration into HMDB: "a RESTful API has been created, and it has already been integrated into the HMDB (Human Metabolome Database)"
- [readme] Database access requirement for self-hosted deployment: "It is necessary to access an internal database of CEU. Request access to <maintainer e-mail, see the repository README>."
- [abstract] Metabolite counts available in CMM 3.0: "has increased the number of experimental metabolites up 332,665 and the number of predicted metabolites to 681,198"
