---
name: toxicity-prediction-spot-check-verification
description: Use when batch toxicity predictions (e.g., LD50 from TEST) must be validated
  before acceptance — spot-checking reference chemicals with known LD50 values and
  confirming prediction coverage across the submitted set.
when_to_use_negative:
- The chemical lacks a CASRN, which prevents retrieving domain data from the U.S.
  EPA Chemistry Dashboard in the article's HTS workflow
- Assessing chronic or subchronic toxicity — the article notes the LD50 approach 'may
  underestimate the risk of the chemicals for their chronic and subchronic toxicity'
- The chemicals require measured (not predicted) toxicity values or experimental validation
  rather than in silico screening
edam_operation: http://edamontology.org/operation_0226
edam_topics:
- http://edamontology.org/topic_3379
tools:
- name: TEST
  role: U.S. EPA Toxicity Estimation Software Tool (v4.2.1) used to generate rat oral
    LD50 predictions submitted via SMILES
- name: U.S. EPA Chemistry Dashboard
  role: Source of domain data for chemicals with CASRN, used alongside TEST in the
    high-throughput workflow
- name: ToxPI
  role: ToxPi scores calculated from ToxCast data to complement TEST LD50 predictions
    in toxicity ranking
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1289/ehp7722
    title: Risk-Based Chemical Ranking and Generating a Prioritized Human Exposome
      Database
  source_content_hash: sha256:ebe7d2d4aba0a4944ff9336bb88955d3fbec2091adad2a658ae6286af718b2ed
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - TEST
  - ToxPI
  - Toxicity Estimation Software Tool
  - U.S. EPA Chemistry Dashboard
  edam_operation: http://edamontology.org/operation_0226
  edam_topics:
  - http://edamontology.org/topic_3379
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 14
---

# toxicity-prediction-spot-check-verification

## Summary

Verifies in silico toxicity predictions by spot-checking reference chemicals against known literature values, using U.S. EPA TEST rat oral LD50 predictions across a compiled chemical database (HExpMetDB, 20,756 substances) as the reference workflow.

## When to use

Use when batch toxicity predictions (e.g., LD50 from TEST) must be validated before acceptance — spot-checking reference chemicals with known LD50 values and confirming prediction coverage across the submitted set.

## When NOT to use

- The chemical lacks a CASRN, which prevents retrieving domain data from the U.S. EPA Chemistry Dashboard in the article's HTS workflow
- Assessing chronic or subchronic toxicity — the article notes the LD50 approach 'may underestimate the risk of the chemicals for their chronic and subchronic toxicity'
- The chemicals require measured (not predicted) toxicity values or experimental validation rather than in silico screening

## Inputs

- Compiled chemical list with QSAR-ready SMILES (HExpMetDB resource, 20,756 substances)
- U.S. EPA TEST (v4.2.1) rat oral LD50 prediction outputs
- Reference LD50 values for spot-check chemicals (BPA, DEHP, PFOA, 2,3-dibromo-7,8-dichlorodibenzo-p-dioxin)

## Outputs

- Table mapping chemical identifiers (CASRN) to predicted rat oral LD50 values for 14,786 substances
- Spot-check verification report comparing TEST predictions to reference LD50 values

## How to apply

Retrieve the QSAR-ready SMILES from the compiled chemical list (HExpMetDB, github.com/FangLabNTU/HExpMetDB) and submit each to U.S. EPA TEST (v4.2.1) for rat oral LD50 prediction. Parse TEST output and assemble a table mapping chemical identifiers (e.g., CASRN) to predicted LD50 values. Verify correctness by comparing spot-checked reference values: bisphenol A 3,247 mg/kg BW, DEHP 29,975 mg/kg BW, and PFOA 176 mg/kg BW, plus the lowest predicted LD50 of 1.78E-7 mol/kg BW for 2,3-dibromo-7,8-dichlorodibenzo-p-dioxin. Accept the prediction table only if predictions cover at least 14,786 of 20,756 submitted substances and spot-checked references agree within expected tool-to-reference tolerance.

## Related tools

- **TEST** (U.S. EPA Toxicity Estimation Software Tool (v4.2.1) used to generate rat oral LD50 predictions submitted via SMILES)
- **U.S. EPA Chemistry Dashboard** (Source of domain data for chemicals with CASRN, used alongside TEST in the high-throughput workflow)
- **ToxPI** (ToxPi scores calculated from ToxCast data to complement TEST LD50 predictions in toxicity ranking)

## Evaluation signals

- Prediction count invariant: rat oral LD50 values obtained for at least 14,786 of 20,756 submitted substances
- Spot-check BPA prediction agrees with 3,247 mg/kg BW within tool-to-reference tolerance
- Spot-check DEHP prediction agrees with 29,975 mg/kg BW
- Spot-check PFOA prediction agrees with 176 mg/kg BW
- Minimum predicted LD50 equals 1.78E-7 mol/kg BW, corresponding to 2,3-dibromo-7,8-dichlorodibenzo-p-dioxin

## Limitations

- TEST could not produce predictions for 5,970 of the 20,756 substances in the human exposome database
- LD50-based ranking may underestimate chronic and subchronic toxicity risk
- Only chemicals with CASRN were successful in retrieving diverse types of relevant domain data from the U.S. EPA Chemistry Dashboard

## Evidence

- [other] TEST LD50 predictions for 14,786/20,756 substances: [evidence span withheld — anchor status: not_found]
- [other] Reference spot-check values: [evidence span withheld — anchor status: not_found]
- [discussion] TEST used for rat oral LD50: "predicted oral LD50 values for rats by TEST were used in the present study"
- [discussion] ToxPi complements toxicity ranking: "we further calculated the ToxPi score in order to more comprehensively estimate the toxicity ranking of chemicals"
- [discussion] LD50 underestimates chronic risk: "may underestimate the risk of the chemicals for their chronic and subchronic toxicity"
