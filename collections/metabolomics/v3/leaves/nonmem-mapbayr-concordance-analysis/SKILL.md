---
name: nonmem-mapbayr-concordance-analysis
description: Use when validating an open-source MAP-BE implementation (e.g. mapbayr)
  against NONMEM on simulated PK profiles, especially when discrepancies arise from
  estimated lag time, infusion duration, or models with large IIV (omega^2 > 1).
when_to_use_negative:
- Input is a clinical dataset needing individualized dosing rather than a software-validation
  comparison against NONMEM
- The goal is assessing whether MAP-BE recovers the true individual eta values (eta_i)
  — MAP estimation cannot inform this
- A runtime benchmark is required — no proper mapbayr vs NONMEM runtime comparison
  was performed and mapbayr is acknowledged to be slower
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_2269
- http://edamontology.org/topic_0091
tools:
- name: mapbayr
  role: Performs MAP-BE with mapbayest(), provides compute_ofv machinery, get_eta()/get_param()
    extraction, and check_mapbayr_model() validation
  repo: https://github.com/FelicienLL/mapbayr
- name: mrgsolve
  role: Implements the population PK models ($PARAM/$OMEGA/$SIGMA/$CMT/$ODE) and solves
    ODEs for prediction during estimation and OFV recomputation
  repo: https://github.com/metrumresearchgroup/mrgsolve
- name: NONMEM
  role: Gold-standard MAP-BE control arm run on the same profiles with identical models,
    data, and LSODA solver to generate reference etas and OFVi
- name: ggplot2
  role: Renders the OFV-comparison figure (ofv_comparison.png) of mapbayr vs NONMEM-like
    OFV for discordant patients
- name: R
  role: Environment for the whole comparison workflow, including dplyr-based aggregation
    of concordance rates
provenance:
  source_task_ids:
  - task_003
  - task_004
  source_papers:
  - doi: 10.1002/psp4.12689
    title: Easy and reliable maximum <i>a posteriori</i> Bayesian estimation of pharmacokinetic
      parameters with the open‐source R package mapbayr
  source_content_hash: sha256:a15615b1d2ecb9c75934e3785012120d464c55b2c855a7a8b7cb71923a82e37d
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
  - dplyr
  - ggplot2
  - mapbayr
  - mrgsolve
  edam_operation: http://edamontology.org/operation_2990
  edam_topics:
  - http://edamontology.org/topic_2269
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7857142857142857
n_claims: 14
---

# nonmem-mapbayr-concordance-analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A validation workflow that performs MAP-Bayesian estimation of individual PK parameters with both mapbayr (R/mrgsolve) and NONMEM on the same simulated profiles, classifies each estimation as concordant or discordant, and, for discordant cases, recomputes OFVi from NONMEM's eta estimates to determine which tool found the more likely etas.

## When to use

Use when validating an open-source MAP-BE implementation (e.g. mapbayr) against NONMEM on simulated PK profiles, especially when discrepancies arise from estimated lag time, infusion duration, or models with large IIV (omega^2 > 1).

## When NOT to use

- Input is a clinical dataset needing individualized dosing rather than a software-validation comparison against NONMEM
- The goal is assessing whether MAP-BE recovers the true individual eta values (eta_i) — MAP estimation cannot inform this
- A runtime benchmark is required — no proper mapbayr vs NONMEM runtime comparison was performed and mapbayr is acknowledged to be slower

## Inputs

- Simulated PK profiles in NM-TRAN format (.csv) with dosing, sampling times, DV, mdv
- Population PK model definitions: omega matrix, sigma, residual-error and log-transformation settings (mrgsolve .cpp and NONMEM .mod files)
- NONMEM posterior eta estimates per individual
- mapbayr estimation outputs (.rds) including OFVi,MBR and ETAs

## Outputs

- Per-patient concordance/discordance classification per drug and per PK parameter/outcome
- ofv_comparison.csv with recomputed OFVi,NM-like vs OFVi,MBR for discordant patients
- OFV-comparison figure (ofv_comparison.png) rendered with ggplot2
- concordance_summary.csv aggregating per-drug concordance rates

## How to apply

Load validation materials from the mapbayr-CPTPSP-2021 repository (data as .csv, mapbayr code as .cpp, NONMEM code as .mod, outputs as .rds/.zip) and translate each population PK model into mrgsolve code ($PARAM, $OMEGA, $SIGMA, $CMT, $ODE), checking it with check_mapbayr_model(). Simulate profiles (e.g. 1000 per real model or 4000 per test model, combining single/multiple dosing with rich/sparse sampling) by sampling ETAs from the published omega matrix, then run mapbayest() with the default L-BFGS-B algorithm and the corresponding NONMEM control streams with the same data and LSODA solver. Extract posterior ETAs with get_eta() and parameters with get_param(), and classify estimations as concordant or discordant per drug on both estimated PK parameters and MIPD outcomes (e.g. AUCtau,SS from oral CL and dose, trough concentrations). For discordant individuals (e.g. the 371 discordant patients in Runs 3, 4, and 6 with rich sampling), recompute OFVi,NM-like from NONMEM's eta estimates using mapbayr's compute_ofv machinery (mrgsolve model, omega inverse, sigma, observation/dose compartment flags, log-transformation flag) and compare against OFVi,MBR per patient. Accept the validation when concordance rates approach published values (98% identical estimations for test models; 97.3% and 99.3% excellent estimations for parameters and PK outcomes, respectively) and OFV comparisons reproduce reported fractions (e.g. 225/371 ≈ 61% of discordant patients with lower OFVi,MBR).

## Related tools

- **mapbayr** (Performs MAP-BE with mapbayest(), provides compute_ofv machinery, get_eta()/get_param() extraction, and check_mapbayr_model() validation) — https://github.com/FelicienLL/mapbayr
- **mrgsolve** (Implements the population PK models ($PARAM/$OMEGA/$SIGMA/$CMT/$ODE) and solves ODEs for prediction during estimation and OFV recomputation) — https://github.com/metrumresearchgroup/mrgsolve
- **NONMEM** (Gold-standard MAP-BE control arm run on the same profiles with identical models, data, and LSODA solver to generate reference etas and OFVi)
- **ggplot2** (Renders the OFV-comparison figure (ofv_comparison.png) of mapbayr vs NONMEM-like OFV for discordant patients)
- **R** (Environment for the whole comparison workflow, including dplyr-based aggregation of concordance rates)

## Examples

```
devtools::install_github("FelicienLL/mapbayr"); my_est <- my_model %>% adm_rows(time=0, amt=100, rate=20) %>% obs_rows(time=6, DV=3.9) %>% mapbayest(); get_eta(my_est)
```

## Evaluation signals

- Concordance of mapbayr vs NONMEM estimations reaches ~98% for test models and near-100% concordance on PK outcomes for the seven real models
- Excellent-estimation rates approach the published 97.3% (PK parameters) and 99.3% (PK outcomes), e.g. only 5/1000 individuals with >10% error on ibrutinib AUCtau,SS
- For discordant patients (n=371 in Runs 3, 4, 6 rich sampling), OFVi,MBR is lower than recomputed OFVi,NM-like in approximately 61% (225/371) of cases
- Residual discordances concentrate on dose-related parameters (lag time, infusion duration) and models with large IIV (omega^2 > 1)
- check_mapbayr_model() passes for every translated model, including the sunitinib parent-plus-metabolite model

## Limitations

- Proprietary OFVi,NM-like computation could be false when comparing estimates ('provided that our own computation of OFVi,NM-like is not false')
- MAP estimation cannot assess the ability to return the true individual parameter value (eta_i)
- Discrepancies occur when dose-related parameters (lag time, infusion duration) are estimated or when models with large IIV variances (omega^2 > 1) are used, where NONMEM may genuinely outperform mapbayr for a few individuals
- No proper comparison of run times between mapbayr and NONMEM was performed

## Evidence

- [other] OFVi,MBR was lower than the recomputed OFVi,NM-like in 225 patients (61%): [evidence span withheld — anchor status: not_found]
- [readme] repository contents for validation: [evidence span withheld — anchor status: external_doc]
