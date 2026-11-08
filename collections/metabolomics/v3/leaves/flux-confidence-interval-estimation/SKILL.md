---
name: flux-confidence-interval-estimation
description: Use when isotopologue distribution tables from a stable-isotope labeling
  time course (e.g., Arabidopsis roots labeled with [13C5,15N2]glutamine over 0–8
  h) must be fitted by INST-MFA to obtain fluxes with 95% CIs and to compare single-
  vs. dual-isotope tracer designs.
when_to_use_negative:
- Input is raw HRMS spectra or an unprocessed XCMS/MZmine2 feature table — data must
  first be post-processed with SIMPEL into natural-abundance-corrected isotopologue
  distributions
- Data were acquired at nominal mass resolution and cannot resolve dual-labeled isotopologues,
  since combining multiple heavy isotopes is limited when nominal m/z values are recorded
- The system is at isotopic stationary state, where INST-MFA fitting of transient
  labeling trajectories over a 0–8 h time course is inappropriate
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: INCA
  role: MATLAB-based environment used to run INST-MFA least-squares fitting of simulated
    labeling trajectories against measured isotopologue distributions and to extract
    fluxes with 95% confidence intervals
- name: SIMPEL
  role: R package that post-processes isotope-enriched HRMS datasets (isotopologue
    distributions, natural abundance correction) and exports the data tables consumed
    by INST-MFA in INCA
  repo: github.com/SIMPELmetabolism/SIMPEL
- name: MATLAB
  role: Runtime environment in which the INCA INST-MFA models are loaded and executed
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s42003-024-05844-z
    title: 'SIMPEL: using stable isotopes to elucidate dynamics of context specific
      metabolism'
  source_content_hash: sha256:e04cf4d873da1a696050a472e6cb5ccc587e92ad31bd7fb6eeb0f4af2af68a1b
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - INCA
  - MATLAB
  - SIMPEL
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# flux-confidence-interval-estimation

## Summary

Estimate metabolic fluxes and their 95% confidence intervals by isotopically non-stationary metabolic flux analysis (INST-MFA) in INCA, using isotopologue distribution tables exported from SIMPEL, and quantify how parallel 13C/15N or dual 13C15N tracer designs narrow CIs relative to single-13C designs.

## When to use

Use when isotopologue distribution tables from a stable-isotope labeling time course (e.g., Arabidopsis roots labeled with [13C5,15N2]glutamine over 0–8 h) must be fitted by INST-MFA to obtain fluxes with 95% CIs and to compare single- vs. dual-isotope tracer designs.

## When NOT to use

- Input is raw HRMS spectra or an unprocessed XCMS/MZmine2 feature table — data must first be post-processed with SIMPEL into natural-abundance-corrected isotopologue distributions
- Data were acquired at nominal mass resolution and cannot resolve dual-labeled isotopologues, since combining multiple heavy isotopes is limited when nominal m/z values are recorded
- The system is at isotopic stationary state, where INST-MFA fitting of transient labeling trajectories over a 0–8 h time course is inappropriate

## Inputs

- SIMPEL-exported isotopologue distribution table from HRMS data of an Arabidopsis root [13C5,15N2]glutamine labeling time course (0–8 h sampling)
- INCA model files encoding single-label, parallel-label, and dual-isotope-label configurations (e.g., Supplementary Data 2, 3, and 4)
- Time-course sampling metadata (labeling time points)

## Outputs

- Fitted reaction flux values in nmol g-1 h-1 for each model configuration
- 95% confidence intervals for each estimated flux
- Side-by-side comparison table of single, parallel, and dual labeling flux estimates (e.g., flux_estimates_fig2c.csv)

## How to apply

Load the SIMPEL-exported isotopologue distribution table (from HRMS data on the [13C5,15N2]glutamine labeling time course) into the MATLAB-based INCA environment together with each INCA model file encoding the single-label, parallel-label, and dual-isotope-label configurations. For each configuration, run INCA's INST-MFA least-squares fit of simulated labeling trajectories against the measured isotopologue distributions over the 0–8 h time course. Extract the fitted reaction fluxes and their 95% confidence intervals from each converged parameter estimation, tabulate the fluxes in nmol g-1 h-1 side by side, and write the comparison table (e.g., flux_estimates_fig2c.csv). The rationale is that parallel 13C/15N and dual 13C15N models constrain more fluxes simultaneously, so their CIs quantify the uncertainty reduction achievable from a single dual-isotope experiment rather than multiple labeling experiments.

## Related tools

- **INCA** (MATLAB-based environment used to run INST-MFA least-squares fitting of simulated labeling trajectories against measured isotopologue distributions and to extract fluxes with 95% confidence intervals)
- **SIMPEL** (R package that post-processes isotope-enriched HRMS datasets (isotopologue distributions, natural abundance correction) and exports the data tables consumed by INST-MFA in INCA) — github.com/SIMPELmetabolism/SIMPEL
- **MATLAB** (Runtime environment in which the INCA INST-MFA models are loaded and executed)

## Evaluation signals

- Reproduced flux values and 95% confidence intervals match those reported for the single, parallel, and dual labeling models in Fig. 2c of the paper
- Parallel 13C/15N and dual 13C15N models yield confidence intervals reduced by 71% and 84%, respectively, versus single 13C MFA
- Each INCA parameter estimation reaches a converged least-squares fit of simulated labeling trajectories against measured isotopologue distributions across the 0–8 h time course
- All fluxes are reported in nmol g-1 h-1 and tabulated side by side for single, parallel, and dual labeling models
- The three nominal-resolution datasets (13C, 15N, 13C15N) are derivable from one 13C15N-glutamine labeling experiment by summing isotopologue abundances from the comprehensive HRMS dataset

## Limitations

- No software versions reported for SIMPEL, XCMS, MZmine2, IsoCorrectoR, or INCA in the article
- Combining information from multiple heavy isotopes is limited when nominal m/z values are recorded, because dual-labeled tracers produce nominally identical mass shifts — HRMS is required
- The 71% and 84% CI reductions are specific to the reported Arabidopsis root glutamine labeling case and the particular INCA model configurations; they do not generalize automatically to other networks or tracer schemes

## Evidence

- [intro] Data tables exported from SIMPEL were used for INST-MFA with INCA.: "Data tables exported from SIMPEL were used for INST-MFA with INCA."
- [results] INST-MFA performed with INCA on SIMPEL-exported isotopologue distributions: "used to perform isotopically non-stationary metabolic flux analysis (INST-MFA) with INCA69"
- [results] All three nominal-resolution datasets derivable from one dual-isotope labeling experiment: "all three datasets could be derived from a single 13C15N-glutamine labeling experiment by summing isotopologue abundances contained in the comprehensive HRMS dataset"
- [abstract] Dual-isotope labeling with SIMPEL and INST-MFA resolves central metabolism fluxes: "a dual-isotope labeling experiment is paired with SIMPEL and isotopically nonstationary metabolic flux analysis (INST-MFA) to resolve fluxes in central metabolism"
- [intro] SIMPEL automates isotopologue distribution generation and natural abundance correction: "SIMPEL automates post-processing data analyses of isotope enriched metabolomics datasets, generates isotopologue distributions, performs natural abundance corrections"
