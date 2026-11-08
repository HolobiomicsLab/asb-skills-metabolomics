---
name: labeling-timecourse-flux-estimation
description: Use when isotopologue distribution tables from a stable-isotope labeling
  time course (e.g. SIMPEL-exported [13C5,15N2]glutamine data) must be fitted by INST-MFA
  in INCA to obtain reaction fluxes with 95% CIs.
when_to_use_negative:
- Only nominal-resolution (low m/z resolution) mass spectrometry data are available,
  since dual-labeled tracers produce nominally identical mass shifts that cannot be
  resolved
- The system is at isotopic/metabolic steady state and stationary MFA (not INST-MFA)
  suffices
- The input is raw HRMS data that still needs XCMS pre-processing and SIMPEL post-processing
  (isotopologue distribution generation and natural abundance correction) before flux
  fitting
edam_topics:
- http://edamontology.org/topic_3334
tools:
- name: INCA
  role: MATLAB-based environment in which INST-MFA least-squares fitting of simulated
    labeling trajectories against measured isotopologue distributions is executed,
    yielding fluxes and 95% CIs
- name: SIMPEL
  role: R package that post-processes isotope-enriched HRMS datasets and exports the
    isotopologue data tables used as INST-MFA input
  repo: github.com/SIMPELmetabolism/SIMPEL
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s42003-024-05844-z
    title: 'SIMPEL: using stable isotopes to elucidate dynamics of context specific
      metabolism'
  source_content_hash: sha256:9f0eb92f704fc1b89c4daab81854387563b9030fe25be75c5d764d84d11eb2f9
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - INCA
  - SIMPEL
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3334
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# labeling-timecourse-flux-estimation

## Summary

Estimate metabolic fluxes and 95% confidence intervals from isotope-labeling time courses by fitting isotopologue distributions exported from SIMPEL using isotopically non-stationary metabolic flux analysis (INST-MFA) in the MATLAB-based INCA environment.

## When to use

Use when isotopologue distribution tables from a stable-isotope labeling time course (e.g. SIMPEL-exported [13C5,15N2]glutamine data) must be fitted by INST-MFA in INCA to obtain reaction fluxes with 95% CIs.

## When NOT to use

- Only nominal-resolution (low m/z resolution) mass spectrometry data are available, since dual-labeled tracers produce nominally identical mass shifts that cannot be resolved
- The system is at isotopic/metabolic steady state and stationary MFA (not INST-MFA) suffices
- The input is raw HRMS data that still needs XCMS pre-processing and SIMPEL post-processing (isotopologue distribution generation and natural abundance correction) before flux fitting

## Inputs

- SIMPEL-exported isotopologue distribution table for a labeling time course (e.g. Arabidopsis root [13C5,15N2]glutamine 0–8 h sampling)
- INCA model files encoding single-label, parallel-label, and dual-isotope-label configurations (e.g. Supplementary Data 2, 3, and 4)
- MATLAB-based INCA environment

## Outputs

- Fitted reaction flux values in nmol g-1 h-1 for each model configuration (single, parallel, dual labeling)
- 95% confidence intervals on each flux estimate
- Comparison table (e.g. flux_estimates_fig2c.csv) of single vs. parallel vs. dual labeling fluxes and CIs

## How to apply

Load the SIMPEL-exported isotopologue distribution table (e.g. from the Arabidopsis root [13C5,15N2]glutamine 0–8 h time course) together with the INCA model files encoding each labeling configuration (single 13C, parallel 13C/15N, dual 13C15N). For each configuration, execute INCA's INST-MFA least-squares fit of simulated labeling trajectories against the measured isotopologue distributions and extract fitted reaction fluxes with 95% confidence intervals upon convergence. Tabulate fluxes in nmol g-1 h-1 for the single, parallel, and dual labeling models side by side. The rationale is that HRMS resolves dual-labeled isotopologues that nominal-resolution MS cannot, so a single dual-isotope experiment can supply the information that traditionally requires multiple labeling experiments; in the source study, parallel and dual models reduced CIs by 71% and 84% versus single 13C MFA.

## Related tools

- **INCA** (MATLAB-based environment in which INST-MFA least-squares fitting of simulated labeling trajectories against measured isotopologue distributions is executed, yielding fluxes and 95% CIs)
- **SIMPEL** (R package that post-processes isotope-enriched HRMS datasets and exports the isotopologue data tables used as INST-MFA input) — github.com/SIMPELmetabolism/SIMPEL

## Evaluation signals

- Reproduced flux values and 95% CIs match those reported for the single, parallel, and dual labeling models in Fig. 2c of the paper
- Confidence intervals from the parallel 13C/15N and dual 13C15N models are narrower than single 13C MFA (reported reductions of 71% and 84%, respectively)
- Each INCA parameter estimation converges to a least-squares fit of simulated labeling trajectories against the measured isotopologue distributions
- Flux values are tabulated in the expected units of nmol g-1 h-1 for all three model configurations
- All three labeling datasets (13C, 15N, 13C15N) are derivable by summing isotopologue abundances from the single [13C5,15N2]glutamine HRMS dataset

## Limitations

- No software versions reported for SIMPEL, XCMS, MZmine2, IsoCorrectoR, or INCA
- Resolving additional fluxes from multiple heavy isotopes is limited when nominal m/z values are recorded, since dual-labeled tracers produce nominally identical mass shifts
- No repository or accession identifiers provided in the results section for the model files

## Evidence

- [intro] SIMPEL-to-INCA export is the documented handoff: "Data tables exported from SIMPEL were used for INST-MFA with INCA."
- [results] INST-MFA executed in INCA on SIMPEL outputs: "used to perform isotopically non-stationary metabolic flux analysis (INST-MFA) with INCA"
- [intro] Dual-isotope experiment replaces multiple labeling experiments: "a dual-isotope labeling experiment processed with SIMPEL was used for isotopically nonstationary metabolic flux analysis, to resolve challenging fluxes"
- [results] All three labeling datasets derive from one dual-isotope experiment: "all three datasets could be derived from a single 13C15N-glutamine labeling experiment by summing isotopologue abundances contained in the comprehensive HRMS dataset"
