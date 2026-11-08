---
name: shiny-web-application-development
description: 'Use when building a browser-based dosing/TDM tool: the user needs clinicians
  without R expertise to run MAP-BE of PK parameters (mapbayest) from a mrgsolve population
  PK model and TDM observations, and derive PK outcomes like AUC or proposed doses.'
when_to_use_negative:
- The goal is population PK/PD model development or fitting — nlmixr or saemix are
  the article's referenced tools for population modeling, not mapbayr-based Shiny
  apps
- The Bayesian estimation design (residual error model, sampling strategy) has not
  been previously validated, e.g. versus NONMEM on simulated data with the expected
  dosing/sampling strategy
- The app would be used to directly individualize patient doses in clinical care —
  the mapbayr-shiny examples are 'for illustrative purpose only' and distributed without
  any warranty, with unresolved medical-device regulatory aspects
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3390
- http://edamontology.org/topic_2269
tools:
- name: mapbayr
  role: 'Estimation engine: single mapbayest() function performs MAP-BE from model
    + data; helpers adm_rows/obs_rows/add_covariates format data; get_eta/get_param/use_estimates
    extract a posteriori outcomes'
  repo: https://github.com/FelicienLL/mapbayr
- name: Shiny
  role: Web framework used to code the UI/server that executes the R estimation and
    simulation tasks in the background
- name: mrgsolve
  role: Implements and solves the user-supplied population PK model (ODEs); use_estimates()
    updates the model object so mrgsim can simulate individual concentrations at any
    time
  repo: https://github.com/metrumresearchgroup/mrgsolve
- name: R
  role: 'Only runtime requirement: the app runs locally with R, shiny, mrgsolve and
    mapbayr installed (e.g. via devtools::install_github(''FelicienLL/mapbayr''))'
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1002/psp4.12689
    title: Easy and reliable maximum <i>a posteriori</i> Bayesian estimation of pharmacokinetic
      parameters with the open‐source R package mapbayr
  source_content_hash: sha256:eabfa856067fd6aae1c99a3d1a726e26b904d7ff75d7e89843e18b48db9a58fd
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
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3390
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# shiny-web-application-development

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Wrap the mapbayr MAP-Bayesian estimation engine into a standalone Shiny web application for model-based Therapeutic Drug Monitoring (MIPD), requiring only an R installation and no external nonlinear mixed-effect software.

## When to use

Use when building a browser-based dosing/TDM tool: the user needs clinicians without R expertise to run MAP-BE of PK parameters (mapbayest) from a mrgsolve population PK model and TDM observations, and derive PK outcomes like AUC or proposed doses.

## When NOT to use

- The goal is population PK/PD model development or fitting — nlmixr or saemix are the article's referenced tools for population modeling, not mapbayr-based Shiny apps
- The Bayesian estimation design (residual error model, sampling strategy) has not been previously validated, e.g. versus NONMEM on simulated data with the expected dosing/sampling strategy
- The app would be used to directly individualize patient doses in clinical care — the mapbayr-shiny examples are 'for illustrative purpose only' and distributed without any warranty, with unresolved medical-device regulatory aspects

## Inputs

- User-supplied population PK model coded in mrgsolve (mcode/mread with $PARAM, $OMEGA, $SIGMA, $CMT, $TABLE, $MAIN, $ODE blocks)
- TDM dosing and observation records (times, amounts, rates, DV concentrations, mdv flags)
- Patient covariates (e.g. BW in kg)

## Outputs

- A posteriori ETA estimates (get_eta) and individual parameters such as CL (get_param) rendered in a Shiny table
- Individual-predicted (IPRED) vs population-predicted (PRED) concentration tables and plots
- Simulated concentration profiles for dose scenarios via use_estimates() + mrgsim (e.g. proposed dose or trough concentration)

## How to apply

Code the population PK model in mrgsolve with annotated $PARAM (including @covariates), $OMEGA, $SIGMA and $CMT blocks and validate it with check_mapbayr_model(). Build the Shiny UI/server with reactive inputs that convert dosing and observation records into NM-TRAN-style data using the pipe-friendly adm_rows(), obs_rows() and add_covariates() helpers, then call the single estimation function mapbayest() (default L-BFGS-B optimization). Extract a posteriori ETAs with get_eta(), individual parameters such as CL with get_param(), and optionally update the model with use_estimates() followed by mrgsim to simulate concentrations at arbitrary times for dose proposals. Render the estimation table and derived PK outcomes in the interface. Validate by confirming the app reproduces the ETAs and derived CL from a headless mapbayest() run on the packaged example (exmodel/exdata); ready-made illustrative apps (e.g. '901-carboplatin') can be launched with shiny::runGitHub from the FelicienLL/mapbayr-shiny repository.

## Related tools

- **mapbayr** (Estimation engine: single mapbayest() function performs MAP-BE from model + data; helpers adm_rows/obs_rows/add_covariates format data; get_eta/get_param/use_estimates extract a posteriori outcomes) — https://github.com/FelicienLL/mapbayr
- **Shiny** (Web framework used to code the UI/server that executes the R estimation and simulation tasks in the background)
- **mrgsolve** (Implements and solves the user-supplied population PK model (ODEs); use_estimates() updates the model object so mrgsim can simulate individual concentrations at any time) — https://github.com/metrumresearchgroup/mrgsolve
- **R** (Only runtime requirement: the app runs locally with R, shiny, mrgsolve and mapbayr installed (e.g. via devtools::install_github('FelicienLL/mapbayr')))

## Examples

```
shiny::runGitHub("mapbayr-shiny", "FelicienLL", subdir = "901-carboplatin", ref = "main")
```

## Evaluation signals

- App-reported ETAs and derived CL must match a headless mapbayest() run on the packaged example (exmodel/exdata)
- check_mapbayr_model() passes on the mrgsolve model with annotated $PARAM/$OMEGA/$SIGMA/$CMT blocks
- Formatted data from adm_rows()/obs_rows()/add_covariates() is valid NM-TRAN format (correct evid, cmt, amt, rate, mdv columns)
- Simulated profiles after use_estimates() reflect individual ETAs and covariates, not population values (e.g. CL ≈ get_param result, here ~1.79 L/h in the README example)
- Publication-level benchmark: 98% of mapbayr estimations identical to NONMEM on test models and close to 100% concordance on PK outcomes for seven real published models

## Limitations

- Shiny applications cannot be hosted on GitHub directly — a server is required; users must run them locally via runGitHub or clone the repository
- mapbayr is slower than NONMEM and discrepancies can occur when dose-related parameters (lag time, infusion duration) are estimated or with large IIV variances (ωk2 > 1)
- The example apps 'should not be used directly to individualize doses of patients' and are distributed without any warranty; regulatory registration as medical devices in some countries remains unresolved
- No estimation-uncertainty quantification in the current version (future versions could use the Fisher information matrix or posterior sampling)

## Evidence

- [abstract] mapbayr performs estimation with a single two-argument function: [evidence span withheld — anchor status: not_found]
- [readme] shiny::runGitHub launch example: [evidence span withheld — anchor status: external_doc]
- [readme] app warnings and disclaimers: [evidence span withheld — anchor status: external_doc]
