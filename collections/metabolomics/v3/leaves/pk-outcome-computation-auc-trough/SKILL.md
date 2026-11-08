---
name: pk-outcome-computation-auc-trough
description: Use when individual PK parameters have been estimated (e.g. via mapbayest()
  or NONMEM MAP-BE) and the goal is to derive clinically relevant MIPD outcomes such
  as AUCτ,SS, trough concentrations, or time-below-threshold metrics, or to compare
  two estimation tools on those outcomes.
when_to_use_negative:
- When the goal is population-level parameter estimation or model building rather
  than individual MAP-BE outcomes — use population PK tools like nlmixr or saemix
  instead.
- When no individual concentrations/ETAs are available (e.g. only prior population
  predictions without observations) — outcomes would not reflect a posteriori individualization.
- 'When assessing estimation uncertainty: the current mapbayr version provides no
  uncertainty quantification (no Fisher information matrix or posterior sampling).'
edam_operation: http://edamontology.org/operation_2426
edam_topics:
- http://edamontology.org/topic_3390
- http://edamontology.org/topic_0202
tools:
- name: mapbayr
  role: Performs the MAP-Bayesian estimation whose ETAs and parameters (CL) feed the
    outcome computation; get_eta() and get_param() extract the values
  repo: https://github.com/FelicienLL/mapbayr
- name: mrgsolve
  role: Simulates individual concentrations at arbitrary times (e.g. troughs, time-below-threshold
    profiles) from the updated model object
  repo: https://github.com/metrumresearchgroup/mrgsolve
- name: NONMEM
  role: Reference MAP-BE engine whose PK outcomes serve as the control arm for concordance
    classification
- name: dplyr
  role: Data manipulation when aggregating per-drug concordance rates into concordance_summary.csv
- name: ggplot2
  role: Visualization of simulated outcome scenarios (e.g. concentration-time lines
    by dose scenario)
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1002/psp4.12689
    title: Easy and reliable maximum <i>a posteriori</i> Bayesian estimation of pharmacokinetic
      parameters with the open‐source R package mapbayr
  source_content_hash: sha256:9714ebc5ebd3760bafa065d7aa1ea66081a3116a109099f20c73ebc4d503bf2c
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
  edam_operation: http://edamontology.org/operation_2426
  edam_topics:
  - http://edamontology.org/topic_3390
  - http://edamontology.org/topic_0202
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# pk-outcome-computation-auc-trough

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Compute model-informed precision dosing (MIPD) PK outcomes — e.g. AUCτ,SS derived from oral CL and dose, trough concentrations, and time below a toxicity threshold (0.2 µM for methotrexate) — from MAP-Bayesian estimated individual parameters (ETAs, CL) to compare estimation tools like mapbayr vs NONMEM.

## When to use

Use when individual PK parameters have been estimated (e.g. via mapbayest() or NONMEM MAP-BE) and the goal is to derive clinically relevant MIPD outcomes such as AUCτ,SS, trough concentrations, or time-below-threshold metrics, or to compare two estimation tools on those outcomes.

## When NOT to use

- When the goal is population-level parameter estimation or model building rather than individual MAP-BE outcomes — use population PK tools like nlmixr or saemix instead.
- When no individual concentrations/ETAs are available (e.g. only prior population predictions without observations) — outcomes would not reflect a posteriori individualization.
- When assessing estimation uncertainty: the current mapbayr version provides no uncertainty quantification (no Fisher information matrix or posterior sampling).

## Inputs

- mapbayests S3 object returned by mapbayest() (model, NM-TRAN data, ETAs, tables)
- Posterior ETAs (via get_eta()) and estimated parameters such as oral CL (via get_param())
- Updated mrgsolve model object (use_estimates()) with dosing/simulation scenarios (amt, rate, time)
- NONMEM MAP-BE outputs for the same profiles (for cross-tool concordance comparison)

## Outputs

- Per-individual PK outcomes: AUCτ,SS (dose/CL), trough concentrations, time below 0.2 µM for methotrexate
- Per-individual concordance labels (concordant/discordant) between mapbayr and NONMEM
- concordance_summary.csv aggregating per-drug excellent-estimation rates (target: 99.3% on PK outcomes)

## How to apply

After running mapbayest(), extract a posteriori ETAs with get_eta() and derived parameters with get_param(my_est, 'CL'), then compute outcomes: AUCτ,SS is derived from (oral) CL and dose (AUC = dose/CL at steady state), while individual concentrations at arbitrary times (e.g. troughs, or cumulative time below 0.2 µM for methotrexate) are predicted using the mrgsolve framework after updating the model object with use_estimates(). For a validation comparison, apply the identical outcome computation to NONMEM MAP-BE results for the same 1000-individual simulated profiles per drug and classify each estimation as concordant or discordant (e.g. >10% relative error flags discordance, as seen for ibrutinib AUCτ,SS where only 5/1000 individuals exceeded it). Aggregate per-drug concordance into a concordance_summary.csv; near-100% concordance on PK outcomes (published: 99.3% excellent estimations) validates the computation, with residual discordances expected to concentrate on dose-related parameters (lag time, infusion duration) and models with large IIV (ω² > 1).

## Related tools

- **mapbayr** (Performs the MAP-Bayesian estimation whose ETAs and parameters (CL) feed the outcome computation; get_eta() and get_param() extract the values) — https://github.com/FelicienLL/mapbayr
- **mrgsolve** (Simulates individual concentrations at arbitrary times (e.g. troughs, time-below-threshold profiles) from the updated model object) — https://github.com/metrumresearchgroup/mrgsolve
- **NONMEM** (Reference MAP-BE engine whose PK outcomes serve as the control arm for concordance classification)
- **dplyr** (Data manipulation when aggregating per-drug concordance rates into concordance_summary.csv)
- **ggplot2** (Visualization of simulated outcome scenarios (e.g. concentration-time lines by dose scenario))

## Examples

```
my_est %>% use_estimates() %>% mrgsim(output = "df") ; get_param(my_est, "CL") ; AUC_tau_ss <- dose / CL
```

## Evaluation signals

- Excellent-estimation rate on PK outcomes approaches the published 99.3% (97.3% on PK parameters) across the seven real published models.
- Per-drug discordances are rare and concentrated on dose-related parameters (lag time, infusion duration) and models with large IIV (ω² > 1).
- For specific drugs, expected error magnitudes are reproduced (e.g. only 5/1000 individuals with >10% error on ibrutinib AUCτ,SS vs NONMEM).
- AUCτ,SS recomputed as dose/CL matches values implied by the estimated oral clearance for each individual.
- concordance_summary.csv is produced with per-drug concordance rates for all seven drugs and n=1000 profiles each.

## Limitations

- MAP estimation cannot inform the ability to return the true individual parameter value (ηi); concordance between tools does not prove accuracy.
- Discrepancies occur when dose-related parameters (lag time, infusion duration) are estimated, or with large IIV variances (ωk² > 1).
- No proper comparison of run times between mapbayr and NONMEM was performed (mapbayr is slower).
- No estimation-uncertainty quantification in the current mapbayr version; future versions could add Fisher information matrix or posterior sampling.

## Evidence

- [results] mapbayr achieved 97.3% and 99.3% excellent estimations for PK parameters and PK outcomes: [evidence span withheld — anchor status: not_found]
