---
name: confidence-interval-reporting
description: Use when exposure predictions from a statistical model (e.g., SEEM consensus
  exposure estimates over the HExpMetDB chemical list) must be reported per substance
  with median values, 95% CI bounds, and coverage fractions rather than point estimates
  alone.
when_to_use_negative:
- The chemical has no SEEM model match (no CASRN or model coverage) — flag as missing
  instead of reporting an imputed CI
- The analysis needs measured biomonitoring concentrations rather than model-predicted
  exposure — SEEM outputs are predictions, not measurements
- Reporting toxicity data (TEST-predicted LD50, ToxPi scores) where CI semantics differ
  from SEEM intake-rate uncertainty
edam_operation: http://edamontology.org/operation_0336
tools:
- name: U.S. EPA Chemistry Dashboard
  role: Source for chemical lookup (CASRN-based) used to match substances to the SEEM
    consensus exposure model before reporting CI-bounded predictions
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1289/ehp7722
    title: Risk-Based Chemical Ranking and Generating a Prioritized Human Exposome
      Database
  source_content_hash: sha256:07452e3a6ef2e1a4600e6333e3e4feb6e50fe2e123ae2d42181eaf2fb83ced21
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - U.S. EPA Chemistry Dashboard
  edam_operation: http://edamontology.org/operation_0336
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 4
---

# confidence-interval-reporting

## Summary

Reports predicted human-exposure estimates (SEEM consensus model, Ring et al. 2019, R2 ≈ 0.8) as median intake rates (mg/kg BW per day) with 95% confidence-interval bounds per chemical, alongside explicit coverage statistics so uncertainty and missingness are visible to downstream risk prioritization.

## When to use

Use when exposure predictions from a statistical model (e.g., SEEM consensus exposure estimates over the HExpMetDB chemical list) must be reported per substance with median values, 95% CI bounds, and coverage fractions rather than point estimates alone.

## When NOT to use

- The chemical has no SEEM model match (no CASRN or model coverage) — flag as missing instead of reporting an imputed CI
- The analysis needs measured biomonitoring concentrations rather than model-predicted exposure — SEEM outputs are predictions, not measurements
- Reporting toxicity data (TEST-predicted LD50, ToxPi scores) where CI semantics differ from SEEM intake-rate uncertainty

## Inputs

- Compiled human exposome chemical list (20,756 substances with CASRN/structures from HExpMetDB)
- SEEM consensus model predicted median intake rates (mg/kg BW per day) with 95% confidence-interval bounds per matched chemical
- Match/unmatch status per chemical (CASRN-to-model lookup results)

## Outputs

- exposure_predictions.csv: table of predicted exposure values (median and 95% CI lower/upper bounds per substance)
- Coverage summary (15,408 of 20,756 substances covered; unmatched substances flagged as missing)
- Spot-check validation record for BPA, DEHP, PFOA, and highest-exposure chemical (dihexyl nonanedioate)

## How to apply

Match each compiled chemical (20,756 substances from HExpMetDB) to the SEEM consensus model by CASRN/structure and record the predicted median intake rate (mg/kg BW per day) together with its 95% confidence-interval lower and upper bounds. Compute and report coverage explicitly (15,408 of 20,756 substances covered; the SEEM database was unable to cover all compounds) and flag unmatched chemicals as missing rather than imputing. Report the span of predicted values (15 orders of magnitude) and identify extremes (dihexyl nonanedioate highest) plus representative spot-check values (BPA = 5.50×10−5, DEHP = 2.72×10−3, PFOA = 5.47×10−8 mg/kg BW per day). Save the median and CI bounds per substance as exposure_predictions.csv with a coverage summary, so downstream probabilistic hazard-quotient and risk-index calculations carry uncertainty through prioritization.

## Related tools

- **U.S. EPA Chemistry Dashboard** (Source for chemical lookup (CASRN-based) used to match substances to the SEEM consensus exposure model before reporting CI-bounded predictions)

## Examples

```
df = pd.read_csv('exposure_predictions.csv'); assert (df.ci_low <= df.median).all() and (df.median <= df.ci_high).all(); print(len(df), 'covered of 20756')
```

## Evaluation signals

- Coverage count matches 15,408 of 20,756 substances, with the remainder explicitly flagged missing (not imputed)
- Spot-check values reproduce: BPA = 5.50×10−5, DEHP = 2.72×10−3, PFOA = 5.47×10−8 mg/kg BW per day
- Reported span of predicted exposures spans 15 orders of magnitude
- Highest predicted exposure is dihexyl nonanedioate
- Every reported substance row contains median, CI-lower, and CI-upper values; CI-lower ≤ median ≤ CI-upper for all rows

## Limitations

- The SEEM database was unable to cover all the compounds summarized in the present study — coverage is partial by design and must be reported
- CI bounds reflect model uncertainty of the consensus exposure model (Ring et al. 2019, R2 ≈ 0.8), not measurement variability
- Only chemicals with CASRN were successful in retrieving domain data from the U.S. EPA Chemistry Dashboard, biasing coverage toward CASRN-annotated substances

## Evidence

- [abstract] chemicals were prioritized by integrating exposure level obtained from the Systematic Empirical Evaluation of Models: "chemicals were prioritized by integrating exposure level obtained from the Systematic Empirical Evaluation of Models with toxicity data predicted by the Toxicity Estimation Software Tool"
- [discussion] the SEEM database was unable to cover all the compounds summarized in the present study: "the SEEM database was unable to cover all the compounds summarized in the present study"
- [other] SEEM-derived predicted exposure values were available for 15,408 of 20,756 substances: [evidence span withheld — anchor status: not_found]
- [other] Spot-check reported values: BPA = 5.50×10−5: [evidence span withheld — anchor status: not_found]
