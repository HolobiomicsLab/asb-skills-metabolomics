---
name: population-pk-model-simulation
description: Use when you need simulated concentration-time profiles from a population
  PK model (e.g. 4000 profiles per model combining single/multiple dosing and rich/sparse
  sampling) to validate estimation tools or explore dosing scenarios.
when_to_use_negative:
- When the goal is estimating individual parameters from observed concentrations —
  use MAP-Bayesian estimation (mapbayest()) instead, simulation only supports it
- When model-based design optimality is required — the article points to dedicated
  software (PFIM or PopED), not plain mrgsolve simulation
- When the model cannot be expressed in mrgsolve's ODE framework (the README notes
  C++ compilers/Rtools are required)
edam_operation: http://edamontology.org/operation_2426
edam_topics:
- http://edamontology.org/topic_3390
- http://edamontology.org/topic_3070
tools:
- name: mrgsolve
  role: Model implementation and ODE-based simulation of population PK profiles
  repo: https://github.com/metrumresearchgroup/mrgsolve
- name: mapbayr
  role: MAP-Bayesian estimation whose results (use_estimates) seed individualized
    simulation scenarios
  repo: https://github.com/FelicienLL/mapbayr
- name: NONMEM
  role: Gold-standard software receiving the same simulated profiles for cross-tool
    validation
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1002/psp4.12689
    title: Easy and reliable maximum <i>a posteriori</i> Bayesian estimation of pharmacokinetic
      parameters with the open‐source R package mapbayr
  source_content_hash: sha256:100e903a0728988b726e3513187073c9e8e09a5a029527964b5aa7b266940b16
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/FelicienLL/mapbayr
  repo_url: https://github.com/metrumresearchgroup/mrgsolve
  tools:
  - NONMEM
  - R
  - mapbayr
  - mrgsolve
  edam_operation: http://edamontology.org/operation_2426
  edam_topics:
  - http://edamontology.org/topic_3390
  - http://edamontology.org/topic_3070
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4666666666666667
n_claims: 15
---

# population-pk-model-simulation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Simulate PK profiles from user-defined population pharmacokinetic models coded in mrgsolve, optionally using MAP-Bayesian-estimated individual parameters (ETA) to generate concentration-time predictions for validation or MIPD scenarios.

## When to use

Use when you need simulated concentration-time profiles from a population PK model (e.g. 4000 profiles per model combining single/multiple dosing and rich/sparse sampling) to validate estimation tools or explore dosing scenarios.

## When NOT to use

- When the goal is estimating individual parameters from observed concentrations — use MAP-Bayesian estimation (mapbayest()) instead, simulation only supports it
- When model-based design optimality is required — the article points to dedicated software (PFIM or PopED), not plain mrgsolve simulation
- When the model cannot be expressed in mrgsolve's ODE framework (the README notes C++ compilers/Rtools are required)

## Inputs

- Population PK model coded in mrgsolve ($PARAM/$OMEGA/$SIGMA/$CMT/$ODE blocks, .cpp files)
- NM-TRAN format dataset (.csv) with ID, time, evid, cmt, amt, rate, DV, mdv, covariates
- Estimated individual parameters (ETA) and covariates from a mapbayests object

## Outputs

- Simulated concentration-time profiles (data.frame from mrgsim(output = "df"))
- Benchmark validation datasets: 4000 simulated PK profiles per test model (.csv in mapbayr-CPTPSP-2021)
- Individual concentration predictions (IPRED/PRED) at arbitrary times

## How to apply

Code the structural model in mrgsolve ($PARAM, $OMEGA, $SIGMA, $CMT, $ODE), install mapbayr from GitHub via devtools::install_github("FelicienLL/mapbayr"), and simulate profiles with mrgsim. For individualized simulation, first run mapbayest(), then update the model with use_estimates() so predicted concentrations reflect posterior ETA values and covariates. Define simulation scenarios as a data frame of doses (e.g. 50/100/200/500 mg with rate = amt/5) and simulate each with mrgsim(output = "df", recover = "scenario"). The validated benchmark repository mapbayr-CPTPSP-2021 stores 4000 simulated profiles per test model as .csv files that were passed to both NONMEM (.mod control streams) and mapbayr (.cpp models) for the Delta-theta-hat concordance study.

## Related tools

- **mrgsolve** (Model implementation and ODE-based simulation of population PK profiles) — https://github.com/metrumresearchgroup/mrgsolve
- **mapbayr** (MAP-Bayesian estimation whose results (use_estimates) seed individualized simulation scenarios) — https://github.com/FelicienLL/mapbayr
- **NONMEM** (Gold-standard software receiving the same simulated profiles for cross-tool validation)

## Examples

```
updated_model <- my_est %>% use_estimates(); simdat <- updated_model %>% data_set(subset(scenarios, scenario == i)) %>% mrgsim(output = "df", recover = "scenario")
```

## Evaluation signals

- Reproduced simulation summary matches the published benchmark: median Δθ̂ of 0.0004%, 98% excellent (Δθ̂ < 0.1%), 1.5% discordant (Δθ̂ > 10%) across 140,000 test-model estimations
- Each test model yields exactly 4000 simulated profiles combining single/multiple dosing and rich/sparse sampling
- Simulated profiles round-trip correctly: use_estimates() then mrgsim produces predictions consistent with mapbayest IPRED values
- Output data.frame contains expected columns (ID, time, DV/IPRED, scenario) and simulated concentrations remain physiologically plausible (non-negative, monotone decay after dose)

## Limitations

- Discordances vs NONMEM arise when dose-related parameters (lag time, infusion duration) are estimated or when models have large IIV variances (ωk2 > 1)
- Simulated-profile validation cannot assess the ability to return the true individual parameter value (ηi)
- MAP-Bayesian estimation design (residual error, sampling schedule) must be previously validated before simulation-based predictions are trusted for dosing

## Evidence

- [readme] validation repository contents: [evidence span withheld — anchor status: external_doc]
