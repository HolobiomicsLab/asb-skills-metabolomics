---
name: exposure-hazard-quotient-calculation
description: Use when a chemical inventory lacks measured toxicity values and you
  need a probabilistic exposure-vs-hazard risk score (PrHQ) to rank chemicals for
  prioritization.
when_to_use_negative:
- The goal is chronic or subchronic toxicity risk assessment — the LD50-based HDMI
  may underestimate the risk of chemicals for their chronic and subchronic toxicity
- Internal dose is required — the PrHQ was calculated using external exposure, and
  the article states the IVIVE model should be considered in future studies
- Chemical lacks a CASRN — the high-throughput screening method based on the U.S.
  EPA Chemistry Dashboard only retrieves domain data for chemicals with CASRN, and
  SEEM was unable to cover all compounds
edam_operation: http://edamontology.org/operation_3501
edam_topics:
- http://edamontology.org/topic_3407
- http://edamontology.org/topic_2815
tools:
- name: TEST
  role: Predicts the oral LD50 values for rats that are converted into the probabilistic
    BMD10/HDMI hazard term of the PrHQ
- name: ToxPI
  role: Provides a complementary ToxPi score computed from the ToxCast database to
    more comprehensively estimate the toxicity ranking of chemicals
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1289/ehp7722
    title: Risk-Based Chemical Ranking and Generating a Prioritized Human Exposome
      Database
  source_content_hash: sha256:8d516dab0262b871958941359fd6e65344f79f1259d93a9fdf8fa924c86aa2ce
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - TEST
  - ToxPI
  edam_operation: http://edamontology.org/operation_3501
  edam_topics:
  - http://edamontology.org/topic_3407
  - http://edamontology.org/topic_2815
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# exposure-hazard-quotient-calculation

## Summary

Calculates a probabilistic hazard quotient (PrHQ) for each chemical as the ratio of SEEM-based exposure estimates (EHE) to an LD50-derived probabilistic human dose of minimal incidence (HDMI), enabling risk-based ranking of thousands of exposome chemicals.

## When to use

Use when a chemical inventory lacks measured toxicity values and you need a probabilistic exposure-vs-hazard risk score (PrHQ) to rank chemicals for prioritization.

## When NOT to use

- The goal is chronic or subchronic toxicity risk assessment — the LD50-based HDMI may underestimate the risk of chemicals for their chronic and subchronic toxicity
- Internal dose is required — the PrHQ was calculated using external exposure, and the article states the IVIVE model should be considered in future studies
- Chemical lacks a CASRN — the high-throughput screening method based on the U.S. EPA Chemistry Dashboard only retrieves domain data for chemicals with CASRN, and SEEM was unable to cover all compounds

## Inputs

- SEEM (Systematic Empirical Evaluation of Models) exposure estimates (EHE) per chemical
- Predicted oral rat LD50 values from the Toxicity Estimation Software Tool (TEST)
- LD50-derived probabilistic BMD10 values with associated uncertainty factors
- Chemical identifiers (CASRN) for each database entry

## Outputs

- PrHQ value per chemical
- Risk-ranked chemical list (prioritized_chemicals.csv) with chemical identifiers and PrHQ values, ranked 13,441 chemicals in descending PrHQ

## How to apply

Load the SEEM exposure estimates (EHE) and the predicted oral LD50 values for rats from TEST, converting LD50 to a probabilistic BMD10 with associated uncertainty factors for each chemical. Apply the uncertainty factors defined in Equations (1)–(3) to derive the HDMI (human dose of minimal incidence). Compute PrHQ = EHE/HDMI per chemical, then sort in descending order to produce the risk ranking. In the source study this ranked 13,441 chemicals spanning 3.32×10−14 to 7.61 (14 orders of magnitude), with DHND highest and N,N′-di-2-naphthyl-p-phenylenediamine (CASRN 93-46-9) lowest. For a more comprehensive toxicity estimate, the article suggests complementing the ranking with a ToxPi score computed from ToxCast assay data.

## Related tools

- **TEST** (Predicts the oral LD50 values for rats that are converted into the probabilistic BMD10/HDMI hazard term of the PrHQ)
- **ToxPI** (Provides a complementary ToxPi score computed from the ToxCast database to more comprehensively estimate the toxicity ranking of chemicals)

## Examples

```
python prioritize_prhq.py --exposures seem_ehe.csv --ld50 test_ld50.csv --output prioritized_chemicals.csv
```

## Evaluation signals

- The ranked output contains 13,441 chemicals (matching the article's PrHQ prioritization count)
- PrHQ values span approximately 3.32×10−14 to 7.61, i.e. 14 orders of magnitude
- DHND appears at the top of the descending PrHQ ranking and N,N′-di-2-naphthyl-p-phenylenediamine (CASRN 93-46-9) at the bottom
- Every chemical with a computed PrHQ has both a non-null EHE and an LD50-derived HDMI
- Ranked results written to prioritized_chemicals.csv include chemical identifiers and PrHQ values

## Limitations

- LD50-based hazard may underestimate the risk of the chemicals for their chronic and subchronic toxicity
- Risk score is based on external exposure; IVIVE should be considered in future studies
- The SEEM database was unable to cover all compounds, and only chemicals with CASRN retrieve domain data from the U.S. EPA Chemistry Dashboard
- Cannot exclude toxicity due to chemical interaction (e.g. synergistic effect) in the mixture effects

## Evidence

- [abstract] Prioritized 13,441 chemicals based on probabilistic hazard quotient and 7,770 chemicals based on risk index: "prioritized 13,441 chemicals based on probabilistic hazard quotient and 7,770 chemicals based on risk index"
- [results] PrHQ as ratio of EHE to HDMI: [evidence span withheld — anchor status: not_found]
- [results] Apply uncertainty factors from Equations (1)-(3) to derive HDMI: [evidence span withheld — anchor status: not_found]
- [discussion] TEST-predicted LD50 used as toxicity input: "predicted oral LD50 values for rats by TEST were used in the present study"
- [discussion] LD50 approach may underestimate chronic/subchronic risk: "may underestimate the risk of the chemicals for their chronic and subchronic toxicity"
- [discussion] Risk score based on external exposure; IVIVE needed: "the risk score was calculated using the external exposure, and the IVIVE model should be considered in future studies"
