---
name: chemical-priority-ranking
description: Use when you have human exposure estimates and predicted toxicity (e.g.,
  oral LD50) for a chemical set and need a ranked risk prioritization spanning many
  orders of magnitude.
when_to_use_negative:
- The goal is a multi-domain toxicity ranking (e.g., ToxPi scoring from ToxCast HTS
  assays) rather than an exposure/hazard quotient — the paper computed ToxPi scores
  separately for comprehensive toxicity ranking
- Chronic or subchronic toxicity risk assessment is required — the LD50-based PrHQ
  approach may underestimate such risks
- A chemical lacks SEEM exposure coverage — the SEEM database was unable to cover
  all compounds in the study
tools:
- name: TEST
  role: Predicts oral LD50 values for rats used to derive probabilistic BMD10 values
    in the PrHQ calculation
- name: U.S. EPA Chemistry Dashboard
  role: Source for retrieving domain data for chemicals with CASRN in the HTS risk-based
    workflow
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1289/ehp7722
    title: Risk-Based Chemical Ranking and Generating a Prioritized Human Exposome
      Database
  source_content_hash: sha256:b0dc883a35d59eb5ca43a04b67382dc8509cfd15341b9f762b59e263fb2ba1f8
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - TEST
  - U.S. EPA Chemistry Dashboard
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# chemical-priority-ranking

## Summary

Prioritizes chemicals by probabilistic hazard quotient (PrHQ), computed as the ratio of SEEM-based exposure estimates (EHE) to LD50-derived probabilistic reference doses (HDMI), producing a risk-ranked list of exposome chemicals.

## When to use

Use when you have human exposure estimates and predicted toxicity (e.g., oral LD50) for a chemical set and need a ranked risk prioritization spanning many orders of magnitude.

## When NOT to use

- The goal is a multi-domain toxicity ranking (e.g., ToxPi scoring from ToxCast HTS assays) rather than an exposure/hazard quotient — the paper computed ToxPi scores separately for comprehensive toxicity ranking
- Chronic or subchronic toxicity risk assessment is required — the LD50-based PrHQ approach may underestimate such risks
- A chemical lacks SEEM exposure coverage — the SEEM database was unable to cover all compounds in the study

## Inputs

- SEEM exposure estimates (EHE) per chemical
- Predicted oral LD50 values for rats (e.g., from TEST) with LD50-derived probabilistic BMD10 values
- Uncertainty factors for deriving HDMI (Equations 1–3)
- Human exposome database chemical identifiers (e.g., CASRN, names)

## Outputs

- Risk-ranked chemical list (prioritized_chemicals.csv) with chemical identifiers and PrHQ values in descending order
- PrHQ range summary (e.g., 3.32×10−14 to 7.61×10^0, 14 orders of magnitude)

## How to apply

Load SEEM exposure estimates (EHE) and LD50-derived probabilistic BMD10 values with their associated uncertainty factors for each chemical. Apply the uncertainty factors from Equations (1)–(3) to derive the HDMI (human dose of minimal incidence) for each chemical. Calculate PrHQ = EHE/HDMI per chemical, then sort all chemicals in descending order of PrHQ. In the source study this ranked 13,441 chemicals whose PrHQs spanned 3.32×10−14 to 7.61×10^0 (14 orders of magnitude), with DHND ranked highest and N,N′-di-2-naphthyl-p-phenylenediamine (CASRN 93-46-9) lowest. Write chemical identifiers and PrHQ values to a ranked CSV (prioritized_chemicals.csv).

## Related tools

- **TEST** (Predicts oral LD50 values for rats used to derive probabilistic BMD10 values in the PrHQ calculation)
- **U.S. EPA Chemistry Dashboard** (Source for retrieving domain data for chemicals with CASRN in the HTS risk-based workflow)

## Examples

```
df['PrHQ'] = df['EHE'] / df['HDMI']; df.sort_values('PrHQ', ascending=False).to_csv('prioritized_chemicals.csv', index=False)
```

## Evaluation signals

- All ranked chemicals have a computable PrHQ = EHE/HDMI (positive HDMI denominator); count of ranked chemicals matches the number of chemicals with both EHE and HDMI
- Output CSV contains chemical identifiers plus PrHQ values sorted in strictly descending order
- PrHQ spread should span ~14 orders of magnitude for a full exposome-scale set (study range: 3.32×10−14 to 7.61×10^0)
- Top and bottom entries in the reference reproduction are DHND (highest) and N,N′-di-2-naphthyl-p-phenylenediamine, CASRN 93-46-9 (lowest)
- Total ranked count matches the expected 13,441 chemicals when run on the HExpMetDB exposome set

## Limitations

- LD50-based reference doses may underestimate the risk of chemicals for chronic and subchronic toxicity
- Risk score is based on external exposure; the IVIVE model should be considered in future studies
- The SEEM database was unable to cover all compounds in the exposome database
- Toxicity from chemical interaction (e.g., synergistic effects) in mixtures cannot be excluded

## Evidence

- [methods] PrHQ = EHE/HDMI: [evidence span withheld — anchor status: not_found]
- [abstract] prioritized 13,441 chemicals by PrHQ: "prioritized 13,441 chemicals based on probabilistic hazard quotient and 7,770 chemicals based on risk index"
- [discussion] TEST LD50 predictions feed PrHQ: "predicted oral LD50 values for rats by TEST were used in the present study"
- [abstract] exposure integrated with toxicity: "chemicals were prioritized by integrating exposure level obtained from the Systematic Empirical Evaluation of Models with toxicity data predicted by the Toxicity Estimation Software Tool"
