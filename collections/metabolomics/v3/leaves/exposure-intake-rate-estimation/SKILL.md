---
name: exposure-intake-rate-estimation
description: Use when a compiled chemical list (e.g., HExpMetDB's 20,756 human exposome
  substances) needs predicted population-level exposure/intake values (mg/kg BW per
  day) from the SEEM consensus model to support exposure- or risk-based chemical prioritization.
when_to_use_negative:
- You need internal (serum/urine) doses rather than external intake rates — the article
  notes external-to-internal exposure conversion and IVIVE modeling are needed in
  future work
- Substances lack a CASRN — the Chemistry Dashboard-based retrieval succeeds only
  for chemicals with CASRN
- You need measured exposures rather than model predictions — SEEM provides predicted,
  not biomonitoring-measured, values
edam_operation: http://edamontology.org/operation_0306
tools:
- name: U.S. EPA Chemistry Dashboard
  role: Retrieve chemical domain data by CASRN to match compiled exposome substances
    against the SEEM consensus exposure model
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1289/ehp7722
    title: Risk-Based Chemical Ranking and Generating a Prioritized Human Exposome
      Database
  source_content_hash: sha256:7f1f45500a4a2ecc2d8cd19536aaeedb4c6d880231be2b2833ef4dcc02db342f
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - U.S. EPA Chemistry Dashboard
  edam_operation: http://edamontology.org/operation_0306
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# exposure-intake-rate-estimation

## Summary

Assigns predicted median human intake rates (mg/kg BW per day, with 95% confidence intervals) to compiled human exposome chemicals by matching each substance against the SEEM consensus exposure model (Ring et al. 2019, R2 ≈ 0.8), enabling exposure-based prioritization across the >20,000-chemical HExpMetDB compilation.

## When to use

Use when a compiled chemical list (e.g., HExpMetDB's 20,756 human exposome substances) needs predicted population-level exposure/intake values (mg/kg BW per day) from the SEEM consensus model to support exposure- or risk-based chemical prioritization.

## When NOT to use

- You need internal (serum/urine) doses rather than external intake rates — the article notes external-to-internal exposure conversion and IVIVE modeling are needed in future work
- Substances lack a CASRN — the Chemistry Dashboard-based retrieval succeeds only for chemicals with CASRN
- You need measured exposures rather than model predictions — SEEM provides predicted, not biomonitoring-measured, values

## Inputs

- Compiled human exposome chemical list (20,756 substances from HExpMetDB) keyed by CASRN/structure
- SEEM consensus exposure model (Ring et al. 2019, R2 ≈ 0.8)
- U.S. EPA Chemistry Dashboard records retrieved by CASRN

## Outputs

- exposure_predictions.csv: per-substance predicted median intake rate (mg/kg BW per day) and 95% CI bounds
- Coverage summary (matched vs. missing substances, e.g., 15,408/20,756)
- Spot-check results for reference chemicals (BPA, DEHP, PFOA, dihexyl nonanedioate)

## How to apply

Load the compiled human exposome chemical list (20,756 substances from HExpMetDB) and match each chemical by CASRN or structure against the SEEM consensus model, using the U.S. EPA Chemistry Dashboard to retrieve records (only chemicals with CASRN can be retrieved). Apply the SEEM consensus model (Ring et al. 2019, R2 ≈ 0.8) to each matched substance to obtain predicted median intake rates in mg/kg BW per day together with 95% confidence interval bounds. Record coverage statistics — in the source study, 15,408 of 20,756 substances received predicted values — and flag unmatched substances as missing, since the SEEM database was unable to cover all compounds. Spot-check anchor chemicals against reported values (BPA = 5.50×10−5, DEHP = 2.72×10−3, PFOA = 5.47×10−8 mg/kg BW per day) and confirm the maximum-exposure substance (dihexyl nonanedioate). Save the full per-substance table of median and CI-bound predictions plus the coverage summary as exposure_predictions.csv.

## Related tools

- **U.S. EPA Chemistry Dashboard** (Retrieve chemical domain data by CASRN to match compiled exposome substances against the SEEM consensus exposure model)

## Examples

```
python -c "import pandas as pd; df = pd.read_csv('exposure_predictions.csv'); print(len(df), df['seem_intake_median_mg_kg_bw_day'].min(), df['seem_intake_median_mg_kg_bw_day'].max())"
```

## Evaluation signals

- Coverage count matches expectation: 15,408 of 20,756 substances receive predicted exposure values; the remainder flagged as missing
- Anchor spot-checks reproduce reported values: BPA = 5.50×10−5, DEHP = 2.72×10−3, PFOA = 5.47×10−8 mg/kg BW per day
- Predicted exposures span approximately 15 orders of magnitude, with dihexyl nonanedioate as the highest-exposure substance
- Every matched row in exposure_predictions.csv carries both a median intake rate and 95% CI bounds; no matched substance has a bare median without CI
- All retrieved substances have a CASRN, consistent with Chemistry Dashboard retrieval requirements

## Limitations

- The SEEM database was unable to cover all compounds in the compiled exposome list, so predictions are partial (15,408/20,756)
- Only chemicals with CASRN could retrieve domain data from the U.S. EPA Chemistry Dashboard
- Predicted exposures are external exposure estimates; chronic/subchronic risk interpretation may be incomplete without IVIVE-based internal exposure conversion

## Evidence

- [abstract] chemicals were prioritized by integrating exposure level obtained from the Systematic Empirical Evaluation of Models: "chemicals were prioritized by integrating exposure level obtained from the Systematic Empirical Evaluation of Models"
- [abstract] We compiled a human exposome database of >20,000 chemicals: "We compiled a human exposome database of >20,000 chemicals"
- [intro] the U.S. EPA initiated the Exposure Forecasting (ExpoCast) program, which contained information on exposure: "the U.S. EPA initiated the Exposure Forecasting (ExpoCast) program, which contained information on exposure"
- [discussion] the SEEM database was unable to cover all the compounds summarized in the present study: "the SEEM database was unable to cover all the compounds summarized in the present study"
- [discussion] only chemicals with CASRN were successful in retrieving diverse types of relevant domain data from the U.S. EPA Chemistry Dashboard: "only chemicals with CASRN were successful in retrieving diverse types of relevant domain data from the U.S. EPA Chemistry Dashboard"
