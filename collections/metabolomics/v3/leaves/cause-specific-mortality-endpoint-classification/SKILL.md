---
name: cause-specific-mortality-endpoint-classification
description: Use when a cohort has cause-of-death coded vital status (e.g., cardiovascular,
  cancer, infection, other) and you must test whether metabolic biomarkers associated
  with all-cause mortality also associate with specific causes of death.
when_to_use_negative:
- The cohort only has all-cause vital status with no cause-of-death coding — there
  are no cause-specific endpoints to classify against.
- The goal is de novo biomarker selection/discovery (stepwise forward-backward selection
  over 226 biomarkers with Bonferroni threshold 2.21 x 10^-4), not testing already-identified
  biomarkers across endpoints.
- Cause-specific event counts are too small to support stable Cox estimates for the
  rarer endpoints (e.g., nonlocalized infections).
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: R
  role: Fits Cox proportional hazards models with age at blood sampling as the time
    scale for each biomarker-outcome pair and assembles the Table 3 results
- name: R-package meta
  role: Fixed-effect meta-analysis across cohorts whose results underpin the biomarker
    set being tested against cause-specific endpoints
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-019-11311-9
    title: A metabolic profile of all-cause mortality risk identified in an observational
      study of 44,168 individuals
  source_content_hash: sha256:987701ea834bf05aa1e7ae51b31e663654e8df8906335de848b13ca34939f1cf
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - R-package meta
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# cause-specific-mortality-endpoint-classification

## Summary

Fits biomarker-specific Cox proportional hazards models against distinct cause-of-death endpoints (cardiovascular, cancer, nonlocalized infections, other) in addition to all-cause mortality, to test whether mortality-associated metabolic biomarkers reflect general health status rather than a single disease pathway.

## When to use

Use when a cohort has cause-of-death coded vital status (e.g., cardiovascular, cancer, infection, other) and you must test whether metabolic biomarkers associated with all-cause mortality also associate with specific causes of death.

## When NOT to use

- The cohort only has all-cause vital status with no cause-of-death coding — there are no cause-specific endpoints to classify against.
- The goal is de novo biomarker selection/discovery (stepwise forward-backward selection over 226 biomarkers with Bonferroni threshold 2.21 x 10^-4), not testing already-identified biomarkers across endpoints.
- Cause-specific event counts are too small to support stable Cox estimates for the rarer endpoints (e.g., nonlocalized infections).

## Inputs

- Cohort survival data with cause-of-death endpoints (e.g., FINRISK 1997: 7603 individuals, 1213 deaths; all-cause, cancer, cardiovascular, nonlocalized infections, other)
- Plasma/serum measurements of the 14 mortality-associated biomarkers (XXL-VLDL-L, S-HDL-L, VLDL-D, PUFA/FA, Glc, Lac, His, Ile, Leu, Val, Phe, AcAce, Alb, GlycA) from the Nightingale Health NMR platform
- Age at blood sampling and follow-up/vital-status variables for each individual

## Outputs

- Table 3-style matrix of hazard ratios, 95% confidence intervals, and P values for 14 biomarker rows x 5 cause-of-death columns
- Per-biomarker direction-of-effect comparison across cause-specific outcomes versus all-cause mortality

## How to apply

Preprocess each biomarker exactly as for the all-cause analysis: add 1 to values below the detection limit (x+1), log-transform, and scale to standard deviation units separately per study. For each of the 14 biomarkers (XXL-VLDL-L, S-HDL-L, VLDL-D, PUFA/FA, Glc, Lac, His, Ile, Leu, Val, Phe, AcAce, Alb, GlycA) and each of the 5 mortality outcomes (all-cause, cancer, cardiovascular, nonlocalized infections, other), fit a Cox proportional hazards model in R with age at blood sampling as the time scale. Collect hazard ratios, 95% CIs, and P values per biomarker-outcome pair, then assemble a table matching the layout of Table 3 (biomarker rows x cause-of-death columns). The decision point is interpretive: associations pointing in the same direction as all-cause mortality across multiple causes — including nonlocalized infections — indicate the biomarkers are general markers of immune-metabolic health adversity rather than indicators of one specific pathology.

## Related tools

- **R** (Fits Cox proportional hazards models with age at blood sampling as the time scale for each biomarker-outcome pair and assembles the Table 3 results)
- **R-package meta** (Fixed-effect meta-analysis across cohorts whose results underpin the biomarker set being tested against cause-specific endpoints)

## Examples

```
library(survival); coxph(Surv(age_blood, age_blood + fu_years, death_cvd) ~ glycA, data = finrisk1997)
```

## Evaluation signals

- All-cause HRs reproduce Table 3 anchors: GlycA HR 1.41 (95% CI 1.26-1.57, P 5.75e-10) and PUFA/FA HR 0.73 (95% CI 0.69-0.78, P < 2.22e-16).
- The output table has exactly 14 biomarker rows and 5 cause-of-death columns (all-cause, cancer, cardiovascular, nonlocalized infections, other), each cell with HR, 95% CI, and P value.
- Preprocessing invariants hold: biomarkers containing zeroes were incremented (x+1), all biomarkers log-transformed and scaled to standard deviation units separately per study.
- The majority of the 14 biomarkers show associations with multiple cause-specific outcomes in the same direction as all-cause mortality, including nonlocalized infections.
- Each Cox model uses age at blood sampling as the time scale, not calendar time or follow-up time.

## Limitations

- Same-direction associations across causes support a general immune-metabolic health adversity interpretation but do not identify the mechanisms or specific health conditions the biomarkers reflect.
- The targeted NMR platform captures only a fraction of the metabolites in human serum, limiting which biomarkers can be tested.
- Per-cohort scaled biomarker values mean the associated score is not yet suitable for clinical patient classification; a concentration-unit score from individual-level data is still needed.

## Evidence

- [discussion] Cox model with age as time scale: "a Cox proportional hazards model with age at blood sampling as the time scale, was used to determine the associations of the biomarkers with all-cause mortality"
- [discussion] Preprocessing of biomarker values: [evidence span withheld — anchor status: not_found]
- [abstract] 14 biomarkers identified: "We apply a stepwise (forward-backward) procedure based on meta-analysis results and identify 14 circulating biomarkers independently associating with all-cause mortality."
- [discussion] Biomarker set context: "By performing high-throughput metabolic biomarker profiling in 44,168 individuals from 12 cohorts, we identified a set of 14 biomarkers independently associating with all-cause mortality"
