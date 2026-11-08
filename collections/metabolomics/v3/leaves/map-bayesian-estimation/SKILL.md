---
name: map-bayesian-estimation
description: Use when individual PK parameters (CL, V, etas) must be estimated from
  patient TDM concentrations given a user-defined population PK model, or when validating/replicating
  NONMEM MAP-BE results in R for MIPD workflows.
when_to_use_negative:
- The goal is population PK/PD model building or parameter estimation at the population
  level — use nlmixr or saemix instead, as mapbayr is dedicated to individual MAP-BE
- The Bayesian estimation design (residual error, sampling schedule) has not been
  previously validated for the intended dosing/sampling strategy
- Quantification of estimation uncertainty (e.g. Fisher information matrix, posterior
  sampling) is required — not implemented in the current mapbayr version
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_3399
- http://edamontology.org/topic_3077
tools:
- name: mapbayr
  role: 'Core package: mapbayest() performs MAP-BE from an mrgsolve model and NM-TRAN
    dataset, with helpers adm_rows/obs_rows/add_covariates, get_eta/get_param/use_estimates
    for post-processing'
  repo: https://github.com/FelicienLL/mapbayr
- name: mrgsolve
  role: Model implementation and ODE solving (LSODA) used to compute predictions during
    objective-function evaluation
  repo: https://github.com/metrumresearchgroup/mrgsolve
- name: NONMEM
  role: Gold-standard reference estimator used to validate mapbayr MAP-BE concordance
    (Delta-theta-hat thresholds 0.001/0.095)
- name: Shiny
  role: Wraps the mapbayest() estimation (model + data, two arguments) into standalone
    MIPD web applications exposing etas, CL, AUC and dose proposals
  repo: https://github.com/FelicienLL/mapbayr-shiny
provenance:
  source_task_ids:
  - task_003
  - task_005
  - task_001
  - task_002
  source_papers:
  - doi: 10.1002/psp4.12689
    title: Easy and reliable maximum <i>a posteriori</i> Bayesian estimation of pharmacokinetic
      parameters with the open‐source R package mapbayr
  source_content_hash: sha256:d2423c59fe1c031a71eebe469e94245d8d4ee359584958188cb50c1c2826a7d9
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
  - mapbayr
  - mrgsolve
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_3399
  - http://edamontology.org/topic_3077
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7222222222222222
n_claims: 18
---

# map-bayesian-estimation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Perform maximum a posteriori Bayesian estimation (MAP-BE) of individual PK parameters in R with mapbayr: code the model in mrgsolve, format dosing/observations in NM-TRAN style, and minimize the objective function over etas with L-BFGS-B, replicating NONMEM-grade estimation without external software.

## When to use

Use when individual PK parameters (CL, V, etas) must be estimated from patient TDM concentrations given a user-defined population PK model, or when validating/replicating NONMEM MAP-BE results in R for MIPD workflows.

## When NOT to use

- The goal is population PK/PD model building or parameter estimation at the population level — use nlmixr or saemix instead, as mapbayr is dedicated to individual MAP-BE
- The Bayesian estimation design (residual error, sampling schedule) has not been previously validated for the intended dosing/sampling strategy
- Quantification of estimation uncertainty (e.g. Fisher information matrix, posterior sampling) is required — not implemented in the current mapbayr version

## Inputs

- Population PK model coded in mrgsolve ($PARAM, $OMEGA, $SIGMA, $CMT blocks; .cpp model file)
- Individual dataset with concentrations in NM-TRAN format (ID, time, evid, cmt, amt, rate, DV, mdv, covariates such as BW)
- Optional NONMEM estimation results / control streams (.mod) for concordance validation
- Simulated PK profiles (e.g. from the mapbayr-CPTPSP-2021 repository, .csv)

## Outputs

- mapbayests S3 object containing posterior ETA estimates, input model/data, and internal optimizer arguments
- Output table with IPRED, PRED, individual parameters (e.g. CL) and etas per record
- Derived PK outcomes: get_eta() values, get_param() parameters, simulated concentration-time profiles after use_estimates()
- Concordance metrics vs NONMEM (Delta-theta-hat per profile, excellent/acceptable/discordant classification)

## How to apply

Code the population PK model in mrgsolve with annotated $PARAM (covariates like BW), $OMEGA (e.g. 0.3 0.3 on CL and V1), $SIGMA for the residual error (e.g. proportional 0.05, additive 0.1), and $CAPTURE of DV; validate it with check_mapbayr_model(). Assemble the individual dataset in NM-TRAN format (or via the pipe-friendly adm_rows(), obs_rows(), add_covariates() helpers) with dose records (amt, rate, evid=1), fitted observations (mdv=0) and mdv=1 dummy records. Call mapbayest(model, data), which computes predictions with mrgsolve LSODA ODE solving, evaluates the objective function from observations, Omega and Sigma, and minimizes over etas with the default L-BFGS-B optimizer. Extract posterior etas with get_eta(), derived parameters with get_param(my_est, 'CL'), and IPRED/PRED from the output table; update the model with use_estimates() to simulate concentrations at arbitrary times for dose proposals. Validate against NONMEM by computing the relative difference Delta-theta-hat per parameter and classifying estimations as excellent (< 0.001), acceptable, or discordant (>= 0.095).

## Related tools

- **mapbayr** (Core package: mapbayest() performs MAP-BE from an mrgsolve model and NM-TRAN dataset, with helpers adm_rows/obs_rows/add_covariates, get_eta/get_param/use_estimates for post-processing) — https://github.com/FelicienLL/mapbayr
- **mrgsolve** (Model implementation and ODE solving (LSODA) used to compute predictions during objective-function evaluation) — https://github.com/metrumresearchgroup/mrgsolve
- **NONMEM** (Gold-standard reference estimator used to validate mapbayr MAP-BE concordance (Delta-theta-hat thresholds 0.001/0.095))
- **Shiny** (Wraps the mapbayest() estimation (model + data, two arguments) into standalone MIPD web applications exposing etas, CL, AUC and dose proposals) — https://github.com/FelicienLL/mapbayr-shiny

## Examples

```
my_est <- my_model %>% adm_rows(time=0, amt=100, rate=20) %>% obs_rows(time=6, DV=3.9) %>% obs_rows(time=15, DV=1.1) %>% add_covariates(BW=90) %>% mapbayest(); get_param(my_est, "CL")
```

## Evaluation signals

- On the packaged example (two-compartment model, 100 mg infusion at 20 mg/h, DV=3.9 at 6 h and DV=1.1 at 15 h, BW=90), posterior estimates must reproduce ETA1=0.3872104, ETA2=0.1569604, CL=1.79217
- Across the 35 test models x 4000 profiles, median Delta-theta-hat should be ~0.0004% with ~98% excellent (< 0.001), ~0.5% acceptable, ~1.5% discordant (>= 0.095) versus NONMEM
- check_mapbayr_model() passes on the input mrgsolve model before estimation
- A Shiny app built on mapbayest() must return ETAs and derived CL identical to a headless mapbayest() run on exmodel/exdata
- In discordant lag-time/infusion-duration cases (Runs 3, 4, 6, rich sampling), recomputed OFVi,NM-like should be higher than OFVi,MBR in ~61% (225/371) of discordant patients, indicating mapbayr found the more likely etas

## Limitations

- Discordances vs NONMEM arise when dose-related parameters (lag time, infusion duration) are estimated or with large IIV variances (omega^2 > 1) on several estimable parameters; for a few individuals NONMEM is genuinely better in the large-IIV case
- No proper run-time comparison was performed, but mapbayr is undoubtedly slower than NONMEM
- MAP-BE cannot assess the ability to return the true individual parameter value (eta_i); only OFV exploration (mapbayr OFVi lower in ~61% of discordant cases) partially addresses this, provided the OFVi,NM-like recomputation is correct
- No estimation-uncertainty quantification in the current version (future versions could use the Fisher information matrix or posterior sampling)

## Evidence

- [methods] mapbayr computes the OFV from the observations (Yij) in the data set together with the Omega and Sigma matrices defined in the model code, and then minimizes it by optimizing eta_i with the limited-memory Broyden-Fletcher-Goldfarb-Shanno with box constraints (L-BFGS-B) algorithm.: [evidence span withheld — anchor status: not_found]
- [abstract] For the test models, 98% of mapbayr estimations were identical to those given by NONMEM: "For the test models, 98% of mapbayr estimations were identical to those given by NONMEM"
- [readme] mapbayr is a free and open source package for *maximum a posteriori* bayesian estimation of PK parameters in R. Thanks to a single function, `mapbayest()`, you can estimate individual PK parameters: "Thanks to a single function, `mapbayest()`, you can estimate individual PK parameters from: - a population PK model (coded in [mrgsolve](https://github.com/metrumresearchgroup/mrgsolve)), - a data"
- [discussion] Discordances vs NONMEM were observed with large IIV variances (omega^2 > 1) on several parameters: "for a few individuals, mapbayr is genuinely outperformed by NONMEM when MAP‐BE is performed with large IIV variances (ωk2 > 1) on several estimable parameters."
- [results] Classify each comparison as excellent if all Delta-theta-hat < 0.001, discordant if any Delta-theta-hat >= 0.095, and otherwise acceptable.: [evidence span withheld — anchor status: not_found]
