---
name: pk-posterior-parameter-extraction
description: Use when a mapbayest() fit has completed and you need the posterior ETA
  values, derived individual parameters (e.g. CL), or the IPRED/PRED columns of the
  output table for reporting, dosing, or simulation.
when_to_use_negative:
- You need estimation uncertainty quantification — the current mapbayr version does
  not implement Fisher information matrix or posterior sampling procedures
- Your goal is population parameter estimation (NLME fitting), which requires packages
  like nlmixr or saemix, not individual MAP-BE extraction
- The model has very large IIV variances (ωk2 > 1) on estimable parameters, where
  NONMEM may genuinely outperform mapbayr
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3379
- http://edamontology.org/topic_2269
tools:
- name: mapbayr
  role: Provides get_eta(), get_param(), use_estimates(), and the mapbayests output
    object from which posterior ETAs, derived parameters, and IPRED/PRED are extracted
  repo: https://github.com/FelicienLL/mapbayr
- name: mrgsolve
  role: Underlying model implementation and ODE solver whose model object is updated
    with estimated ETAs/covariates by use_estimates() for later simulation
  repo: https://github.com/metrumresearchgroup/mrgsolve
- name: R
  role: Runtime environment in which mapbayr extraction functions and downstream ggplot2/tibble
    simulation scripting are executed
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1002/psp4.12689
    title: Easy and reliable maximum <i>a posteriori</i> Bayesian estimation of pharmacokinetic
      parameters with the open‐source R package mapbayr
  source_content_hash: sha256:cb7c8a26de8a7c9c452ff75adf0f6279cca2e828d28ff3b99ae4799e48da6855
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
  - R
  - mapbayr
  - mrgsolve
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3379
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# PK posterior parameter extraction

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Extracts a posteriori individual PK parameters (posterior ETAs, derived parameters like CL) and IPRED/PRED predictions from the single 'mapbayests' S3 object returned by mapbayest(), enabling downstream computation of outcomes of interest for MIPD.

## When to use

Use when a mapbayest() fit has completed and you need the posterior ETA values, derived individual parameters (e.g. CL), or the IPRED/PRED columns of the output table for reporting, dosing, or simulation.

## When NOT to use

- You need estimation uncertainty quantification — the current mapbayr version does not implement Fisher information matrix or posterior sampling procedures
- Your goal is population parameter estimation (NLME fitting), which requires packages like nlmixr or saemix, not individual MAP-BE extraction
- The model has very large IIV variances (ωk2 > 1) on estimable parameters, where NONMEM may genuinely outperform mapbayr

## Inputs

- mapbayests S3 object returned by mapbayest() (contains model, data, eta estimates, and output tables)
- mrgsolve model code with $OMEGA (e.g. 0.3 0.3) and $SIGMA (proportional 0.05, additive 0.1) defining estimable parameters
- NM-TRAN format dataset or adm_rows()/obs_rows()/add_covariates() assembled records with DV, mdv, and covariates (e.g. BW=90)

## Outputs

- Posterior ETA vector via get_eta() (e.g. ETA1=0.3872104, ETA2=0.1569604)
- Derived individual parameters via get_param(my_est, 'CL') (e.g. CL=1.79217 L/h)
- Output table with IPRED, PRED, CL, and ETA columns per record
- Updated mrgsolve model object via use_estimates() for subsequent simulation scenarios

## How to apply

After running mapbayest() on an mrgsolve model and NM-TRAN-style dataset, treat the returned 'mapbayests' S3 object as the single container of inputs, outputs, and internal optimizer arguments. Call get_eta() to obtain the a posteriori ETA vector (e.g. ETA1=0.3872104, ETA2=0.1569604 for the two-compartment example) and get_param(my_est, 'CL') for derived parameters computed from the ETAs and covariates (e.g. CL=1.79217 with BW=90). Read the IPRED and PRED columns of the output table to check individual and population predictions at each record, including mdv=1 records. To propagate estimates into new simulations, pipe the object through use_estimates(), which updates the mrgsolve model with the estimated ETA values and covariates so standard mrgsolve/mrgsim workflows can be used. Verify extraction by confirming the ETAs reproduce the reference values and that IPRED tracks the observed DV (e.g. DV=3.9 vs IPRED=4.162 at 6 h).

## Related tools

- **mapbayr** (Provides get_eta(), get_param(), use_estimates(), and the mapbayests output object from which posterior ETAs, derived parameters, and IPRED/PRED are extracted) — https://github.com/FelicienLL/mapbayr
- **mrgsolve** (Underlying model implementation and ODE solver whose model object is updated with estimated ETAs/covariates by use_estimates() for later simulation) — https://github.com/metrumresearchgroup/mrgsolve
- **R** (Runtime environment in which mapbayr extraction functions and downstream ggplot2/tibble simulation scripting are executed)

## Examples

```
my_est <- mapbayest(my_model, data = my_data); get_eta(my_est); get_param(my_est, 'CL'); updated_model <- my_est %>% use_estimates()
```

## Evaluation signals

- get_eta() returns the reference posterior values (ETA1=0.3872104, ETA2=0.1569604) for the reference two-compartment example dataset
- get_param(my_est, 'CL') reproduces CL=1.79217 for the BW=90, TVCL=0.9, (BW/70)^1.2 model
- IPRED values in the output table track the fitted DV observations (e.g. IPRED=4.162 vs DV=3.9 at 6 h; IPRED=1.087 vs DV=1.1 at 15 h)
- use_estimates() yields a model that simulates like a regular mrgsolve model with the posterior ETAs and covariates set
- Posterior estimates are consistent with the reported 98% mapbayr-NONMEM concordance on test models

## Limitations

- No estimation-uncertainty quantification in the current mapbayr version; future versions could add Fisher information matrix or posterior sampling procedures
- MAP estimation cannot assess the ability to return the true individual parameter value (ηi)
- Discordances vs NONMEM can occur when dose-related parameters (lag time, infusion duration) are estimated or with large IIV variances (ωk2 > 1)

## Evidence
