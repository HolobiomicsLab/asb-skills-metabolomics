---
name: toxpi-score-computation
description: Use when a risk-based chemical prioritization task requires combining
  multiple in vitro HTS assay endpoints and chemical properties (log P, BCF) into
  a single transparent, slice-based toxicity ranking score.
when_to_use_negative:
- The chemical of interest lacks a CASRN — the U.S. EPA Chemistry Dashboard pipeline
  only retrieves data for chemicals with CASRN.
- The prioritization goal is exposure-based or risk-index-based rather than hazard/toxicity-based
  — the broader article used SEEM exposure and probabilistic hazard quotient methods
  for that purpose, not ToxPi alone.
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_3379
tools:
- name: ToxPI
  role: ToxPi GUI 2.0 computes ToxPi scores by integrating HTS assay data, log P,
    and BCF into composite toxicity slices
- name: U.S. EPA Chemistry Dashboard
  role: Source of ToxCast AC50/efficacy HTS data plus log P and BCF values for chemicals
    with CASRN
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1289/ehp7722
    title: Risk-Based Chemical Ranking and Generating a Prioritized Human Exposome
      Database
  source_content_hash: sha256:8feda53d24a39dceee966923878f9d7026826b5c2f2429eeaacf262a28d3c7f3
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ToxPI
  - U.S. EPA Chemistry Dashboard
  edam_operation: http://edamontology.org/operation_2990
  edam_topics:
  - http://edamontology.org/topic_3379
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# toxpi-score-computation

## Summary

Compute Toxicological Priority Index (ToxPi) scores by integrating ToxCast high-throughput screening assay data (AC50, efficacy) with physicochemical descriptors (log P) and bioconcentration factors (BCF) into composite toxicity slices, then rank chemicals with bootstrapped 95% confidence intervals.

## When to use

Use when a risk-based chemical prioritization task requires combining multiple in vitro HTS assay endpoints and chemical properties (log P, BCF) into a single transparent, slice-based toxicity ranking score.

## When NOT to use

- The chemical of interest lacks a CASRN — the U.S. EPA Chemistry Dashboard pipeline only retrieves data for chemicals with CASRN.
- The prioritization goal is exposure-based or risk-index-based rather than hazard/toxicity-based — the broader article used SEEM exposure and probabilistic hazard quotient methods for that purpose, not ToxPi alone.

## Inputs

- ToxCast AC50 and efficacy data for 97 in vitro HTS assays
- log P and BCF values from the U.S. EPA Chemistry Dashboard (chemicals identified by CASRN)
- Chemical list filtered to those with HTS data (8,845 chemicals)

## Outputs

- Ranked ToxPi scores with bootstrap 95% confidence intervals for all chemicals (toxpi_scores.csv)

## How to apply

Retrieve ToxCast AC50 and efficacy data for the in vitro HTS assays (97 assays in this study) and log P / BCF values from the U.S. EPA Chemistry Dashboard, noting that only chemicals with CASRN can be retrieved. Filter the chemical list to those with HTS data (8,845 chemicals here). Use ToxPi GUI 2.0 to compute ToxPi scores by integrating the HTS assay data, log P, and BCF into composite toxicity slices, so that the Toxicological Priority Index calculated from the ToxCast database comprehensively estimates the toxicity ranking. Rank chemicals by score and bootstrap 95% confidence intervals for each score, saving the ranked output with CIs (e.g., as toxpi_scores.csv).

## Related tools

- **ToxPI** (ToxPi GUI 2.0 computes ToxPi scores by integrating HTS assay data, log P, and BCF into composite toxicity slices)
- **U.S. EPA Chemistry Dashboard** (Source of ToxCast AC50/efficacy HTS data plus log P and BCF values for chemicals with CASRN)

## Evaluation signals

- Scores are computed for exactly 8,845 chemicals (the full filtered set with HTS data).
- All ToxPi scores fall within the expected range 0.00–0.50.
- Top-ranked chemicals match known actives: 17α-ethinyl estradiol (EE2) and tributyltin chloride at 0.50.
- Bootstrap 95% CIs accompany each score (e.g., EE2 0.22–0.62; tributyltin chloride 0.26–0.67).
- Every scored chemical has a CASRN, since data retrieval from the Chemistry Dashboard requires it.

## Limitations

- HTS retrieval is limited to chemicals with CASRN — chemicals without CASRN cannot obtain diverse domain data from the U.S. EPA Chemistry Dashboard.
- ToxPi scores were bounded at 0.50 in this dataset, so the top of the ranking saturates and ties (e.g., EE2 and tributyltin chloride) must be interpreted with the confidence intervals.
- ToxPi ranking integrates the HTS data with predicted/curated log P and BCF, so missing descriptor values reduce coverage of the chemical list.

## Evidence

- [methods] workflow step 3: [evidence span withheld — anchor status: not_found]
- [discussion] discussion tool evidence: "we further calculated the ToxPi score in order to more comprehensively estimate the toxicity ranking of chemicals"
- [discussion] discussion dataset evidence: "our HTS method is based on the U.S. EPA Chemistry Dashboard, only chemicals with CASRN were successful in retrieving diverse types of relevant domain data"
- [abstract] abstract dataset evidence: "the Toxicological Priority Index calculated from the ToxCast database"
