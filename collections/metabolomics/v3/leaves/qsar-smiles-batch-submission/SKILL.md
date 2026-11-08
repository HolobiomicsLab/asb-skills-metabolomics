---
name: qsar-smiles-batch-submission
description: Use when a chemical inventory (with QSAR-ready SMILES, e.g. from HExpMetDB)
  lacks experimental toxicity values and predicted rat oral LD50 endpoints are needed
  for hazard or ToxPi-style risk ranking.
when_to_use_negative:
- The substance contains a metal atom or has molecular weight >1,000 — the article
  notes such chemicals fall outside prediction coverage (stated for the IFS HLB approach);
  TEST predictions may likewise be unavailable.
- Experimental in vivo LD50 or chronic/subchronic toxicity data are already available
  for the chemicals — measured values should be preferred, since the article notes
  the LD50 approach may underestimate chronic and subchronic toxicity risk.
- The endpoint needed is chronic toxicity or internal exposure (IVIVE) rather than
  acute rat oral LD50.
edam_topics:
- http://edamontology.org/topic_2815
tools:
- name: TEST (Toxicity Estimation Software Tool)
  role: QSAR engine that predicts rat oral LD50 from submitted SMILES; output is parsed
    into the CASRN-to-LD50 table
- name: U.S. EPA Chemistry Dashboard
  role: Source of chemical domain data keyed by CASRN; chemicals without CASRN cannot
    retrieve data
- name: HExpMetDB
  role: Provides the compiled list of >20,000 human exposome chemicals with QSAR-ready
    SMILES submitted to TEST
  repo: github.com/FangLabNTU/HExpMetDB
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1289/ehp7722
    title: Risk-Based Chemical Ranking and Generating a Prioritized Human Exposome
      Database
  source_content_hash: sha256:2a7845a8e5fba43f3704ede2b36aac42c698e996053fdfdb5c6ef6bab26db968
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - HExpMetDB
  - TEST
  - TEST (Toxicity Estimation Software Tool)
  - U.S. EPA Chemistry Dashboard
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_2815
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9285714285714286
n_claims: 14
---

# qsar-smiles-batch-submission

## Summary

Batch-submits QSAR-ready SMILES for a compiled chemical list to U.S. EPA TEST (v4.2.1) to obtain predicted rat oral LD50 values, producing a CASRN-to-LD50 mapping used for risk-based prioritization of human exposome chemicals.

## When to use

Use when a chemical inventory (with QSAR-ready SMILES, e.g. from HExpMetDB) lacks experimental toxicity values and predicted rat oral LD50 endpoints are needed for hazard or ToxPi-style risk ranking.

## When NOT to use

- The substance contains a metal atom or has molecular weight >1,000 — the article notes such chemicals fall outside prediction coverage (stated for the IFS HLB approach); TEST predictions may likewise be unavailable.
- Experimental in vivo LD50 or chronic/subchronic toxicity data are already available for the chemicals — measured values should be preferred, since the article notes the LD50 approach may underestimate chronic and subchronic toxicity risk.
- The endpoint needed is chronic toxicity or internal exposure (IVIVE) rather than acute rat oral LD50.

## Inputs

- Chemical inventory with QSAR-ready SMILES and identifiers (e.g. CASRN) drawn from HExpMetDB (github.com/FangLabNTU/HExpMetDB)
- U.S. EPA TEST (v4.2.1) installed and configured with the rat oral LD50 prediction module

## Outputs

- Table mapping chemical identifiers (CASRN) to predicted rat oral LD50 values
- Coverage report of chemicals with successful TEST predictions vs. those without (e.g. 14,786 predicted / 5,970 unpredicted of 20,756)

## How to apply

Retrieve the compiled chemical list with QSAR-ready SMILES from the HExpMetDB resource (github.com/FangLabNTU/HExpMetDB), then submit each chemical's SMILES to U.S. EPA TEST (v4.2.1) to obtain rat oral LD50 predictions. Parse each TEST output and assemble a table mapping chemical identifiers such as CASRN to the predicted LD50 value. Spot-check reference chemicals against known values — bisphenol A 3,247 mg/kg BW, DEHP 29,975 mg/kg BW, PFOA 176 mg/kg BW — and confirm the lowest predicted LD50 (1.78E-7 mol/kg BW for 2,3-dibromo-7,8-dichlorodibenzo-p-dioxin). Accept the batch only if predictions are obtained for the bulk of submissions (in this study, 14,786 of 20,756 substances) and spot-checked values agree within expected tool-to-reference tolerance.

## Related tools

- **TEST (Toxicity Estimation Software Tool)** (QSAR engine that predicts rat oral LD50 from submitted SMILES; output is parsed into the CASRN-to-LD50 table)
- **U.S. EPA Chemistry Dashboard** (Source of chemical domain data keyed by CASRN; chemicals without CASRN cannot retrieve data)
- **HExpMetDB** (Provides the compiled list of >20,000 human exposome chemicals with QSAR-ready SMILES submitted to TEST) — github.com/FangLabNTU/HExpMetDB

## Evaluation signals

- Coverage check: TEST predictions obtained for at least 14,786 of the 20,756 submitted substances (this study's benchmark).
- Reference spot-check: predicted LD50 values agree with reported values for BPA (3,247 mg/kg BW), DEHP (29,975 mg/kg BW), and PFOA (176 mg/kg BW) within tool-to-reference tolerance.
- Minimum-value sanity check: lowest predicted LD50 matches 1.78E-7 mol/kg BW for 2,3-dibromo-7,8-dichlorodibenzo-p-dioxin.
- Schema check: every row in the output table maps a chemical identifier (CASRN) to a predicted LD50 value with a traceable SMILES submission record.
- Non-null accounting: the count of substances lacking TEST predictions is explicitly reported (5,970 in this study) rather than silently dropped.

## Limitations

- TEST could not produce predictions for 5,970 of the 20,756 substances, so coverage is incomplete.
- Predicted LD50 reflects acute rat oral toxicity and may underestimate the risk of chronic and subchronic toxicity.
- The downstream HTS/prioritization workflow based on the U.S. EPA Chemistry Dashboard only works for chemicals with CASRN.
- Only chemicals with valid QSAR-ready SMILES can be submitted; chemicals with metal atoms or MW >1,000 are outside the scope of the related IFS prediction approach in the same pipeline.

## Evidence

- [discussion] predicted oral LD50 values for rats by TEST were used in the present study: "predicted oral LD50 values for rats by TEST were used in the present study"
- [abstract] toxicity data predicted by the Toxicity Estimation Software Tool: "toxicity data predicted by the Toxicity Estimation Software Tool"
- [discussion] only chemicals with CASRN were successful in retrieving diverse types of relevant domain data: "only chemicals with CASRN were successful in retrieving diverse types of relevant domain data from the U.S. EPA Chemistry Dashboard"
- [discussion] may underestimate the risk of the chemicals for their chronic and subchronic toxicity: "may underestimate the risk of the chemicals for their chronic and subchronic toxicity"
