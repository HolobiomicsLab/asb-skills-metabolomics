---
name: nonmem-map-estimation-execution
description: Use when replicating or benchmarking individual parameter estimation
  in NONMEM for simulated PK profiles, especially to cross-check mapbayr or another
  open-source MAP-BE tool on the same model-profile pairs.
when_to_use_negative:
- The goal is population-level PK/PD model building or parameter estimation with mixed
  effects, not individual MAP-BE (use population tools like nlmixr or saemix).
- No NONMEM license is available — mapbayr alone performs the equivalent MAP-BE in
  R without external nonlinear mixed effect modeling software.
- Estimating dose-related parameters (lag time, infusion duration) or models with
  large IIV variances (ωk2 > 1), where NONMEM-mapbayr discordances are expected and
  OFVi exploration suggests mapbayr may be more likely.
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_3307
- http://edamontology.org/topic_3379
tools:
- name: NONMEM
  role: Gold-standard reference implementation of MAP-BE executed via deposited control
    streams to generate comparator individual parameter estimates
- name: mapbayr
  role: Open-source R package whose mapbayest() (L-BFGS-B) estimates are validated
    against NONMEM output using Δθ̂ thresholds
  repo: https://github.com/FelicienLL/mapbayr
- name: mrgsolve
  role: ODE solver and model-coding framework underlying mapbayr model implementation
    in the comparison
  repo: https://github.com/metrumresearchgroup/mrgsolve
- name: R
  role: Environment for loading simulated profiles, running mapbayest(), and computing
    Δθ̂ classifications
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1002/psp4.12689
    title: Easy and reliable maximum <i>a posteriori</i> Bayesian estimation of pharmacokinetic
      parameters with the open‐source R package mapbayr
  source_content_hash: sha256:95fde5b153fec379e1e6edc0335454a8eaf95b2109cd5f7b0c80840e25117814
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
  - http://edamontology.org/topic_3307
  - http://edamontology.org/topic_3379
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# nonmem-map-estimation-execution

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Perform maximum a posteriori Bayesian estimation (MAP-BE) of individual PK parameters in NONMEM using deposited control streams, as the reference implementation against which mapbayr's mapbayest() estimates are validated across 35 test models and 4000 simulated profiles each.

## When to use

Use when replicating or benchmarking individual parameter estimation in NONMEM for simulated PK profiles, especially to cross-check mapbayr or another open-source MAP-BE tool on the same model-profile pairs.

## When NOT to use

- The goal is population-level PK/PD model building or parameter estimation with mixed effects, not individual MAP-BE (use population tools like nlmixr or saemix).
- No NONMEM license is available — mapbayr alone performs the equivalent MAP-BE in R without external nonlinear mixed effect modeling software.
- Estimating dose-related parameters (lag time, infusion duration) or models with large IIV variances (ωk2 > 1), where NONMEM-mapbayr discordances are expected and OFVi exploration suggests mapbayr may be more likely.

## Inputs

- NONMEM control streams (.mod files) from github.com/FelicienLL/mapbayr-CPTPSP-2021
- Simulated PK profiles in .csv format (4000 profiles per test model, 35 models)
- Population PK model structure with $OMEGA/$SIGMA definitions (single/multiple dosing, rich/sparse sampling)

## Outputs

- NONMEM MAP-BE individual parameter estimates (thetas/etas) per profile
- Per-model-profile Δθ̂ relative differences vs mapbayr estimates
- Aggregated classification percentages (excellent/acceptable/discordant) and median Δθ̂

## How to apply

Use the deposited NONMEM control streams (.mod files) from the mapbayr-CPTPSP-2021 repository to run MAP Bayesian estimation for each of the 4000 simulated PK profiles per test model, covering default, bicompartmental, time-varying covariate, alternative residual error, Michaelis-Menten, lag time/DUR, and large-IIV models. For every model-profile pair, extract the individual parameter estimates and compute the relative difference (Δθ̂) between NONMEM and mapbayr estimates for all estimated parameters. Classify each comparison as excellent if all Δθ̂ < 0.001, discordant if any Δθ̂ >= 0.095, and otherwise acceptable. Accept the validation output if the aggregate summary matches the reported benchmark of 98% excellent, 1.5% discordant, and median Δθ̂ of 0.0004%, allowing reasonable optimizer-tolerance variability. Note that both NONMEM and mapbayr use the LSODA ODE solver, so residual differences stem from optimization, not integration.

## Related tools

- **NONMEM** (Gold-standard reference implementation of MAP-BE executed via deposited control streams to generate comparator individual parameter estimates)
- **mapbayr** (Open-source R package whose mapbayest() (L-BFGS-B) estimates are validated against NONMEM output using Δθ̂ thresholds) — https://github.com/FelicienLL/mapbayr
- **mrgsolve** (ODE solver and model-coding framework underlying mapbayr model implementation in the comparison) — https://github.com/metrumresearchgroup/mrgsolve
- **R** (Environment for loading simulated profiles, running mapbayest(), and computing Δθ̂ classifications)

## Examples

```
nmfe74 run1.mod run1.lst  # per-profile MAP-BE using control streams from github.com/FelicienLL/mapbayr-CPTPSP-2021 nonmem_code/
```

## Evaluation signals

- Median Δθ̂ across all 140,000 model-profile estimations approximates 0.0004%
- ~98% of estimations fall in the excellent class (all Δθ̂ < 0.1%) and ~1.5% in the discordant class (any Δθ̂ > 10%)
- Per-profile Δθ̂ computed for every estimated parameter; classification is excellent only if ALL Δθ̂ < 0.001 and discordant if ANY Δθ̂ >= 0.095
- Residual disagreement stays within reasonable optimizer-tolerance variability given both tools use the LSODA solver
- For real published models used in MIPD, PK-outcome concordance with mapbayr approaches 100%

## Limitations

- Discordances arise when dose-related parameters (lag time, infusion duration) are estimated or when models have large IIV variances (ωk2 > 1); OFVi exploration suggested mapbayr estimates were often more likely, so NONMEM is not always superior.
- No proper run-time comparison was performed, and no software version numbers are stated for mapbayr or NONMEM.
- The proprietary OFVi,NM-like computation used in the comparison could be false when comparing estimates.

## Evidence

- [readme] nonmem control streams deposited in validation repo: [evidence span withheld — anchor status: external_doc]
