---
name: pharmacometric-model-validation
description: Use when mapbayr and NONMEM produce discordant individual eta estimates
  (e.g., Runs 3, 4, 6 with lag-time/infusion-duration estimation, large IIV, or dose-related
  parameters) and you need to determine which estimates have lower OFV.
when_to_use_negative:
- When mapbayr and NONMEM estimates are concordant (98% of test-model estimations
  were identical), there are no discordant etas to adjudicate via OFV comparison
- When the goal is to return the true individual parameter value (ηi) — MAP estimation
  cannot assess this, only likelihood of estimates
- When a proper run-time or estimation-uncertainty comparison is needed — neither
  was performed, and current mapbayr versions provide no Fisher-information or posterior-sampling
  uncertainty quantification
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3307
tools:
- name: mapbayr
  role: Performs MAP-Bayesian estimation in R and provides the compute_ofv machinery
    (mrgsolve model, omega inverse, sigma, compartment flags, log-transformation flag)
    used to recompute OFVi from NONMEM's eta estimates
  repo: https://github.com/FelicienLL/mapbayr
- name: mrgsolve
  role: Implements the population PK models and solves ODEs needed for prediction
    and OFV computation during validation
  repo: https://github.com/metrumresearchgroup/mrgsolve
- name: NONMEM
  role: Gold-standard reference tool whose MAP-BE estimates and OFV are compared against
    mapbayr on the simulated profiles
- name: R
  role: Language in which the discordance analysis, OFV comparison, and ggplot2 OFV-comparison
    figure are produced
  repo: https://github.com/FelicienLL/mapbayr
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1002/psp4.12689
    title: Easy and reliable maximum <i>a posteriori</i> Bayesian estimation of pharmacokinetic
      parameters with the open‐source R package mapbayr
  source_content_hash: sha256:7162bf28c88a7c5935c14338118eeed3e3dd9d56c379e809c68b86827cee89c5
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
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3307
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# pharmacometric-model-validation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Validates MAP-Bayesian estimation of pharmacokinetic parameters by comparing mapbayr and NONMEM estimates across simulated profiles, and adjudicates discordant cases by recomputing individual objective function values (OFVi,NM-like) from NONMEM's eta estimates using mapbayr's compute_ofv machinery. This determines which tool found the more likely eta estimates when point estimates diverge.

## When to use

Use when mapbayr and NONMEM produce discordant individual eta estimates (e.g., Runs 3, 4, 6 with lag-time/infusion-duration estimation, large IIV, or dose-related parameters) and you need to determine which estimates have lower OFV.

## When NOT to use

- When mapbayr and NONMEM estimates are concordant (98% of test-model estimations were identical), there are no discordant etas to adjudicate via OFV comparison
- When the goal is to return the true individual parameter value (ηi) — MAP estimation cannot assess this, only likelihood of estimates
- When a proper run-time or estimation-uncertainty comparison is needed — neither was performed, and current mapbayr versions provide no Fisher-information or posterior-sampling uncertainty quantification

## Inputs

- Simulated PK profiles per test model (single/multiple dosing, rich/sparse sampling; 4000 profiles per model), stored as .csv in the mapbayr-CPTPSP-2021 repository
- mapbayr estimation outputs (.rds files) including posterior eta estimates and OFVi,MBR
- NONMEM estimation outputs (.mod control streams and .zip result files) including posterior eta estimates
- mrgsolve model definitions with $OMEGA, $SIGMA, $CMT ADM/OBS flags and log-transformation settings

## Outputs

- ofv_comparison.csv — per-patient comparison of recomputed OFVi,NM-like vs OFVi,MBR for all 371 discordant individuals
- ofv_comparison.png — ggplot2 scatter figure of mapbayr OFV vs NONMEM-like OFV for discordant cases
- Tallied fraction of discordant patients with lower OFVi,MBR (expected ~61%, i.e., 225/371 for Runs 3, 4, 6 with rich sampling)

## How to apply

Load validation materials from the mapbayr-CPTPSP-2021 repository (mapbayr .rds outputs, NONMEM .mod/.zip outputs, and simulated .csv profiles) and select the discordant individuals whose eta estimates differ between tools — e.g., the 371 discordant patients in Runs 3, 4, and 6 with rich sampling. Extract NONMEM's posterior eta estimates along with the model definition (omega matrix, sigma, residual-error and log-transformation settings), then recompute OFVi,NM-like from the NONMEM etas using mapbayr's compute_ofv machinery (mrgsolve model, omega inverse, sigma, observation/dose compartment flags, log-transformation flag). Compare the recomputed OFVi,NM-like against OFVi,MBR for every discordant patient, tallying the fraction where OFVi,MBR is lower — in the reference study this was 225/371 (61%), supporting that mapbayr often found the more likely etas. Assemble the per-patient comparison into ofv_comparison.csv and render an OFV-comparison figure (mapbayr vs NONMEM-like OFV) with ggplot2 as ofv_comparison.png.

## Related tools

- **mapbayr** (Performs MAP-Bayesian estimation in R and provides the compute_ofv machinery (mrgsolve model, omega inverse, sigma, compartment flags, log-transformation flag) used to recompute OFVi from NONMEM's eta estimates) — https://github.com/FelicienLL/mapbayr
- **mrgsolve** (Implements the population PK models and solves ODEs needed for prediction and OFV computation during validation) — https://github.com/metrumresearchgroup/mrgsolve
- **NONMEM** (Gold-standard reference tool whose MAP-BE estimates and OFV are compared against mapbayr on the simulated profiles)
- **R** (Language in which the discordance analysis, OFV comparison, and ggplot2 OFV-comparison figure are produced) — https://github.com/FelicienLL/mapbayr

## Examples

```
my_est <- my_model %>% adm_rows(time = 0, amt = 100, rate = 20) %>% obs_rows(time = 6, DV = 3.9) %>% mapbayest(); get_eta(my_est)
```

## Evaluation signals

- The computed fraction of the 371 discordant patients (Runs 3, 4, 6, rich sampling) with lower OFVi,MBR reproduces the reported 61% (225/225 counted patients)
- Overall concordance between mapbayr and NONMEM reproduces the reported 98% of identical estimations for test models and near-100% concordance on PK outcomes for the seven real models
- ofv_comparison.csv contains one row per discordant individual with both OFVi,MBR and recomputed OFVi,NM-like values
- The ggplot2 figure ofv_comparison.png renders the mapbayr vs NONMEM-like OFV comparison without errors
- Discordances, when present, localize to the known sensitive cases: estimated dose-related parameters (lag time or infusion duration) and models with large IIV variances (ωk2 > 1)

## Limitations

- The comparison depends on the authors' own OFVi,NM-like computation being correct; if it is false, the conclusion that mapbayr outperformed NONMEM is unreliable
- Discordances were observed when dose-related parameters were estimated or when models with large IIV were used (ωk2 > 1 on several estimable parameters)
- MAP-BE concordance cannot inform the ability to return the actual value of ηi
- No proper run-time comparison between mapbayr and NONMEM was performed, and no software version numbers are stated
- Current mapbayr versions provide no estimation-uncertainty quantification (no Fisher information matrix or posterior sampling)

## Evidence

- [results] OFVi,MBR was lower than the recomputed OFVi,NM-like in 225 patients (61%): [evidence span withheld — anchor status: not_found]
- [readme] validation materials repository: [evidence span withheld — anchor status: external_doc]
