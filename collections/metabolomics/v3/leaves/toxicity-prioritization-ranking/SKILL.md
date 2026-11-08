---
name: toxicity-prioritization-ranking
description: Use when a chemical inventory needs risk-based prioritization and in
  vitro HTS data (e.g., ToxCast AC50/efficacy for ~97 assays) plus physicochemical/endpoints
  (log P, BCF, LD50) are available per chemical with CASRN.
when_to_use_negative:
- Chemicals lack CASRN — the HTS retrieval workflow on the U.S. EPA Chemistry Dashboard
  only succeeds for chemicals with CASRN
- The analysis goal is biotransformation/metabolite prediction rather than hazard-based
  prioritization of parent chemicals
- Only exposure data are available with no toxicity endpoints (ToxPi slices require
  HTS/physicochemical toxicity inputs)
edam_operation: http://edamontology.org/operation_3791
edam_topics:
- http://edamontology.org/topic_2840
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3379
tools:
- name: ToxPI
  role: Compute composite ToxPi scores integrating HTS assay data, log P, and BCF
    into toxicity slices, with bootstrap confidence intervals
- name: U.S. EPA Chemistry Dashboard
  role: Retrieve ToxCast HTS assay data and log P / BCF values for chemicals by CASRN
- name: TEST
  role: Predict rat oral LD50 values used as toxicity input for prioritization
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1289/ehp7722
    title: Risk-Based Chemical Ranking and Generating a Prioritized Human Exposome
      Database
  source_content_hash: sha256:b4ee9d6e2682f4ff2ba6ab93bad839f15b447f771f42afbe8b7d84cac014b40f
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - TEST
  - ToxPI
  - U.S. EPA Chemistry Dashboard
  edam_operation: http://edamontology.org/operation_3791
  edam_topics:
  - http://edamontology.org/topic_2840
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3379
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6875
n_claims: 16
---

# toxicity-prioritization-ranking

## Summary

Composite toxicity ranking of environmental chemicals by integrating ToxCast in vitro HTS assay data (AC50, efficacy) with predicted toxicity (TEST oral LD50), log P, and BCF into ToxPi scores, yielding a ranked prioritization with bootstrap 95% confidence intervals.

## When to use

Use when a chemical inventory needs risk-based prioritization and in vitro HTS data (e.g., ToxCast AC50/efficacy for ~97 assays) plus physicochemical/endpoints (log P, BCF, LD50) are available per chemical with CASRN.

## When NOT to use

- Chemicals lack CASRN — the HTS retrieval workflow on the U.S. EPA Chemistry Dashboard only succeeds for chemicals with CASRN
- The analysis goal is biotransformation/metabolite prediction rather than hazard-based prioritization of parent chemicals
- Only exposure data are available with no toxicity endpoints (ToxPi slices require HTS/physicochemical toxicity inputs)

## Inputs

- Chemical list with CASRN identifiers
- ToxCast HTS assay data (AC50 and efficacy values) from the U.S. EPA Chemistry Dashboard
- log P and BCF values (U.S. EPA Chemistry Dashboard / TEST predictions)
- Predicted rat oral LD50 values from TEST (for chemicals lacking measured toxicity)
- Exposure estimates (e.g., SEEM) for risk-index integration (optional)

## Outputs

- Ranked ToxPi score table with bootstrap 95% confidence intervals (e.g., toxpi_scores.csv)
- Prioritized chemical list (13,441 chemicals by probabilistic hazard quotient; 7,770 by risk index in this study)

## How to apply

Retrieve ToxCast AC50 and efficacy data plus log P and BCF values from the U.S. EPA Chemistry Dashboard for each chemical (CASRN required); filter the inventory to chemicals with HTS data (8,845 in this study). Optionally supplement with predicted rat oral LD50 from TEST for chemicals lacking measured toxicity. Compute ToxPi scores in ToxPi GUI 2.0 by integrating the HTS assay data, log P, and BCF into composite toxicity slices, and optionally combine with SEEM-based exposure levels (probabilistic hazard quotient / risk index) for risk ranking. Rank chemicals by ToxPi score and estimate bootstrap 95% confidence intervals per score. In this study scores spanned 0.00–0.50, with 17α-ethinylestradiol and tributyltin chloride at the top (0.50; 95% CIs 0.22–0.62 and 0.26–0.67).

## Related tools

- **ToxPI** (Compute composite ToxPi scores integrating HTS assay data, log P, and BCF into toxicity slices, with bootstrap confidence intervals)
- **U.S. EPA Chemistry Dashboard** (Retrieve ToxCast HTS assay data and log P / BCF values for chemicals by CASRN)
- **TEST** (Predict rat oral LD50 values used as toxicity input for prioritization)

## Evaluation signals

- Scores are computed for exactly 8,845 chemicals when applied to the ToxCast-filtered inventory
- ToxPi scores span 0.00–0.50
- 17α-ethinylestradiol and tributyltin chloride rank highest at 0.50, with bootstrap 95% CIs of 0.22–0.62 and 0.26–0.67 respectively
- Every scored chemical has a CASRN and non-missing HTS assay data
- Output CSV contains scores paired with their 95% confidence intervals for all ranked chemicals

## Limitations

- HTS-based retrieval is limited to chemicals with CASRN; only those were successful in retrieving diverse types of relevant domain data from the U.S. EPA Chemistry Dashboard
- TEST-predicted oral LD50 values may underestimate the risk of chemicals for chronic and subchronic toxicity
- The risk score is based on external exposure; IVIVE-based internal exposure conversion should be considered, and chemical interaction (e.g., synergistic effects) in mixture toxicity cannot be excluded
- SEEM exposure estimates could not cover all compounds in the study

## Evidence

- [results] ToxPi scores were calculated for 8,845 chemicals spanning 0.00–0.50, with 17α-ethinylestradiol and tributyltin chloride attaining the highest ToxPi value of 0.50: [evidence span withheld — anchor status: not_found]
- [discussion] we further calculated the ToxPi score in order to more comprehensively estimate the toxicity ranking of chemicals: "we further calculated the ToxPi score in order to more comprehensively estimate the toxicity ranking of chemicals"
- [discussion] only chemicals with CASRN were successful in retrieving diverse types of relevant domain data: "our HTS method is based on the U.S. EPA Chemistry Dashboard, only chemicals with CASRN were successful in retrieving diverse types of relevant domain data"
- [discussion] predicted oral LD50 values for rats by TEST were used in the present study: "predicted oral LD50 values for rats by TEST were used in the present study"
- [abstract] prioritized 13,441 chemicals based on probabilistic hazard quotient and 7,770 chemicals based on risk index: "prioritized 13,441 chemicals based on probabilistic hazard quotient and 7,770 chemicals based on risk index"
