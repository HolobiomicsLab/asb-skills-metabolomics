---
name: dose-response-threshold-estimation
description: Use when you need to prioritize chemicals by risk from exposure estimates
  plus predicted acute toxicity (LD50) data, and no measured chronic reference dose
  or BMD is available for most compounds.
when_to_use_negative:
- The goal is chronic or subchronic risk assessment — the LD50-based approach may
  underestimate chronic/subchronic toxicity risk.
- Chemicals lack SEEM exposure estimates (the SEEM database was unable to cover all
  compounds in the study).
- Ameasured, chemical-specific reference dose or in vivo toxicity data are available;
  a measured BMD is preferable to an LD50-derived probabilistic surrogate.
tools:
- name: TEST
  role: Provides the predicted oral LD50 values for rats that serve as the toxicity
    endpoint converted into the probabilistic reference dose (HDMI)
- name: U.S. EPA Chemistry Dashboard
  role: Source of chemicals' domain data (CASRN-indexed) underlying the high-throughput
    toxicity estimates
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1289/ehp7722
    title: Risk-Based Chemical Ranking and Generating a Prioritized Human Exposome
      Database
  source_content_hash: sha256:4e4cbf22d1e0e06d6017ecb97649531d4a3d67b4ca9dbcc65e91a4fb5501799a
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - TEST
  - Toxicity Estimation Software Tool
  - U.S. EPA Chemistry Dashboard
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 14
---

# dose-response-threshold-estimation

## Summary

Derives probabilistic reference dose thresholds (HDMI) from predicted oral LD50 values with applied uncertainty factors, then computes a probabilistic hazard quotient (PrHQ = EHE/HDMI) to risk-rank chemicals in the human exposome database.

## When to use

Use when you need to prioritize chemicals by risk from exposure estimates plus predicted acute toxicity (LD50) data, and no measured chronic reference dose or BMD is available for most compounds.

## When NOT to use

- The goal is chronic or subchronic risk assessment — the LD50-based approach may underestimate chronic/subchronic toxicity risk.
- Chemicals lack SEEM exposure estimates (the SEEM database was unable to cover all compounds in the study).
- Ameasured, chemical-specific reference dose or in vivo toxicity data are available; a measured BMD is preferable to an LD50-derived probabilistic surrogate.

## Inputs

- SEEM exposure estimates (EHE) per chemical
- Predicted oral LD50 values for rats from TEST
- Uncertainty factors for deriving probabilistic BMD10 values (Equations 1–3)

## Outputs

- HDMI (human dose of minimal incidence) per chemical
- PrHQ value per chemical (EHE/HDMI)
- Risk-ranked chemical list (prioritized_chemicals.csv) with chemical identifiers and PrHQ values

## How to apply

Load the SEEM exposure estimates (EHE) and TEST-predicted oral LD50 values for rats for each chemical in the human exposome database. Apply the uncertainty factors defined in Equations (1)–(3) to convert the LD50-derived probabilistic BMD10 values into the human dose of minimal incidence (HDMI). Calculate PrHQ as the ratio PrHQ = EHE/HDMI for each chemical. Rank all chemicals in descending PrHQ order to generate the prioritized list (13,441 chemicals spanning 3.32×10−14 to 7.61×100, 14 orders of magnitude, in the source study). Write the ranked identifiers and PrHQ values to prioritized_chemicals.csv.

## Related tools

- **TEST** (Provides the predicted oral LD50 values for rats that serve as the toxicity endpoint converted into the probabilistic reference dose (HDMI))
- **U.S. EPA Chemistry Dashboard** (Source of chemicals' domain data (CASRN-indexed) underlying the high-throughput toxicity estimates)

## Examples

```
df["HDMI"] = df["LD50_pred"] * df["uncertainty_factor"]; df["PrHQ"] = df["EHE"]/df["HDMI"]; df.sort_values("PrHQ", ascending=False).to_csv("prioritized_chemicals.csv", index=False)
```

## Evaluation signals

- Every chemical with both EHE and HDMI yields a finite PrHQ; counts should match the number of chemicals with both inputs (13,441 in the source study).
- PrHQ values span ~14 orders of magnitude (3.32×10−14 to 7.61×100); a much narrower span suggests misapplied uncertainty factors.
- Sanity-check extreme ranks: DHND should top the ranking and N,N′-di-2-naphthyl-p-phenylenediamine (CASRN 93-46-9) should sit at the bottom.
- Output CSV schema includes chemical identifiers (e.g., CASRN) and PrHQ columns, sorted in descending PrHQ order.
- Cross-check a ToxPi-based toxicity ranking on a subset for directional consistency with the PrHQ ranking.

## Limitations

- The LD50-based approach may underestimate the risk of chemicals for chronic and subchronic toxicity.
- The risk score uses external exposure; IVIVE-based internal exposure conversion should be considered in future work.
- SEEM database was unable to cover all compounds, limiting the number of rankable chemicals.
- Chemical interactions (e.g., synergistic effects) in mixture effects cannot be excluded from the ranking.

## Evidence

- [abstract] Prioritized 13,441 chemicals based on probabilistic hazard quotient and 7,770 chemicals based on risk index: "prioritized 13,441 chemicals based on probabilistic hazard quotient and 7,770 chemicals based on risk index"
- [abstract] SEEM+TEST integration for prioritization: "chemicals were prioritized by integrating exposure level obtained from the Systematic Empirical Evaluation of Models with toxicity data predicted by the Toxicity Estimation Software Tool"
- [discussion] TEST LD50 values as toxicity input: "predicted oral LD50 values for rats by TEST were used in the present study"
- [discussion] Chronic risk underestimation caveat: "may underestimate the risk of the chemicals for their chronic and subchronic toxicity"
- [discussion] External-exposure caveat: "the risk score was calculated using the external exposure, and the IVIVE model should be considered in future studies"
