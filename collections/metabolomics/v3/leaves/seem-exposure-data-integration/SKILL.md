---
name: seem-exposure-data-integration
description: Use when a chemical inventory needs risk-based prioritization by combining
  modeled external exposure (SEEM/EHE) with LD50-based probabilistic toxicity, as
  in ranking the >20,000-chemical human exposome database.
when_to_use_negative:
- Chemicals not covered by the SEEM database — the study notes 'the SEEM database
  was unable to cover all the compounds summarized in the present study', so no EHE
  means no PrHQ
- Assessment of chronic or subchronic toxicity risk — the LD50-based approach may
  underestimate risk for chronic/subchronic endpoints
- Chemicals lacking a CASRN, since the HTS pipeline built on the U.S. EPA Chemistry
  Dashboard only retrieved data for chemicals with CASRN
tools:
- name: TEST
  role: Supplies the predicted oral LD50 values for rats that feed into the probabilistic
    reference dose (HDMI) used as the PrHQ denominator
- name: SEEM (Systematic Empirical Evaluation of Models)
  role: Provides the exposure-level estimates (EHE) used as the PrHQ numerator
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1289/ehp7722
    title: Risk-Based Chemical Ranking and Generating a Prioritized Human Exposome
      Database
  source_content_hash: sha256:6c3b471f5fc21bad3e722a0f9400ed84f72a2e7847aabcb0c86a62d2710e5585
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - SEEM (Systematic Empirical Evaluation of Models)
  - TEST
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# seem-exposure-data-integration

## Summary

Integrates SEEM (Systematic Empirical Evaluation of Models) external exposure estimates with LD50-derived probabilistic reference doses (HDMI, computed from TEST-predicted oral rat LD50 values and uncertainty factors in Equations (1)–(3)) to compute a probabilistic hazard quotient (PrHQ = EHE/HDMI) for risk-based ranking of human exposome chemicals.

## When to use

Use when a chemical inventory needs risk-based prioritization by combining modeled external exposure (SEEM/EHE) with LD50-based probabilistic toxicity, as in ranking the >20,000-chemical human exposome database.

## When NOT to use

- Chemicals not covered by the SEEM database — the study notes 'the SEEM database was unable to cover all the compounds summarized in the present study', so no EHE means no PrHQ
- Assessment of chronic or subchronic toxicity risk — the LD50-based approach may underestimate risk for chronic/subchronic endpoints
- Chemicals lacking a CASRN, since the HTS pipeline built on the U.S. EPA Chemistry Dashboard only retrieved data for chemicals with CASRN

## Inputs

- SEEM exposure estimates (EHE) per chemical from the human exposome database (HExpMetDB)
- Predicted oral LD50 values for rats (from TEST) for each chemical
- Uncertainty factors for converting LD50/BMD10 to HDMI (Equations (1)–(3))
- Chemical identifiers (CASRN) for database chemicals

## Outputs

- prioritized_chemicals.csv with chemical identifiers and PrHQ values ranked in descending order (13,441 chemicals)
- PrHQ values spanning ~3.32e-14 to 7.61e0 (14 orders of magnitude)

## How to apply

Load the SEEM exposure estimates (EHE) and LD50-derived probabilistic BMD10 values with their associated uncertainty factors for each chemical in the human exposome database. Apply the uncertainty factors defined in Equations (1)–(3) to convert each LD50 into an HDMI (human dose of minimal incidence). Compute PrHQ as the ratio EHE/HDMI per chemical using TEST-predicted oral rat LD50 values as the toxicity endpoint. Rank all chemicals in descending PrHQ and write chemical identifiers plus PrHQ values to prioritized_chemicals.csv. In the source study this yielded 13,441 ranked chemicals spanning 14 orders of magnitude, so chemical coverage (not just score computation) is the key checkpoint.

## Related tools

- **TEST** (Supplies the predicted oral LD50 values for rats that feed into the probabilistic reference dose (HDMI) used as the PrHQ denominator)
- **SEEM (Systematic Empirical Evaluation of Models)** (Provides the exposure-level estimates (EHE) used as the PrHQ numerator)

## Examples

```
df["PrHQ"] = df["EHE"] / df["HDMI"]; df.sort_values("PrHQ", ascending=False).to_csv("prioritized_chemicals.csv", index=False)
```

## Evaluation signals

- The ranked output should contain 13,441 chemicals with non-null PrHQ values
- PrHQ values should span 14 orders of magnitude, approximately 3.32e-14 to 7.61e0
- DHND should rank highest and N,N'-di-2-naphthyl-p-phenylenediamine (CASRN 93-46-9) lowest in the prioritized list
- Output schema must include chemical identifiers (CASRN) alongside PrHQ values in prioritized_chemicals.csv
- Ranking order must be strictly descending in PrHQ = EHE/HDMI; spot-check that monotonicity holds after sorting

## Limitations

- The LD50-based probabilistic reference dose may underestimate the risk of chemicals for chronic and subchronic toxicity
- The SEEM database was unable to cover all compounds in the study, limiting chemical coverage of the ranking
- The risk score is based on external exposure; the IVIVE model should be considered in future studies to convert external to internal exposure
- Toxicity due to chemical interactions (e.g., synergistic effects) in mixtures is not captured

## Evidence

- [abstract] Prioritized 13,441 chemicals based on probabilistic hazard quotient and 7,770 chemicals based on risk index: "prioritized 13,441 chemicals based on probabilistic hazard quotient and 7,770 chemicals based on risk index"
- [abstract] chemicals were prioritized by integrating exposure level obtained from the Systematic Empirical Evaluation of Models with toxicity data predicted by the Toxicity Estimation Software Tool: "prioritized by integrating exposure level obtained from the Systematic Empirical Evaluation of Models with toxicity data predicted by the Toxicity Estimation Software Tool"
- [discussion] predicted oral LD50 values for rats by TEST were used in the present study: "predicted oral LD50 values for rats by TEST were used in the present study"
- [discussion] the SEEM database was unable to cover all the compounds summarized in the present study: "the SEEM database was unable to cover all the compounds summarized in the present study"
- [discussion] the risk score was calculated using the external exposure, and the IVIVE model should be considered in future studies: "the risk score was calculated using the external exposure, and the IVIVE model should be considered in future studies"
