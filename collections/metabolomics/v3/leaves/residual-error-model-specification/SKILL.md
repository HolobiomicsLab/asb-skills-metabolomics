---
name: residual-error-model-specification
description: Use when coding a population PK model in mrgsolve for MAP-BE with mapbayr
  and observations require a combined proportional (e.g. 0.05) and additive (e.g.
  0.1) residual error structure on predicted concentrations.
when_to_use_negative:
- The goal is population PK/PD modeling (estimating fixed effects from rich population
  data) — use nlmixr or saemix instead of mapbayr
- The model is not coded in mrgsolve, since mapbayr relies on mrgsolve for model implementation
  and ODE solving
- Models with large IIV variances (ωk2 > 1) or dose-related parameter estimation,
  where mapbayr showed discordances versus NONMEM
edam_operation: http://edamontology.org/operation_3660
edam_topics:
- http://edamontology.org/topic_0218
tools:
- name: mapbayr
  role: Performs MAP-BE; reads the $SIGMA and $TABLE-specified DV to compute the objective
    function and optimize ETA via mapbayest() with L-BFGS-B
  repo: github.com/FelicienLL/mapbayr
- name: mrgsolve
  role: Hosts the $SIGMA residual error specification and $TABLE DV definition; provides
    LSODA ODE predictions used to evaluate the objective function
  repo: github.com/metrumresearchgroup/mrgsolve
- name: NONMEM
  role: Reference implementation used to validate concordance (~98% on test models)
    of estimation under the same residual error structure
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1002/psp4.12689
    title: Easy and reliable maximum <i>a posteriori</i> Bayesian estimation of pharmacokinetic
      parameters with the open‐source R package mapbayr
  source_content_hash: sha256:358b0002350accd2088f5de047bc76aeebb551b190d28c111c01a444bb840293
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/FelicienLL/mapbayr
  tools:
  - NONMEM
  - R
  - mapbayr
  - mrgsolve
  edam_operation: http://edamontology.org/operation_3660
  edam_topics:
  - http://edamontology.org/topic_0218
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5625
n_claims: 16
---

# Residual error model specification

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Specification of a mixed residual error model (proportional and additive components) directly in the mrgsolve model code ($SIGMA plus $TABLE) so that mapbayr can evaluate its objective function from the observed DV, Omega, and Sigma matrices during MAP-Bayesian estimation.

## When to use

Use when coding a population PK model in mrgsolve for MAP-BE with mapbayr and observations require a combined proportional (e.g. 0.05) and additive (e.g. 0.1) residual error structure on predicted concentrations.

## When NOT to use

- The goal is population PK/PD modeling (estimating fixed effects from rich population data) — use nlmixr or saemix instead of mapbayr
- The model is not coded in mrgsolve, since mapbayr relies on mrgsolve for model implementation and ODE solving
- Models with large IIV variances (ωk2 > 1) or dose-related parameter estimation, where mapbayr showed discordances versus NONMEM

## Inputs

- mrgsolve model code block ($PARAM, $OMEGA, $SIGMA, $CMT, $TABLE, $MAIN, $ODE, $CAPTURE)
- Residual error variances: proportional 0.05 and additive 0.1 for the mixed error model
- Individual concentration observations in NM-TRAN format (e.g. DV=3.9 at 6 h, DV=1.1 at 15 h, mdv=1 at 24 h)

## Outputs

- Compiled mrgsolve model object with $SIGMA (0.05 proportional, 0.1 additive) and $TABLE-defined DV
- Captured DV column feeding the mapbayr objective function evaluation
- mapbayests object with IPRED and PRED columns reflecting the specified error model

## How to apply

Declare the residual error variances in a $SIGMA block of the mrgsolve model code — for the two-compartment example, 0.05 for the proportional component and 0.1 for the additive component. Apply both components to the captured prediction in $TABLE by defining DV = (CENT/V1) * (1 + EPS(1)) + EPS(2), where EPS(1) scales the proportional error and EPS(2) adds the additive term. Capture DV and the parameter of interest (e.g. CL) with $CAPTURE so mapbayest() can evaluate the objective function from the observed DV values together with the Omega ($OMEGA 0.3 0.3 on CL and V1) and Sigma matrices. mapbayr accepts additive, proportional, mixed, or exponential residual error models without prior log-transformation of data, and both mapbayr and NONMEM use the LSODA ODE solver to predict concentrations for OFV evaluation.

## Related tools

- **mapbayr** (Performs MAP-BE; reads the $SIGMA and $TABLE-specified DV to compute the objective function and optimize ETA via mapbayest() with L-BFGS-B) — github.com/FelicienLL/mapbayr
- **mrgsolve** (Hosts the $SIGMA residual error specification and $TABLE DV definition; provides LSODA ODE predictions used to evaluate the objective function) — github.com/metrumresearchgroup/mrgsolve
- **NONMEM** (Reference implementation used to validate concordance (~98% on test models) of estimation under the same residual error structure)

## Examples

```
my_est <- mapbayest(my_model, data = my_data)
```

## Evaluation signals

- The $SIGMA block contains exactly two entries (0.05 proportional, 0.1 additive) matching the intended mixed residual error model
- The $TABLE defines DV as (CENT/V1) * (1 + EPS(1)) + EPS(2), applying both error components to the captured prediction
- mapbayest() posterior estimates reproduce the reference values ETA1=0.3872104, ETA2=0.1569604, CL=1.79217
- The output table contains IPRED and PRED columns consistent with the error model (e.g. IPRED=4.162 vs DV=3.9 at 6 h)
- Estimation concordance with NONMEM on comparable models approaches the reported 98% for test models and ~100% for real models

## Limitations

- Discordances vs NONMEM occurred with large IIV variances (ωk2 > 1) on several parameters and when dose-related parameters (lag time, infusion duration) were estimated
- No proper comparison of run times between mapbayr and NONMEM was performed, though mapbayr is expected to be slower
- The current mapbayr version provides no estimation-uncertainty quantification (no Fisher information matrix or posterior sampling)

## Evidence

- [readme] $SIGMA
0.05 // proportional
0.1 // additive: "0.05 // proportional
0.1 // additive"
