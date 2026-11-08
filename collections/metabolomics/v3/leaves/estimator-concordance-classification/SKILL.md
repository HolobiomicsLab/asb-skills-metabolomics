---
name: estimator-concordance-classification
description: Use when you need to benchmark a new or alternative parameter estimation
  tool against a gold-standard estimator (e.g. mapbayr vs NONMEM) over simulated PK
  profiles and quantify agreement with explicit thresholds.
when_to_use_negative:
- Input is a population-level (FOCE/SAiEM) estimation task rather than individual
  MAP-BE of PK parameters
- You need to assess whether the estimator recovers the true individual parameter
  value (eta_i) — MAP estimation agreement between tools does not inform this
- The goal is a runtime/performance comparison between estimators — the article explicitly
  did not perform a proper run-time comparison
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_3399
tools:
- name: mapbayr
  role: Performs MAP-Bayesian estimation of individual PK parameters in R via mapbayest();
    the tool being validated
  repo: https://github.com/FelicienLL/mapbayr
- name: NONMEM
  role: Gold-standard reference estimator used to produce control-stream-based MAP-BE
    estimates for concordance comparison
- name: mrgsolve
  role: Used by mapbayr for model implementation and ODE solving of the population
    PK models
  repo: https://github.com/metrumresearchgroup/mrgsolve
- name: R
  role: Environment in which mapbayr estimation, Delta-theta-hat computation, and
    classification/aggregation are performed
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1002/psp4.12689
    title: Easy and reliable maximum <i>a posteriori</i> Bayesian estimation of pharmacokinetic
      parameters with the open‐source R package mapbayr
  source_content_hash: sha256:af454da2f70149ba0d43da13a21c95d11312094dc67692af1de842e82a62910f
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/FelicienLL/mapbayr
  repo_url: https://github.com/FelicienLL/mapbayr
  tools:
  - NONMEM
  - R
  - mapbayr
  - mrgsolve
  edam_operation: http://edamontology.org/operation_2990
  edam_topics:
  - http://edamontology.org/topic_3399
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# estimator-concordance-classification

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Validates that an alternative MAP-Bayesian estimator (mapbayr) reproduces the reference estimator (NONMEM) by computing relative differences (Delta-theta-hat) between individual parameter estimates across many model-profile pairs and classifying each comparison as excellent, acceptable, or discordant.

## When to use

Use when you need to benchmark a new or alternative parameter estimation tool against a gold-standard estimator (e.g. mapbayr vs NONMEM) over simulated PK profiles and quantify agreement with explicit thresholds.

## When NOT to use

- Input is a population-level (FOCE/SAiEM) estimation task rather than individual MAP-BE of PK parameters
- You need to assess whether the estimator recovers the true individual parameter value (eta_i) — MAP estimation agreement between tools does not inform this
- The goal is a runtime/performance comparison between estimators — the article explicitly did not perform a proper run-time comparison

## Inputs

- 35 test population PK models coded in mrgsolve (default, bicompartmental, time-varying covariates, alternative residual error, Michaelis-Menten, lag time/DUR, large IIV)
- 4000 simulated PK profiles per test model (.csv files in the mapbayr-CPTPSP-2021 repository)
- NONMEM control streams (.mod files) for each model-profile pair
- mapbayr model code files (.cpp) and mapbayr outputs (.rds)

## Outputs

- Per-estimation Delta-theta-hat relative differences between mapbayr and NONMEM parameter estimates
- Classification of each estimation as excellent (all Delta-theta-hat < 0.001), acceptable, or discordant (any Delta-theta-hat >= 0.095)
- Aggregated summary: percentage excellent (~98%), percentage discordant (~1.5%), median Delta-theta-hat (0.0004%)

## How to apply

For each of the 35 test models and their 4000 simulated PK profiles per model, run MAP-Bayesian estimation in R with mapbayr's mapbayest() function (L-BFGS-B default optimizer, model coded in mrgsolve) and in NONMEM using the deposited control streams. For every model-profile pair, compute the relative difference Delta-theta-hat between mapbayr and NONMEM individual parameter estimates across all estimated parameters. Classify each comparison as excellent if all Delta-theta-hat < 0.001, discordant if any Delta-theta-hat >= 0.095, and acceptable otherwise. Aggregate across all models and profiles to report the percentage of excellent/discordant estimations and the median Delta-theta-hat; accept the result if it matches the reported benchmark (98% excellent, 1.5% discordant, median Delta-theta-hat 0.0004%) within reasonable optimizer-tolerance variability. Investigate discordances particularly for dose-related parameters (lag time, infusion duration) and models with large IIV variances (omega^2 > 1).

## Related tools

- **mapbayr** (Performs MAP-Bayesian estimation of individual PK parameters in R via mapbayest(); the tool being validated) — https://github.com/FelicienLL/mapbayr
- **NONMEM** (Gold-standard reference estimator used to produce control-stream-based MAP-BE estimates for concordance comparison)
- **mrgsolve** (Used by mapbayr for model implementation and ODE solving of the population PK models) — https://github.com/metrumresearchgroup/mrgsolve
- **R** (Environment in which mapbayr estimation, Delta-theta-hat computation, and classification/aggregation are performed)

## Examples

```
my_est <- my_model %>% adm_rows(time = 0, amt = 100, rate = 20) %>% obs_rows(time = 6, DV = 3.9) %>% add_covariates(BW = 90) %>% mapbayest()
```

## Evaluation signals

- Median Delta-theta-hat across all model-profile pairs approximates the reported 0.0004%
- About 98% of estimations fall in the excellent class (all Delta-theta-hat < 0.001), with ~0.5% acceptable and ~1.5% discordant (any Delta-theta-hat >= 0.095)
- For the seven real published models, PK outcome concordance approaches 100%
- Estimated parameter vectors (etas) from mapbayr match NONMEM estimates within optimizer-tolerance variability for each profile
- Discordances, if any, cluster in models with large IIV (omega^2 > 1) or with estimated lag time / infusion duration (dose-related parameters)

## Limitations

- Discordances occur when dose-related parameters (lag time, infusion duration) are estimated or when models have large IIV variances (omega^2 > 1); OFVi exploration suggested mapbayr estimates were often more likely in those cases
- MAP estimation agreement does not inform the ability to return the true individual parameter value (eta_i)
- No proper run-time comparison between mapbayr and NONMEM was performed
- The proprietary-like OFVi computation could be false when comparing estimates, and no software version numbers were stated

## Evidence

- [results] Across 140,000 test-model estimations with both NONMEM and mapbayr, the median Δθ^i was 0.0004%, with 98% of estimations classified as excellent: [evidence span withheld — anchor status: not_found]
- [abstract] For the test models, 98% of mapbayr estimations were identical to those given by NONMEM: "For the test models, 98% of mapbayr estimations were identical to those given by NONMEM"
- [discussion] for a few individuals, mapbayr is genuinely outperformed by NONMEM when MAP‐BE is performed with large IIV variances (ωk2 > 1) on several estimable parameters: "for a few individuals, mapbayr is genuinely outperformed by NONMEM when MAP‐BE is performed with large IIV variances (ωk2 > 1) on several estimable parameters."
- [discussion] Several discrepancies were also seen when lag time or infusion duration were estimated, but the exploration of OFVi suggested that estimates from mapbayr were often more likely than those of NONMEM: "Several discrepancies were also seen when lag time or infusion duration were estimated, but the exploration of OFVisuggested that estimates from mapbayr were often more likely than those of NONMEM."
- [readme] data: simulated data passed to NONMEM and mapbayr. Stored as .csv files.: [evidence span withheld — anchor status: external_doc]
