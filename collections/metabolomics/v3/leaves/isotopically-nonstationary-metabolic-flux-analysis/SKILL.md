---
name: isotopically-nonstationary-metabolic-flux-analysis
description: Use when time-course stable-isotope labeling data (e.g., [13C5,15N2]
  glutamate 0-8 h sampling) have been processed into natural-abundance-corrected isotopologue
  distributions and fluxes with confidence intervals are needed.
when_to_use_negative:
- Only steady-state (stationary) labeling endpoint data are available with no time-course
  sampling — classic 13C MFA, not INST-MFA, applies
- Data come from nominal-resolution mass spectrometry where dual-labeled isotopologues
  produce nominally identical mass shifts and cannot be resolved — a dual-isotope
  model is not identifiable
- Isotopologue distributions have not been post-processed (generated and natural abundance
  corrected) by SIMPEL from the HRMS dataset
edam_operation: http://edamontology.org/operation_3660
edam_topics:
- http://edamontology.org/topic_3292
tools:
- name: INCA
  role: MATLAB-based platform used to execute the INST-MFA least-squares parameter
    estimation against the SIMPEL-exported isotopologue distributions and extract
    fluxes with 95% CIs
- name: SIMPEL
  role: R package that generates isotopologue distributions and performs natural abundance
    correction on HRMS data, producing the data tables used as INST-MFA input
  repo: github.com/SIMPELmetabolism/SIMPEL
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s42003-024-05844-z
    title: 'SIMPEL: using stable isotopes to elucidate dynamics of context specific
      metabolism'
  source_content_hash: sha256:6a178f3d9030f004ab90dae1bddc002d2f160945fdae451add20c1c919799845
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - INCA
  - SIMPEL
  edam_operation: http://edamontology.org/operation_3660
  edam_topics:
  - http://edamontology.org/topic_3292
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# isotopically-nonstationary-metabolic-flux-analysis

## Summary

Isotopically non-stationary metabolic flux analysis (INST-MFA) fits simulated isotopologue labeling trajectories to time-course HRMS data to estimate reaction fluxes with 95% confidence intervals, performed here with the MATLAB-based INCA tool on data tables exported from SIMPEL.

## When to use

Use when time-course stable-isotope labeling data (e.g., [13C5,15N2] glutamate 0-8 h sampling) have been processed into natural-abundance-corrected isotopologue distributions and fluxes with confidence intervals are needed.

## When NOT to use

- Only steady-state (stationary) labeling endpoint data are available with no time-course sampling — classic 13C MFA, not INST-MFA, applies
- Data come from nominal-resolution mass spectrometry where dual-labeled isotopologues produce nominally identical mass shifts and cannot be resolved — a dual-isotope model is not identifiable
- Isotopologue distributions have not been post-processed (generated and natural abundance corrected) by SIMPEL from the HRMS dataset

## Inputs

- Natural-abundance-corrected isotopologue distribution table exported from SIMPEL (from XCMS-pre-processed HRMS data of a time-course isotope labeling experiment)
- INCA model file encoding the stoichiometric network and labeling design (single-label, parallel-label, or dual-isotope-label configuration)

## Outputs

- Fitted reaction flux estimates in nmol g-1 h-1
- 95% confidence intervals for each flux estimate
- Comparison table of fluxes across single, parallel, and dual labeling models (e.g., flux_estimates_fig2c.csv)

## How to apply

Load the natural-abundance-corrected isotopologue distribution table exported from SIMPEL (e.g., from the Arabidopsis root [13C5,15N2]glutamine labeling time course sampled 0-8 h) into the MATLAB-based INCA environment. Load the INCA model file encoding the network and labeling design — the article provides three configurations: single-label, parallel 13C/15N, and dual 13C15N models (Supplementary Data 2, 3 and 4). Run INCA's INST-MFA least-squares fit of simulated labeling trajectories against the measured isotopologue distributions across the time course. Extract fitted reaction fluxes (reported in nmol g-1 h-1) and their 95% confidence intervals from the converged parameter estimation. Compare designs: dual-isotope labeling resolved fluxes that traditionally require multiple labeling experiments, reducing confidence intervals by 71% (parallel) and 84% (dual) versus single 13C MFA. Validate by checking that reproduced flux values and CIs match the published figures (e.g., Fig. 2c).

## Related tools

- **INCA** (MATLAB-based platform used to execute the INST-MFA least-squares parameter estimation against the SIMPEL-exported isotopologue distributions and extract fluxes with 95% CIs)
- **SIMPEL** (R package that generates isotopologue distributions and performs natural abundance correction on HRMS data, producing the data tables used as INST-MFA input) — github.com/SIMPELmetabolism/SIMPEL

## Evaluation signals

- Reproduced flux values and 95% confidence intervals match those reported in Fig. 2c of the paper for the single, parallel, and dual labeling models
- Confidence intervals of the dual-isotope model are narrower than the single 13C model (expected reductions: ~71% for parallel, ~84% for dual)
- INCA parameter estimation converges to a least-squares fit of simulated labeling trajectories against measured isotopologue distributions at every time point (0-8 h)
- Flux values are reported in physically interpretable units (nmol g-1 h-1) and are consistent across the three model configurations where reactions overlap

## Limitations

- No software versions reported for INCA (or SIMPEL/XCMS) in the results section, complicating exact reproduction
- Combining multiple heavy isotopes is limited with nominal m/z resolution because dual-labeled tracers produce nominally identical mass shifts; HRMS capable of resolving m/z values differing by <0.01 Da is required
- The number of multivariate isotopologues increases exponentially with the number of labeled atoms (I = ∏(Ne+1)), which can inflate model complexity

## Evidence

- [intro] Data tables exported from SIMPEL were used for INST-MFA with INCA.: "Data tables exported from SIMPEL were used for INST-MFA with INCA."
- [intro] dual-isotope INST-MFA with SIMPEL resolves fluxes: "a dual-isotope labeling experiment processed with SIMPEL was used for isotopically nonstationary metabolic flux analysis, to resolve challenging fluxes that traditionally require data from multiple"
- [results] Arabidopsis root glutamine time-course labeling: "ten-day-old Arabidopsis roots were grown on a predefined carbon and nitrogen rich medium68 supplemented with [13C515N2] glutamine (Fig. 1a) and sampled over time (0–8 h)"
- [results] INST-MFA performed with INCA on SIMPEL isotopologue distributions: "used to perform isotopically non-stationary metabolic flux analysis (INST-MFA) with INCA69"
- [results] three labeling datasets derivable from one dual-isotope experiment: "all three datasets could be derived from a single 13C15N-glutamine labeling experiment by summing isotopologue abundances contained in the comprehensive HRMS dataset"
