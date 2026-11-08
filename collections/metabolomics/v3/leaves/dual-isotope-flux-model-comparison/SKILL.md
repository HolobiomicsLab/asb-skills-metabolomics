---
name: dual-isotope-flux-model-comparison
description: Use when HRMS isotopologue distributions from a dual-isotope tracer (e.g.,
  [13C5,15N2]glutamine time course) are available and you must decide whether dual-isotope
  INST-MFA reduces flux uncertainty versus single- or parallel-label designs.
when_to_use_negative:
- Only nominal-resolution (low m/z) MS data are available, since dual-labeled tracers
  produce nominally identical mass shifts that cannot be resolved
- The system is at isotopic steady state, where INST-MFA (and its time-course labeling
  data requirement) is unnecessary compared to steady-state MFA
- Only a single-isotope tracer experiment exists and no 15N (or second-element) labeling
  information can be summed from the HRMS data
edam_operation: http://edamontology.org/operation_3665
edam_topics:
- http://edamontology.org/topic_3292
- http://edamontology.org/topic_0203
tools:
- name: INCA
  role: MATLAB-based environment for performing isotopically non-stationary metabolic
    flux analysis (INST-MFA) least-squares fitting and extracting fluxes with 95%
    confidence intervals from each model configuration
- name: SIMPEL
  role: R package that generates isotopologue distributions and natural abundance
    corrections from HRMS data and exports the data tables used as INST-MFA inputs
  repo: github.com/SIMPELmetabolism/SIMPEL
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s42003-024-05844-z
    title: 'SIMPEL: using stable isotopes to elucidate dynamics of context specific
      metabolism'
  source_content_hash: sha256:a300a4699aa17fc401c6480e877411d6fda831fd2b54fe4dddbd7210e9378f11
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - INCA
  - SIMPEL
  edam_operation: http://edamontology.org/operation_3665
  edam_topics:
  - http://edamontology.org/topic_3292
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# dual-isotope-flux-model-comparison

## Summary

Fits single-13C, parallel-13C/15N, and dual-13C15N INST-MFA model configurations in INCA against SIMPEL-exported isotopologue distributions to quantify how dual-isotope labeling narrows flux confidence intervals.

## When to use

Use when HRMS isotopologue distributions from a dual-isotope tracer (e.g., [13C5,15N2]glutamine time course) are available and you must decide whether dual-isotope INST-MFA reduces flux uncertainty versus single- or parallel-label designs.

## When NOT to use

- Only nominal-resolution (low m/z) MS data are available, since dual-labeled tracers produce nominally identical mass shifts that cannot be resolved
- The system is at isotopic steady state, where INST-MFA (and its time-course labeling data requirement) is unnecessary compared to steady-state MFA
- Only a single-isotope tracer experiment exists and no 15N (or second-element) labeling information can be summed from the HRMS data

## Inputs

- SIMPEL-exported isotopologue distribution table for the Arabidopsis root [13C5,15N2]glutamine labeling time course (0–8 h sampling)
- Three INCA model files encoding single-label, parallel-label, and dual-isotope-label configurations (e.g., Supplementary Data 2, 3, and 4)

## Outputs

- Fitted reaction fluxes (nmol g-1 h-1) with 95% confidence intervals for each of the three model configurations
- Side-by-side flux comparison table (e.g., flux_estimates_fig2c.csv) quantifying CI reduction across labeling designs

## How to apply

Derive the single-13C, 15N, and combined 13C15N datasets from one dual-isotope HRMS labeling experiment by summing isotopologue abundances, since HRMS resolves mass shifts <0.01 Da that nominal-resolution MS cannot. Load the SIMPEL-exported isotopologue distribution table for the Arabidopsis root [13C5,15N2]glutamine 0–8 h time course together with the three INCA model files encoding single-label, parallel-label, and dual-isotope-label configurations. For each configuration, run INCA's INST-MFA least-squares fit of simulated labeling trajectories against the measured isotopologue distributions and extract fitted fluxes (nmol g-1 h-1) with their 95% CIs. Tabulate fluxes side by side and compare interval widths; expect the parallel 13C/15N and dual 13C15N models to narrow CIs by roughly 71% and 84%, respectively, relative to single-13C MFA.

## Related tools

- **INCA** (MATLAB-based environment for performing isotopically non-stationary metabolic flux analysis (INST-MFA) least-squares fitting and extracting fluxes with 95% confidence intervals from each model configuration)
- **SIMPEL** (R package that generates isotopologue distributions and natural abundance corrections from HRMS data and exports the data tables used as INST-MFA inputs) — github.com/SIMPELmetabolism/SIMPEL

## Examples

```
for model in {single,parallel,dual}; do matlab -batch "load('${model}.mat'); inca(model);" && grep flux flux_estimates_fig2c.csv; done
```

## Evaluation signals

- Reproduced flux values and 95% CIs for the single, parallel, and dual labeling models match those reported in Fig. 2c of the paper
- Parallel 13C/15N model shows ~71% CI reduction versus single-13C MFA, and dual 13C15N model shows ~84% CI reduction
- All three INCA parameter estimations converge to a least-squares fit of simulated labeling trajectories against measured isotopologue distributions
- Flux units are consistently reported as nmol g-1 h-1 across all three models in the comparison table

## Limitations

- Deriving single-isotope and dual-isotope datasets from nominal m/z data is not possible because combining multiple heavy isotopes is limited when nominally identical mass shifts are recorded
- No software versions reported for INCA (or SIMPEL) in this section, complicating exact reproduction
- The number of multivariate isotopologues grows exponentially with labeled atoms (I = ∏(Ne+1)), inflating data-table dimensionality for dual-isotope models

## Evidence

- [results] parallel 13C/15N and dual 13C15N models reduced CIs by 71% and 84% versus single 13C MFA: [evidence span withheld — anchor status: not_found]
- [intro] Data tables exported from SIMPEL were used for INST-MFA with INCA.: "Data tables exported from SIMPEL were used for INST-MFA with INCA."
- [results] all three datasets could be derived from a single 13C15N-glutamine labeling experiment: "all three datasets could be derived from a single 13C15N-glutamine labeling experiment by summing isotopologue abundances contained in the comprehensive HRMS dataset"
- [intro] dual-isotope labeling processed with SIMPEL enables INST-MFA to resolve fluxes: "a dual-isotope labeling experiment processed with SIMPEL was used for isotopically nonstationary metabolic flux analysis, to resolve challenging fluxes that traditionally require data from multiple"
