---
name: mrgsolve-pk-model-implementation
description: Use when an individual PK parameter estimation (MAP-BE) task requires
  coding a population PK model in mrgsolve, e.g. a two-compartment model with IIV
  on CL and V1 and a mixed residual error model.
when_to_use_negative:
- The task is population-level model fitting (NLME), not individual MAP-BE — packages
  like nlmixr or saemix are dedicated to population PK/PD modeling
- The model cannot be expressed in mrgsolve syntax or requires NONMEM-specific features
  (e.g. NONMEM's own estimation routines)
- Only simulation without parameter estimation is needed, with no MAP-BE intention
edam_operation: http://edamontology.org/operation_3929
edam_topics:
- http://edamontology.org/topic_3398
- http://edamontology.org/topic_2258
tools:
- name: mrgsolve
  role: Implements the population PK model ($PARAM, $OMEGA, $SIGMA, $CMT blocks) and
    solves the ODE system used by mapbayr for predictions
  repo: https://github.com/metrumresearchgroup/mrgsolve
- name: mapbayr
  role: Consumes the mrgsolve model object for MAP-Bayesian estimation; the model
    must pass check_mapbayr_model()
  repo: https://github.com/FelicienLL/mapbayr
- name: R
  role: Host language in which the model is coded, compiled (C++ via mrgsolve) and
    executed
provenance:
  source_task_ids:
  - task_005
  - task_001
  source_papers:
  - doi: 10.1002/psp4.12689
    title: Easy and reliable maximum <i>a posteriori</i> Bayesian estimation of pharmacokinetic
      parameters with the open‐source R package mapbayr
  source_content_hash: sha256:73276ec9a77ffa284bdb46670c8ad247321f0f3537272c0249d09719fd0d5bda
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
  - R
  - mapbayr
  - mrgsolve
  edam_operation: http://edamontology.org/operation_3929
  edam_topics:
  - http://edamontology.org/topic_3398
  - http://edamontology.org/topic_2258
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# mrgsolve-pk-model-implementation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Implement a user-supplied population PK model in mrgsolve with annotated $PARAM, $OMEGA, $SIGMA and $CMT blocks so it can serve as input to mapbayr's MAP-Bayesian estimation and downstream simulation of PK outcomes.

## When to use

Use when an individual PK parameter estimation (MAP-BE) task requires coding a population PK model in mrgsolve, e.g. a two-compartment model with IIV on CL and V1 and a mixed residual error model.

## When NOT to use

- The task is population-level model fitting (NLME), not individual MAP-BE — packages like nlmixr or saemix are dedicated to population PK/PD modeling
- The model cannot be expressed in mrgsolve syntax or requires NONMEM-specific features (e.g. NONMEM's own estimation routines)
- Only simulation without parameter estimation is needed, with no MAP-BE intention

## Inputs

- Structural PK model specification (compartments, clearances, volumes, covariate effects)
- Inter-individual variability variances ($OMEGA, e.g. 0.3 0.3 on CL and V1)
- Residual error model specification ($SIGMA: proportional 0.05, additive 0.1)
- Covariate annotations ($PARAM @annotated @covariates, e.g. BW = 70)

## Outputs

- Compiled mrgsolve model object created via mcode()
- Validated model compatible with check_mapbayr_model() and mapbayest()
- Captured DV and CL columns via $CAPTURE for downstream estimation and simulation

## How to apply

Write the structural model in mrgsolve model code using $PARAM for typical values (e.g. TVCL=0.9, TVV1=10, Q=1, V2=10) and covariates (e.g. BW with a (BW/70)^1.2 effect on CL), $OMEGA for inter-individual variances on CL and V1 (e.g. 0.3 0.3), and $SIGMA for the residual error (proportional 0.05, additive 0.1). Declare compartments with $CMT using [ADM, OBS] annotations and compute DV in $TABLE as (CENT/V1)*(1+EPS(1))+EPS(2) so the observation column is captured. Instantiate the model with mcode() and validate it with check_mapbayr_model() before estimation, since mapbayr relies on mrgsolve for ODE solving (LSODA) to evaluate predictions in the objective function. The same model object is later updated via use_estimates() to simulate concentrations at arbitrary times.

## Related tools

- **mrgsolve** (Implements the population PK model ($PARAM, $OMEGA, $SIGMA, $CMT blocks) and solves the ODE system used by mapbayr for predictions) — https://github.com/metrumresearchgroup/mrgsolve
- **mapbayr** (Consumes the mrgsolve model object for MAP-Bayesian estimation; the model must pass check_mapbayr_model()) — https://github.com/FelicienLL/mapbayr
- **R** (Host language in which the model is coded, compiled (C++ via mrgsolve) and executed)

## Examples

```
my_model <- mcode("Example_model", code)
```

## Evaluation signals

- check_mapbayr_model() reports no structural errors on the annotated $PARAM/$OMEGA/$SIGMA/$CMT blocks
- Model compiles successfully via mcode() (requires C++ compiler / Rtools on Windows)
- DV in $TABLE is correctly computed as (CENT/V1)*(1+EPS(1))+EPS(2), matching the intended mixed residual error model
- mapbayest() run on the packaged example (exmodel/exdata) with this model reproduces reference ETAs (ETA1=0.3872104, ETA2=0.1569604) and CL=1.79217
- use_estimates() successfully updates the model with posterior ETAs and covariates, allowing mrgsim simulations to run

## Limitations

- mrgsolve requires C++ compilers; Windows users typically need to install Rtools
- Large IIV variances (ωk2 > 1) on several estimable parameters can degrade mapbayr estimation performance relative to NONMEM
- No explicit version number stated for mrgsolve or mapbayr in the article

## Evidence

- [methods] validation workflow: [evidence span withheld — anchor status: not_found]
