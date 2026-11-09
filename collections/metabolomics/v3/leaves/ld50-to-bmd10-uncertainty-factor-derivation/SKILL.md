---
name: ld50-to-bmd10-uncertainty-factor-derivation
description: Use when ranking thousands of exposome chemicals by risk and only predicted
  LD50 toxicity (no measured reference dose) is available, requiring LD50-to-BMD10
  conversion with uncertainty factors.
when_to_use_negative:
- Measured chronic/subchronic toxicity data or reference doses (e.g., RfD, BMD from
  guideline studies) are available — the LD50-based approach may underestimate chronic
  and subchronic toxicity risk
- The chemical lacks a CASRN and thus cannot be retrieved through the U.S. EPA Chemistry
  Dashboard-based HTS workflow
- The analysis requires internal (IVIVE-based) rather than external exposure — the
  PrHQ here is computed from external exposure only
edam_operation: http://edamontology.org/operation_2945
edam_topics: []
tools:
- name: TEST
  role: Supplies the predicted oral LD50 values for rats that are converted into probabilistic
    BMD10/HDMI values via the uncertainty factors
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1289/ehp7722
    title: Risk-Based Chemical Ranking and Generating a Prioritized Human Exposome
      Database
  source_content_hash: sha256:ab9c66d0814e0bf7aea4b559ea77878e6711fd8cfc1ac823af66fe8053382324
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - TEST
  edam_operation: http://edamontology.org/operation_2945
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# ld50-to-bmd10-uncertainty-factor-derivation

## Summary

Converts predicted rat oral LD50 values (from TEST) into probabilistic BMD10-derived reference doses by applying chemical-specific uncertainty factors (Equations 1–3) to obtain the human dose of minimal incidence (HDMI), enabling probabilistic hazard quotient (PrHQ) risk ranking across the human exposome database.

## When to use

Use when ranking thousands of exposome chemicals by risk and only predicted LD50 toxicity (no measured reference dose) is available, requiring LD50-to-BMD10 conversion with uncertainty factors.

## When NOT to use

- Measured chronic/subchronic toxicity data or reference doses (e.g., RfD, BMD from guideline studies) are available — the LD50-based approach may underestimate chronic and subchronic toxicity risk
- The chemical lacks a CASRN and thus cannot be retrieved through the U.S. EPA Chemistry Dashboard-based HTS workflow
- The analysis requires internal (IVIVE-based) rather than external exposure — the PrHQ here is computed from external exposure only

## Inputs

- TEST-predicted oral LD50 values for rats for each chemical (keyed by CASRN)
- SEEM exposure estimates (EHE) for each chemical in the human exposome database
- Uncertainty factor definitions from Equations (1)–(3) of EHP7722

## Outputs

- HDMI (human dose of minimal incidence) per chemical with associated uncertainty
- PrHQ (EHE/HDMI) per chemical
- prioritized_chemicals.csv: 13,441 chemicals ranked in descending PrHQ order with chemical identifiers and PrHQ values

## How to apply

First retrieve the TEST-predicted oral LD50 values for rats and the SEEM exposure estimates (EHE) for each chemical in the human exposome database. Convert each LD50 to a probabilistic BMD10-based probabilistic reference dose by applying the uncertainty factors defined in Equations (1)–(3) of the paper, yielding the human dose of minimal incidence (HDMI) per chemical. Compute PrHQ for every chemical as the ratio EHE/HDMI, propagating uncertainty so that each PrHQ is probabilistic rather than a point estimate. Rank all chemicals in descending order of PrHQ; in this application 13,441 chemicals received a ranking spanning 14 orders of magnitude (3.32×10−14 to 7.61×10^0), with DHND highest and N,N′-di-2-naphthyl-p-phenylenediamine (CASRN 93-46-9) lowest. Rationale: LD50 is the only toxicity endpoint available at exposome scale, so the probabilistic treatment of the LD50-to-BMD10 conversion substitutes for absent measured reference doses.

## Related tools

- **TEST** (Supplies the predicted oral LD50 values for rats that are converted into probabilistic BMD10/HDMI values via the uncertainty factors)

## Examples

```
df['PrHQ'] = df['EHE'] / df['HDMI']; df.sort_values('PrHQ', ascending=False).to_csv('prioritized_chemicals.csv', index=False)
```

## Evaluation signals

- Exactly 13,441 chemicals receive a PrHQ-based risk ranking out of the >20,000-chemical human exposome database
- PrHQ values span 14 orders of magnitude, from 3.32×10−14 to 7.61×10^0
- DHND ranks highest and N,N′-di-2-naphthyl-p-phenylenediamine (CASRN 93-46-9) ranks lowest in the prioritized list
- For each chemical, PrHQ recomputes exactly as EHE/HDMI (monotonic ratio check); the output list is sorted in descending PrHQ order
- Every ranked chemical has both a TEST-predicted LD50-derived HDMI and a SEEM EHE value (no missing ratio components)

## Limitations

- The LD50-based approach may underestimate the risk of chemicals for their chronic and subchronic toxicity
- The SEEM database was unable to cover all the compounds summarized in the study, limiting exposure (EHE) coverage
- Only chemicals with CASRN were successful in retrieving domain data from the U.S. EPA Chemistry Dashboard
- PrHQ is based on external exposure; IVIVE-based internal exposure should be considered in future work

## Evidence

- [abstract] prioritized 13,441 chemicals based on probabilistic hazard quotient and 7,770 chemicals based on risk index: "prioritized 13,441 chemicals based on probabilistic hazard quotient and 7,770 chemicals based on risk index"
- [discussion] predicted oral LD50 values for rats by TEST were used in the present study: "predicted oral LD50 values for rats by TEST were used in the present study"
- [methods] Apply the uncertainty factors defined in Equations (1)–(3) to derive the HDMI (human dose of minimal incidence) for each chemical.: [evidence span withheld — anchor status: not_found]
- [results] PrHQs spanned 3.32×10−14 to 7.61×100 (14 orders of magnitude): [evidence span withheld — anchor status: not_found]
- [discussion] may underestimate the risk of the chemicals for their chronic and subchronic toxicity: "may underestimate the risk of the chemicals for their chronic and subchronic toxicity"
