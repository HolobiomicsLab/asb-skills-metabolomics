---
name: isotopologue-distribution-fitting
description: Use when HRMS data from a transient stable-isotope labeling time course
  (e.g., [13C5,15N2]glutamine or [U-13C6]glucose) must be converted to isotopologue
  distributions and used to estimate metabolic fluxes.
when_to_use_negative:
- Data were acquired on nominal-resolution (low-mass-accuracy) MS, where dual-labeled
  isotopologues producing nominally identical mass shifts cannot be resolved
- The goal is untargeted metabolomics feature detection only, with no isotope labeling
  experiment — SIMPEL's isotopologue mining and NA correction steps are not applicable
- Steady-state labeling data where isotopically non-stationary MFA (INST-MFA) time-course
  fitting in INCA is unnecessary
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: SIMPEL
  role: R package that builds isotopologue distribution tables (MIDsTableNoScale),
    performs natural abundance correction (NACorrectionFxnII wrapping IsoCorrectoR),
    and exports data tables for INST-MFA
  repo: github.com/SIMPELmetabolism/SIMPEL
- name: XCMS
  role: Pre-processes the acquired HRMS data into a feature table prior to SIMPEL
    post-processing
- name: IsoCorrectoR
  role: Performs natural abundance correction of the isotopologue distribution table,
    wrapped by SIMPEL's NACorrectionFxnII
- name: INCA
  role: MATLAB environment in which the SIMPEL-exported isotopologue tables are fit
    by isotopically non-stationary MFA least-squares to yield fluxes and 95% CIs
provenance:
  source_task_ids:
  - task_002
  - task_001
  - task_005
  source_papers:
  - doi: 10.1038/s42003-024-05844-z
    title: 'SIMPEL: using stable isotopes to elucidate dynamics of context specific
      metabolism'
  source_content_hash: sha256:fda6d647612d351270045c8e5247dd550dbb935d4afe078d2faa16ea3c5b13a1
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - INCA
  - IsoCorrectoR
  - R
  - SIMPEL
  - XCMS
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# isotopologue-distribution-fitting

## Summary

Generates natural-abundance-corrected isotopologue distributions from HRMS stable-isotope labeling data using SIMPEL (R), then fits them with isotopically non-stationary metabolic flux analysis (INST-MFA) in INCA to estimate fluxes and 95% confidence intervals.

## When to use

Use when HRMS data from a transient stable-isotope labeling time course (e.g., [13C5,15N2]glutamine or [U-13C6]glucose) must be converted to isotopologue distributions and used to estimate metabolic fluxes.

## When NOT to use

- Data were acquired on nominal-resolution (low-mass-accuracy) MS, where dual-labeled isotopologues producing nominally identical mass shifts cannot be resolved
- The goal is untargeted metabolomics feature detection only, with no isotope labeling experiment — SIMPEL's isotopologue mining and NA correction steps are not applicable
- Steady-state labeling data where isotopically non-stationary MFA (INST-MFA) time-course fitting in INCA is unnecessary

## Inputs

- XCMS-pre-processed HRMS feature table from a stable-isotope labeling time course (e.g., Arabidopsis roots labeled with [13C5,15N2]glutamine, 0-8 h sampling)
- Compound database with molecular formulas, retention times, and polarity
- SIMPEL label_scheme defining the isotope label (e.g., 13C5,15N2)
- INCA model files encoding single-label, parallel-label, and dual-isotope-label configurations (MATLAB)
- High-resolution MS2 spectra of resolved isotopologues for acyl-chain labeling analysis
- Lipid pool size measurements for natom-equivalent scaling

## Outputs

- Natural-abundance-corrected isotopologue distribution table (NA_corrected_MIDs.csv)
- Average labeling values per compound over the time course
- Fitted reaction fluxes in nmol g-1 h-1 with 95% confidence intervals (flux_estimates_fig2c.csv)
- Acyl-chain natom-equivalent 13C enrichment table (MIDs_table_NA_corrected.csv)

## How to apply

Pre-process the HRMS data with XCMS, then import the feature table into the SIMPEL R package: build the compound m/z lookup with get_comp_mz_lookup using compound formulas, retention times, polarity, and label_scheme so features match by exact mass within a ppm tolerance; assign features to compound isotopologue groups with get_comp_stage given rt_tolerance; then call get_table_objects_NA_corrected to build the MIDsTableNoScale table and apply NACorrectionFxnII (wrapping IsoCorrectoR) for natural abundance correction. Export the corrected isotopologue distributions and average labeling values (e.g., NA_corrected_MIDs.csv), noting that 13C-only, 15N-only, and combined 13C15N datasets can be derived from a single dual-isotope experiment by summing isotopologue abundances. Load the resulting tables plus the single-, parallel-, and dual-label INCA model files into MATLAB and run INCA's INST-MFA least-squares fit of simulated labeling trajectories against measured isotopologue distributions; extract fitted fluxes (nmol g-1 h-1) and 95% CIs. Dual/parallel isotope designs are preferred because they reduce confidence intervals substantially versus single-isotope MFA. For lipid acyl-chain labeling (e.g., PC(36:4) 18:1/18:2/18:3), use MS2 spectra of resolved isotopologues (M0 vs M6), NA-correct the chain MIDs with NACorrectionFxnII, and check that summed pair weights reproduce the precursor isotopologue fraction.

## Related tools

- **SIMPEL** (R package that builds isotopologue distribution tables (MIDsTableNoScale), performs natural abundance correction (NACorrectionFxnII wrapping IsoCorrectoR), and exports data tables for INST-MFA) — github.com/SIMPELmetabolism/SIMPEL
- **XCMS** (Pre-processes the acquired HRMS data into a feature table prior to SIMPEL post-processing)
- **IsoCorrectoR** (Performs natural abundance correction of the isotopologue distribution table, wrapped by SIMPEL's NACorrectionFxnII)
- **INCA** (MATLAB environment in which the SIMPEL-exported isotopologue tables are fit by isotopically non-stationary MFA least-squares to yield fluxes and 95% CIs)

## Examples

```
get_table_objects_NA_corrected(XCMS_data, compounds_data, ppm, rt_tolerance, output, label_scheme)
```

## Evaluation signals

- Corrected MIDs from unlabeled (0 h) samples show natural-abundance contributions removed after NACorrectionFxnII
- Average labeling enrichment increases monotonically over the 0-8 h [13C5,15N2]glutamine feeding time course, matching Fig. 1/2 patterns
- Reproduced flux values and 95% CIs match those reported in Fig. 2c for the single, parallel, and dual labeling models (parallel and dual reduce CIs by ~71% and 84% vs single 13C MFA)
- For PC(36:4) acyl-chain analysis, summed pair weights (M0/M6, M3/M3, M6/M0) must reproduce the precursor isotopologue M6 fraction (conservation check)
- Feature-to-compound assignments hold within the specified ppm and rt_tolerance windows in get_comp_mz_lookup / get_comp_stage

## Limitations

- Combining multiple heavy isotopes is limited at nominal m/z resolution because dual-labeled tracers produce nominally identical mass shifts; HRMS resolving mass shifts <0.01 Da is required
- No software versions reported for SIMPEL, XCMS, MZmine2, IsoCorrectoR, or INCA, so exact reproduction may require version experimentation
- Only Depends packages are auto-attached in R; dependencies such as IsoCorrectoR must be explicitly attached with library()
- The number of multivariate isotopologues grows exponentially with labeled atoms per element (I = ∏(Ne+1)), complicating dual-isotope table handling

## Evidence

- [intro] Data tables exported from SIMPEL were used for INST-MFA with INCA.: "Data tables exported from SIMPEL were used for INST-MFA with INCA."
- [methods] MIDsTableNoScale signature: "MIDsTableNoScale(XCMS_data, label_scheme)"
- [results] Three datasets from one dual-label experiment: "all three datasets could be derived from a single 13C15N-glutamine labeling experiment by summing isotopologue abundances"
- [results] INST-MFA with INCA on SIMPEL data: "used to perform isotopically non-stationary metabolic flux analysis (INST-MFA) with INCA"
- [intro] HRMS resolves near-identical dual labels: "accurate measurement of m/z values that differ by <0.01 Da"
