---
name: pk-outcome-computation-from-posterior-parameters
description: Use when a mapbayest() estimation object is available and downstream
  MIPD outcomes such as CL, AUC, trough concentration, or a proposed dose must be
  derived from the estimated ETAs.
when_to_use_negative:
- No MAP-Bayesian estimation has been performed yet — build and validate the model
  (check_mapbayr_model()) and run mapbayest() first, since outcomes depend on posterior
  ETAs
- Population-level (typical-value) simulation without individual TDM data is intended
  — posterior ETAs are not identifiable without observations
- Non-PK parameter estimation tasks (e.g. population fitting with nlmixr/saemix or
  NONMEM model development), which this mapbayr post-estimation chain does not address
tools:
- name: mapbayr
  role: Provides mapbayest() estimation objects plus get_eta(), get_param() and use_estimates()
    to extract posterior parameters and update the model
  repo: https://github.com/FelicienLL/mapbayr
- name: mrgsolve
  role: Simulation engine used to predict individual concentrations at arbitrary times
    after the model is updated with use_estimates()
  repo: https://github.com/metrumresearchgroup/mrgsolve
- name: Shiny
  role: Interface layer in which the derived PK outcomes (dose proposal, trough concentration)
    are rendered for MIPD
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1002/psp4.12689
    title: Easy and reliable maximum <i>a posteriori</i> Bayesian estimation of pharmacokinetic
      parameters with the open‐source R package mapbayr
  source_content_hash: sha256:033031da6fed0407ff00a9e4c0210d38d9c59017556c8b1b492f0bee33e794c9
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
  - Shiny
  - mapbayr
  - mrgsolve
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6153846153846154
n_claims: 13
---

# pk-outcome-computation-from-posterior-parameters

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Computes clinically relevant PK outcomes (individual clearance, AUC, predicted concentrations at arbitrary times, dose proposals) from the a posteriori parameters of a mapbayr MAP-Bayesian estimation, by extracting ETAs with get_eta()/get_param() and updating the mrgsolve model with use_estimates() before simulation.

## When to use

Use when a mapbayest() estimation object is available and downstream MIPD outcomes such as CL, AUC, trough concentration, or a proposed dose must be derived from the estimated ETAs.

## When NOT to use

- No MAP-Bayesian estimation has been performed yet — build and validate the model (check_mapbayr_model()) and run mapbayest() first, since outcomes depend on posterior ETAs
- Population-level (typical-value) simulation without individual TDM data is intended — posterior ETAs are not identifiable without observations
- Non-PK parameter estimation tasks (e.g. population fitting with nlmixr/saemix or NONMEM model development), which this mapbayr post-estimation chain does not address

## Inputs

- A mapbayests S3 object returned by mapbayest(model, data)
- The original mrgsolve population PK model (with $PARAM, $OMEGA, $SIGMA, $CMT blocks)
- NM-TRAN-format data set with concentrations (or adm_rows()/obs_rows()/add_covariates() records)
- User-defined simulation scenario(s) (e.g. candidate doses as a tibble with amt, rate, time)

## Outputs

- A posteriori ETA values (get_eta())
- Derived individual parameters such as CL (get_param())
- A use_estimates()-updated mrgsolve model reflecting the individual's posterior parameters
- Simulated concentration–time profiles (via mrgsim, e.g. IPRED/DV columns) for dose proposals or trough prediction
- PK outcomes such as AUC computed from (oral) CL and dose

## How to apply

After running mapbayest() on the mrgsolve model and NM-TRAN-format data, extract a posteriori ETAs with get_eta() and derived individual parameters with get_param() (e.g. get_param(my_est, "CL")); an AUC can then be derived from (oral) CL and dose. To simulate concentrations at any time, pipe the estimation object into use_estimates(), which updates the mrgsolve model with the estimated ETA values and covariates, then run mrgsim over user-defined dosing scenarios. In the MIPD/Shiny context this chain (mapbayest → get_eta/get_param → use_estimates → mrgsim) yields the reported dose proposal or trough concentration. Rationale: mapbayr deliberately returns a single 'mapbayests' object exposing etas and tables so user-coded functions can compute these outcomes, enabling standalone Shiny apps without external software.

## Related tools

- **mapbayr** (Provides mapbayest() estimation objects plus get_eta(), get_param() and use_estimates() to extract posterior parameters and update the model) — https://github.com/FelicienLL/mapbayr
- **mrgsolve** (Simulation engine used to predict individual concentrations at arbitrary times after the model is updated with use_estimates()) — https://github.com/metrumresearchgroup/mrgsolve
- **Shiny** (Interface layer in which the derived PK outcomes (dose proposal, trough concentration) are rendered for MIPD)

## Examples

```
updated_model <- my_est %>% use_estimates(); get_eta(my_est); get_param(my_est, "CL")
```

## Evaluation signals

- get_eta() values from the app-based run must match a headless mapbayest() run on the packaged example (exmodel/exdata)
- Derived CL from get_param(my_est, "CL") must equal CL computed in the estimation table (TVCL * exp(ETA1) with covariate scaling)
- IPRED concentrations from the use_estimates()-updated model at observation times must reproduce the estimation output table values
- Reported dose proposal or trough concentration must be consistent with the simulated concentration–time profile for the candidate dosing scenario
- For the article's real models, PK outcomes should match NONMEM with concordance close to 100% (98% identical estimations for test models)

## Limitations

- MAP-BE performance can degrade versus NONMEM with large IIV variances (ωk2 > 1) on several estimable parameters, affecting downstream outcomes
- Discordances vs NONMEM may occur when dose-related parameters (lag time, infusion duration) are estimated, though OFVi exploration suggested mapbayr estimates were often more likely
- The Bayesian estimation design must have been previously validated (residual error, sampling strategy) before outcomes are trusted for dosing
- mapbayr-shiny applications are illustrative only and distributed without any warranty; they should not be used directly to individualize patient doses

## Evidence
