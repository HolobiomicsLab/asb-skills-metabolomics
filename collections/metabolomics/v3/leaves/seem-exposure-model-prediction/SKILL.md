---
name: seem-exposure-model-prediction
description: Use when a compiled chemical list (e.g. CASRN-annotated exposome substances)
  needs predicted population exposure/intake rates to support risk-based ranking or
  coverage assessment against a consensus exposure model.
when_to_use_negative:
- Chemicals lacking a CASRN — the Dashboard-based HTS retrieval only succeeds for
  chemicals with CASRN
- The goal is measured internal exposure or biomarker concentrations rather than model-predicted
  external intake rates
- A compound-level exposure estimate is required for every substance — the SEEM database
  was unable to cover all compounds in the study
edam_operation: http://edamontology.org/operation_0334
edam_topics:
- http://edamontology.org/topic_3407
tools:
- name: U.S. EPA Chemistry Dashboard
  role: Source for retrieving SEEM consensus exposure predictions and domain data
    by CASRN
- name: ExpoCast
  role: Exposure information program underlying the exposure modeling context (Exposure
    Forecasting program)
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1289/ehp7722
    title: Risk-Based Chemical Ranking and Generating a Prioritized Human Exposome
      Database
  source_content_hash: sha256:14b1959fb6a4bc0239ffe7aa9ec9630850e5809982734e7caf2f917dc9449878
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ExpoCast
  - U.S. EPA Chemistry Dashboard
  edam_operation: http://edamontology.org/operation_0334
  edam_topics:
  - http://edamontology.org/topic_3407
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# seem-exposure-model-prediction

## Summary

Applies the SEEM (Systematic Empirical Evaluation of Models) consensus exposure model (Ring et al. 2019, R2 ≈ 0.8) to assign predicted median intake rates (mg/kg BW per day) with 95% confidence intervals to human exposome chemicals, enabling exposure-based prioritization in HExpMetDB.

## When to use

Use when a compiled chemical list (e.g. CASRN-annotated exposome substances) needs predicted population exposure/intake rates to support risk-based ranking or coverage assessment against a consensus exposure model.

## When NOT to use

- Chemicals lacking a CASRN — the Dashboard-based HTS retrieval only succeeds for chemicals with CASRN
- The goal is measured internal exposure or biomarker concentrations rather than model-predicted external intake rates
- A compound-level exposure estimate is required for every substance — the SEEM database was unable to cover all compounds in the study

## Inputs

- Compiled human exposome chemical list with CASRN/structure identifiers (e.g. HExpMetDB, 20,756 substances)
- SEEM consensus model predictions (Ring et al. 2019) accessible via the U.S. EPA Chemistry Dashboard

## Outputs

- exposure_predictions.csv with median predicted intake rate and 95% CI bounds per substance (mg/kg BW per day)
- Coverage summary (e.g. 15,408 of 20,756 substances matched) and list of unmatched/missing substances
- Spot-check results against reference values (BPA, DEHP, PFOA) and identification of highest-exposure chemicals (e.g. dihexyl nonanedioate)

## How to apply

Match each chemical in the compiled exposome list (e.g. the 20,756 HExpMetDB substances) by CASRN/structure against the SEEM consensus model, retrieving values via the U.S. EPA Chemistry Dashboard. Apply SEEM to each matched substance to obtain predicted median intake rates (mg/kg BW per day) and 95% confidence intervals. Record coverage statistics and flag unmatched substances as missing — in the source study, 15,408 of 20,756 substances received predictions, spanning 15 orders of magnitude. Spot-check known values (BPA = 5.50×10−5, DEHP = 2.72×10−3, PFOA = 5.47×10−8 mg/kg BW per day) before accepting the output table. Save the full median/CI table (exposure_predictions.csv) with coverage summary and spot-check results, noting that SEEM-derived exposure is the exposure input integrated with TEST-predicted toxicity and ToxCast ToxPi scores for chemical prioritization.

## Related tools

- **U.S. EPA Chemistry Dashboard** (Source for retrieving SEEM consensus exposure predictions and domain data by CASRN)
- **ExpoCast** (Exposure information program underlying the exposure modeling context (Exposure Forecasting program))

## Evaluation signals

- Coverage count matches expected fraction (15,408 of 20,756 substances in the source study); unmatched substances flagged as missing
- Spot-check substances reproduce reference values: BPA = 5.50×10−5, DEHP = 2.72×10−3, PFOA = 5.47×10−8 mg/kg BW per day
- Predicted exposures span ~15 orders of magnitude; no negative or non-finite median values in the output table
- Highest predicted exposure assigned to dihexyl nonanedioate; every output row has both median and 95% CI bounds

## Limitations

- The SEEM database was unable to cover all the compounds summarized in the present study, so predictions are incomplete (74% coverage in the source study)
- SEEM predictions are model-based external exposures; the risk score based on external exposure means the IVIVE model should be considered in future studies
- Only chemicals with CASRN were successful in retrieving diverse types of relevant domain data from the U.S. EPA Chemistry Dashboard

## Evidence

- [abstract] Prioritized chemicals by integrating exposure level (SEEM) with predicted toxicity data: "chemicals were prioritized by integrating exposure level obtained from the Systematic Empirical Evaluation of Models with toxicity data predicted by the Toxicity Estimation Software Tool"
- [discussion] Only CASRN-bearing chemicals retrievable: "only chemicals with CASRN were successful in retrieving diverse types of relevant domain data from the U.S. EPA Chemistry Dashboard"
- [discussion] SEEM coverage limitation: "the SEEM database was unable to cover all the compounds summarized in the present study"
- [intro] Exposure source program: "the U.S. EPA initiated the Exposure Forecasting (ExpoCast) program, which contained information on exposure"
