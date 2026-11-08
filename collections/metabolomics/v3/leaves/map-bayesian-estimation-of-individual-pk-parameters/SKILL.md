---
name: map-bayesian-estimation-of-individual-pk-parameters
description: Use when individual PK parameters (CL, V, ETA) must be estimated from
  sparse TDM concentration data given a published population PK model, e.g., for model-informed
  precision dosing (MIPD) or therapeutic drug monitoring in R.
when_to_use_negative:
- Population PK/pharmacodynamics model building or parameter estimation from population
  data — use nlmixr or saemix, which are dedicated to population modeling
- Situations requiring estimation-uncertainty quantification (Fisher information matrix
  or posterior sampling), which the current mapbayr version does not implement
- When a validated MIPD app is needed for direct clinical dose individualization —
  mapbayr-shiny apps are illustrative only and distributed without any warranty
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_3399
- http://edamontology.org/topic_2258
tools:
- name: mapbayr
  role: Core R package performing MAP-BE via mapbayest() from any mrgsolve-coded population
    PK model
  repo: https://github.com/FelicienLL/mapbayr
- name: mrgsolve
  role: Model implementation and ODE solving (LSODA) underlying mapbayr; used to simulate
    profiles and predict individual concentrations
  repo: https://github.com/metrumresearchgroup/mrgsolve
- name: NONMEM
  role: Gold-standard reference used to benchmark mapbayr MAP-BE estimates on identical
    models and data
- name: R
  role: Runtime environment; mapbayr installed from CRAN or GitHub via devtools::install_github
- name: Shiny
  role: Wrapping mapbayr into standalone MIPD/TDM web applications
  repo: https://github.com/FelicienLL/mapbayr-shiny
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1002/psp4.12689
    title: Easy and reliable maximum <i>a posteriori</i> Bayesian estimation of pharmacokinetic
      parameters with the open‐source R package mapbayr
  source_content_hash: sha256:9182d4824e0c08fbe2a20bdcf4d17b816dc9ebf5078af2fae2faa1a65f826e49
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
  - Shiny
  - dplyr
  - ggplot2
  - mapbayr
  - mrgsolve
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_3399
  - http://edamontology.org/topic_2258
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8666666666666667
n_claims: 15
---

# map-bayesian-estimation-of-individual-pk-parameters

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Perform maximum a posteriori Bayesian estimation (MAP-BE) of individual pharmacokinetic parameters in R using the mapbayr package with mrgsolve-coded population PK models, avoiding external nonlinear mixed-effect software such as NONMEM. The skill is validated against NONMEM, showing 98% identical estimations on test models and near-100% concordance on PK outcomes for seven real published models.

## When to use

Use when individual PK parameters (CL, V, ETA) must be estimated from sparse TDM concentration data given a published population PK model, e.g., for model-informed precision dosing (MIPD) or therapeutic drug monitoring in R.

## When NOT to use

- Population PK/pharmacodynamics model building or parameter estimation from population data — use nlmixr or saemix, which are dedicated to population modeling
- Situations requiring estimation-uncertainty quantification (Fisher information matrix or posterior sampling), which the current mapbayr version does not implement
- When a validated MIPD app is needed for direct clinical dose individualization — mapbayr-shiny apps are illustrative only and distributed without any warranty

## Inputs

- Population PK model coded in mrgsolve ($PARAM, $OMEGA, $SIGMA, $CMT, $ODE), e.g., .cpp model files
- Concentration dataset in NM-TRAN format (ID, time, evid, cmt, amt, rate, DV, mdv, covariates) as .csv
- Published TDM design specifications (rich/sparse sampling, single/multiple dosing)
- Covariate values (e.g., BW) per individual

## Outputs

- mapbayests S3 object containing ETAs, IPRED/PRED tables and internal algorithm arguments
- Posterior ETA values via get_eta() and individual parameters (e.g., CL) via get_param()
- Updated mrgsolve model object via use_estimates() for simulating outcomes (AUCtau,SS, troughs)
- Plot/hist visualizations of predictions and parameter distributions; .rds output files

## How to apply

Code the population PK model in mrgsolve ($PARAM, $OMEGA, $SIGMA, $CMT, $ODE), including covariate effects and the residual error model, and verify it with check_mapbayr_model(); for parent-metabolite drugs like sunitinib, code simultaneous parent and metabolite compartments. Build the dataset in NM-TRAN format or pipe-friendly via adm_rows(), obs_rows() and add_covariates(), then run mapbayest() with the default L-BFGS-B optimization. Extract posterior ETAs with get_eta() and derived parameters with get_param(), and update the model object with use_estimates() to simulate MIPD-relevant outcomes such as AUCtau,SS (from oral CL and dose), trough concentrations, or time below a toxicity threshold (e.g., 0.2 uM for methotrexate). Optionally benchmark against NONMEM using the same models, data, and LSODA solver, classifying each estimation as concordant or discordant; expect discordances to concentrate on dose-related parameters (lag time, infusion duration) and models with large IIV (omega^2 > 1).

## Related tools

- **mapbayr** (Core R package performing MAP-BE via mapbayest() from any mrgsolve-coded population PK model) — https://github.com/FelicienLL/mapbayr
- **mrgsolve** (Model implementation and ODE solving (LSODA) underlying mapbayr; used to simulate profiles and predict individual concentrations) — https://github.com/metrumresearchgroup/mrgsolve
- **NONMEM** (Gold-standard reference used to benchmark mapbayr MAP-BE estimates on identical models and data)
- **R** (Runtime environment; mapbayr installed from CRAN or GitHub via devtools::install_github)
- **Shiny** (Wrapping mapbayr into standalone MIPD/TDM web applications) — https://github.com/FelicienLL/mapbayr-shiny

## Examples

```
my_est <- my_model %>% adm_rows(time=0, amt=100, rate=20) %>% obs_rows(time=6, DV=3.9) %>% add_covariates(BW=90) %>% mapbayest(); get_eta(my_est); get_param(my_est, "CL")
```

## Evaluation signals

- ≥97% excellent-estimation rates vs NONMEM: 97.3% for PK parameters and 99.3% for PK outcomes across the seven real models (e.g., only 5/1000 individuals with >10% error on ibrutinib AUCtau,SS)
- Per-drug concordance rates aggregated in concordance_summary.csv classify estimations as concordant or discordant; discordances should concentrate on dose-related parameters (lag time, infusion duration) and models with large IIV (omega^2 > 1)
- Residuals vs IPRED in plot(my_est) should show observations consistent with the declared $SIGMA error model (proportional 0.05, additive 0.1 in the example)
- get_eta() returns one ETA per $OMEGA element and get_param(my_est, 'CL') returns the individualized parameter (e.g., 1.79 in the README example)

## Limitations

- No proper comparison of run times was performed; mapbayr is slower than NONMEM
- MAP estimation cannot assess ability to return the true individual parameter value (eta_i)
- Discordances vs NONMEM occur when lag time or infusion duration are estimated or when IIV variances are large (omega^2 > 1)
- No estimation-uncertainty quantification in the current version (no Fisher information matrix or posterior sampling)
- Software version numbers for mapbayr and NONMEM are not stated; both use the LSODA ODE solver

## Evidence

- [readme] single function: "mapbayr is a free and open source package for *maximum a posteriori* bayesian estimation of PK parameters in R."
- [methods] mapbayest estimation: "a unique function to perform the estimation: `mapbayest()`."
- [abstract] validation concordance: "For the test models, 98% of mapbayr estimations were identical to those given by NONMEM"
- [abstract] real-model outcomes: "For the real models, a concordance close to 100% on PK outcomes was observed."
- [discussion] large IIV discordance: "mapbayr is genuinely outperformed by NONMEM when MAP‐BE is performed with large IIV variances (ωk2 > 1) on several estimable parameters."
- [discussion] AUC derivation: "an AUC can be derived from (oral) CL and dose"
