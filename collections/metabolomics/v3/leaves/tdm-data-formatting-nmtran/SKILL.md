---
name: tdm-data-formatting-nmtran
description: Use when individual concentration records and dosing history (times,
  amounts, infusion rates, covariates) must be assembled into an NM-TRAN dataset to
  feed mapbayest() from a mrgsolve-coded population PK model.
when_to_use_negative:
- You already have a complete, correctly coded NM-TRAN dataset with evid/cmt/amt/mdv
  columns — feed it directly as the data argument of mapbayest()
- The task is population (NLME) model fitting rather than individual MAP-BE — nlmixr
  or saemix are the dedicated packages cited for population PK/PD modeling
- The model is not coded in mrgsolve or fails check_mapbayr_model() validation — mapbayr
  requires mrgsolve-formatted models
edam_operation: http://edamontology.org/operation_2409
tools:
- name: mapbayr
  role: Provides adm_rows(), obs_rows(), and add_covariates() pipe-friendly helpers
    that build the NM-TRAN dataset, and mapbayest() which consumes it for MAP-BE
  repo: https://github.com/FelicienLL/mapbayr
- name: mrgsolve
  role: Hosts the user-supplied population PK model into which dosing/observation
    rows are piped before estimation
  repo: https://github.com/metrumresearchgroup/mrgsolve
- name: Shiny
  role: Wraps the adm_rows/obs_rows/add_covariates formatting calls in reactive UI/server
    code for standalone MIPD web applications
  repo: https://github.com/FelicienLL/mapbayr-shiny
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1002/psp4.12689
    title: Easy and reliable maximum <i>a posteriori</i> Bayesian estimation of pharmacokinetic
      parameters with the open‐source R package mapbayr
  source_content_hash: sha256:f504d42cc78fb3933d97933a7967835893eba24f4270f7b293bacce0eca4adde
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
  edam_operation: http://edamontology.org/operation_2409
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# tdm-data-formatting-nmtran

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Format therapeutic drug monitoring dosing and observation records into an NM-TRAN-style dataset suitable for MAP-Bayesian estimation with mapbayr, using the pipe-friendly adm_rows(), obs_rows(), and add_covariates() helpers instead of hand-building the data frame.

## When to use

Use when individual concentration records and dosing history (times, amounts, infusion rates, covariates) must be assembled into an NM-TRAN dataset to feed mapbayest() from a mrgsolve-coded population PK model.

## When NOT to use

- You already have a complete, correctly coded NM-TRAN dataset with evid/cmt/amt/mdv columns — feed it directly as the data argument of mapbayest()
- The task is population (NLME) model fitting rather than individual MAP-BE — nlmixr or saemix are the dedicated packages cited for population PK/PD modeling
- The model is not coded in mrgsolve or fails check_mapbayr_model() validation — mapbayr requires mrgsolve-formatted models

## Inputs

- mrgsolve population PK model object (coded with annotated $PARAM, $OMEGA, $SIGMA, $CMT blocks, validated by check_mapbayr_model())
- Dosing records: times, amounts, infusion rates, route/compartment
- Observed concentration records: time points and DV values (optionally mdv flags)
- Patient covariates (e.g. body weight) used in the model $PARAM @covariates block

## Outputs

- NM-TRAN-formatted data frame (columns such as ID, time, evid, cmt, amt, rate, DV, mdv, covariates) suitable for mapbayest()
- A mapbayests S3 object returned by mapbayest(), containing input model/data, a posteriori ETAs, and output tables (IPRED, PRED, CL)

## How to apply

Start from a user-supplied population PK model coded in mrgsolve (via mcode/mread with annotated $PARAM, $OMEGA, $SIGMA, and $CMT blocks) and validate it with check_mapbayr_model(). Rather than manually building an NM-TRAN data frame with columns like ID, time, evid, cmt, amt, rate, DV, mdv, chain adm_rows() calls for each dosing event (with time, amt, and rate for infusions), obs_rows() calls for each concentration record (with DV and optionally mdv=1 for non-observation records), and add_covariates() for patient covariates such as body weight. Pipe the model through these helpers directly into mapbayest(), which uses L-BFGS-B optimization for MAP-BE. The rationale is that constructing NM-TRAN format by hand is error-prone; the helpers automatically set evid/cmt/amt/mdv bookkeeping. In a Shiny MIPD app, these formatting calls are wrapped in reactive expressions driven by user inputs.

## Related tools

- **mapbayr** (Provides adm_rows(), obs_rows(), and add_covariates() pipe-friendly helpers that build the NM-TRAN dataset, and mapbayest() which consumes it for MAP-BE) — https://github.com/FelicienLL/mapbayr
- **mrgsolve** (Hosts the user-supplied population PK model into which dosing/observation rows are piped before estimation) — https://github.com/metrumresearchgroup/mrgsolve
- **Shiny** (Wraps the adm_rows/obs_rows/add_covariates formatting calls in reactive UI/server code for standalone MIPD web applications) — https://github.com/FelicienLL/mapbayr-shiny

## Examples

```
my_est <- my_model %>% adm_rows(time = 0, amt = 100, rate = 20) %>% obs_rows(time = 6, DV = 3.9) %>% obs_rows(time = 15, DV = 1.1) %>% add_covariates(BW = 90) %>% mapbayest()
```

## Evaluation signals

- The formatted data frame matches the manual NM-TRAN layout in the README example (ID=1, evid=1 only at dose rows, mdv=1 on dose and non-used records, NA in DV at dose rows)
- The headless mapbayest() run on the formatted data reproduces the estimates obtained via the alternative data-frame input (identical ETAs and derived CL)
- All observation rows carry non-missing DV values and mdv=0; dose rows carry amt/rate with mdv=1 and DV=NA
- check_mapbayr_model() passes on the model before rows are added
- print(my_est) reports the expected counts: '1 individual(s)', the number of observation(s) matching obs_rows() calls, and the number of ETAs matching the $OMEGA dimension

## Limitations

- mapbayr is slower than NONMEM for MAP-BE (no formal run-time comparison was performed, but NONMEM is acknowledged as faster)
- Formatting must respect the residual error model encoded in $SIGMA/$TABLE (proportional, additive, mixed, or exponential); mis-specified mdv/evid records will silently bias the estimation
- Infusion rates must be supplied explicitly to adm_rows() when the model expects rate-based input (e.g. rate = amt/5 in simulation scenarios)

## Evidence

- [other] Build the Shiny UI/server with reactive inputs that format TDM dosing and observation records into NM-TRAN-style data using the pipe-friendly adm_rows() and obs_rows() helpers plus add_covariates().: [evidence span withheld — anchor status: not_found]
- [other] Load the user-supplied population PK model coded in mrgsolve and validate it with check_mapbayr_model().: [evidence span withheld — anchor status: not_found]
- [methods] Call mapbayest() on the formatted data and model to perform MAP-BE with the default L-BFGS-B optimization.: [evidence span withheld — anchor status: not_found]
