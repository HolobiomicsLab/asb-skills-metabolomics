---
name: rat-oral-ld50-prediction
description: Use when a large compiled chemical list (with QSAR-ready SMILES and CASRNs,
  e.g. from HExpMetDB) lacks acute mammalian toxicity values and rat oral LD50 predictions
  are needed for hazard or risk-prioritization.
when_to_use_negative:
- The endpoint needed is chronic or subchronic toxicity — the article's discussion
  notes the LD50 approach 'may underestimate the risk of the chemicals for their chronic
  and subchronic toxicity'
- The input list is small and measured experimental LD50 data are available — measured
  values are preferable to TEST predictions where they exist
- The task requires coverage of every compound — TEST failed to yield predictions
  for 5,970 of the 20,756 submitted substances
tools:
- name: TEST (Toxicity Estimation Software Tool)
  role: Predicts rat oral LD50 values from submitted chemical SMILES; endpoint used
    in the study's toxicity ranking
- name: U.S. EPA Chemistry Dashboard
  role: Underlying Chemistry Dashboard-based HTS route for retrieving domain data;
    only chemicals with CASRN were successful
- name: HExpMetDB
  role: Source of the compiled chemical list with QSAR-ready SMILES submitted to TEST
  repo: github.com/FangLabNTU/HExpMetDB
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1289/ehp7722
    title: Risk-Based Chemical Ranking and Generating a Prioritized Human Exposome
      Database
  source_content_hash: sha256:9fe76a40f86e7b748a0d77544f28369742c8fec74789ef5e4b11c89cefc3911b
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - HExpMetDB
  - TEST
  - TEST (Toxicity Estimation Software Tool)
  - U.S. EPA Chemistry Dashboard
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# rat-oral-ld50-prediction

## Summary

Batch-predict rat oral LD50 toxicity values by submitting QSAR-ready SMILES from the HExpMetDB exposome chemical list (20,756 substances) to U.S. EPA TEST (v4.2.1), yielding in-silico acute-toxicity values for risk-based chemical ranking.

## When to use

Use when a large compiled chemical list (with QSAR-ready SMILES and CASRNs, e.g. from HExpMetDB) lacks acute mammalian toxicity values and rat oral LD50 predictions are needed for hazard or risk-prioritization.

## When NOT to use

- The endpoint needed is chronic or subchronic toxicity — the article's discussion notes the LD50 approach 'may underestimate the risk of the chemicals for their chronic and subchronic toxicity'
- The input list is small and measured experimental LD50 data are available — measured values are preferable to TEST predictions where they exist
- The task requires coverage of every compound — TEST failed to yield predictions for 5,970 of the 20,756 submitted substances

## Inputs

- Compiled chemical list with QSAR-ready SMILES and CASRNs (e.g., 20,756-substance human exposome database from HExpMetDB, github.com/FangLabNTU/HExpMetDB)
- U.S. EPA TEST (v4.2.1) tool installation or U.S. EPA Chemistry Dashboard access

## Outputs

- Table mapping chemical identifiers (CASRN) to predicted rat oral LD50 values (mg/kg BW)
- Coverage report: predicted LD50s for 14,786 of 20,756 substances; 5,970 substances without TEST predictions

## How to apply

Retrieve the compiled chemical list with QSAR-ready SMILES from the HExpMetDB resource (github.com/FangLabNTU/HExpMetDB) and submit each chemical's SMILES to U.S. EPA TEST (v4.2.1) to obtain rat oral LD50 predictions. Parse the TEST output and assemble a table mapping chemical identifiers (e.g., CASRN) to predicted LD50 values, recording that predictions fail for some substances (5,970 of 20,756 in the source study). Verify results by spot-checking reference values: bisphenol A 3,247 mg/kg BW, DEHP 29,975 mg/kg BW, PFOA 176 mg/kg BW, and the lowest predicted LD50 of 1.78E-7 mol/kg BW for 2,3-dibromo-7,8-dichlorodibenzo-p-dioxin. The rationale is that TEST-derived oral LD50 values feed downstream ToxPi scoring and probabilistic hazard quotient calculations, but the LD50 endpoint approximates only acute toxicity.

## Related tools

- **TEST (Toxicity Estimation Software Tool)** (Predicts rat oral LD50 values from submitted chemical SMILES; endpoint used in the study's toxicity ranking)
- **U.S. EPA Chemistry Dashboard** (Underlying Chemistry Dashboard-based HTS route for retrieving domain data; only chemicals with CASRN were successful)
- **HExpMetDB** (Source of the compiled chemical list with QSAR-ready SMILES submitted to TEST) — github.com/FangLabNTU/HExpMetDB

## Evaluation signals

- Coverage check: predictions available for at least 14,786 of the 20,756 submitted substances, with the remaining 5,970 failures logged
- Spot-check: TEST prediction for bisphenol A agrees with the reported 3,247 mg/kg BW within expected tool-to-reference tolerance
- Spot-check: DEHP prediction agrees with 29,975 mg/kg BW and PFOA with 176 mg/kg BW
- Most-toxic invariant: the lowest predicted LD50 is 1.78E-7 mol/kg BW for 2,3-dibromo-7,8-dichlorodibenzo-p-dioxin
- Schema check: every prediction row carries a chemical identifier (e.g., CASRN) linked to a QSAR-ready SMILES from the source list

## Limitations

- LD50-based risk scores may underestimate the risk of chemicals for their chronic and subchronic toxicity
- TEST could not produce predictions for 5,970 of 20,756 substances in the human exposome database
- Chemistry Dashboard-derived retrieval only succeeds for chemicals with CASRN, limiting identifier coverage
- Predicted values are in-silico estimates and should be spot-checked against reference LD50s (BPA, DEHP, PFOA) before use in ranking

## Evidence

- [discussion] TEST predictions used: "predicted oral LD50 values for rats by TEST were used in the present study"
- [other] Coverage and spot-check reference values: [evidence span withheld — anchor status: not_found]
- [other] Coverage expectation: [evidence span withheld — anchor status: not_found]
- [discussion] CASRN limitation: "only chemicals with CASRN were successful in retrieving diverse types of relevant domain data from the U.S. EPA Chemistry Dashboard"
- [discussion] LD50 endpoint limitation: "may underestimate the risk of the chemicals for their chronic and subchronic toxicity"
