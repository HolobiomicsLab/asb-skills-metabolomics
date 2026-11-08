---
name: lobov-validation-accuracy-interpretation
description: Use when projected Nightingale Health 1H-NMR surrogate scores (e.g. diabetes,
  high hscrp) need their discriminative performance (ROC AUC) and Platt-calibrated
  metrics (ECE/MCE/logLoss) validated against MiMIR's published LOBOV reference accuracies.
when_to_use_negative:
- The input is raw Nightingale Health metabolomics that has not yet been QC-prepped
  (QCprep/QCprep_surrogates) or scored — run score projection first
- The goal is mortality, MetaboAge, COVID-severity, or cardiovascular-event scores
  rather than the 24 surrogate clinical models, since these use different validation
  schemes than LOBOV surrogates
- The target cohort lacks phenotypic ground truth for the endpoint, making ROC AUC
  and calibration deltas uncomputable
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_2269
tools:
- name: MiMIR
  role: R package providing calculate_surrogate_scores, roc_surro/LOBOV_accuracies,
    plattCalibration, getECE/getMCE and the acc_LOBOV reference accuracies used to
    project, calibrate and validate surrogate scores
  repo: https://github.com/DanieleBizzarri/MiMIR
- name: pROC
  role: Statistics/imaging dependency installed by MiMIR used for ROC curve computation
    behind surrogate AUC evaluation
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btac388
    title: 'MiMIR: R-shiny application to infer risk factors and endpoints from Nightingale
      Health’s 1H-NMR metabolomics data'
  source_content_hash: sha256:cddd8754a977009921e8431fc2275c4a254c8b1b64abf016204ddd96a43e27ce
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/DanieleBizzarri/MiMIR
  tools:
  - MiMIR
  - R
  - pROC
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7857142857142857
n_claims: 14
---

# lobov-validation-accuracy-interpretation

## Summary

Interpret and benchmark leave-one-block-out validation (LOBOV) accuracies for metabolomics-based surrogate clinical scores projected with MiMIR, comparing case-study ROC AUCs and Platt-calibration deltas against the reference LOBOV values shipped in the package (acc_LOBOV dataset).

## When to use

Use when projected Nightingale Health 1H-NMR surrogate scores (e.g. diabetes, high hscrp) need their discriminative performance (ROC AUC) and Platt-calibrated metrics (ECE/MCE/logLoss) validated against MiMIR's published LOBOV reference accuracies.

## When NOT to use

- The input is raw Nightingale Health metabolomics that has not yet been QC-prepped (QCprep/QCprep_surrogates) or scored — run score projection first
- The goal is mortality, MetaboAge, COVID-severity, or cardiovascular-event scores rather than the 24 surrogate clinical models, since these use different validation schemes than LOBOV surrogates
- The target cohort lacks phenotypic ground truth for the endpoint, making ROC AUC and calibration deltas uncomputable

## Inputs

- Projected surrogate scores for the target cohort (e.g. LLS_PAROFFs) computed from Nightingale Health 1H-NMR metabolomics data
- acc_LOBOV reference dataset of surrogate-model validation accuracies shipped with MiMIR
- Phenotypic labels for binarized endpoints (e.g. diabetes, high hscrp) from binarize_all_pheno
- PARAM_surrogates parameter set required by calculate_surrogate_scores

## Outputs

- Per-surrogate ROC AUCs in the case-study cohort with matched mean LOBOV validation AUCs
- Platt-calibration deltas: ΔECE, ΔMCE, ΔlogLoss (e.g. -0.063, -0.013, -0.05)
- Structured results table (CSV/TSV) with metric_name, value, applies_under columns
- Qualitative text rows for non-numeric findings (e.g. lipid-medication accuracy notes)

## How to apply

Project the surrogate scores on the target cohort using MiMIR's calculate_surrogate_scores (with roc=TRUE to obtain ROC AUCs), then compare each surrogate's case-study AUC to the mean LOBOV AUC in the package's acc_LOBOV reference data — e.g. in the LLS_PAROFFs case study 'diabetes' reached AUC=0.85 (LOBOV 0.8) and 'high hscrp' AUC=0.811 (LOBOV 0.81), performing as well as or better than in the BBMRI-nl cohorts. Next, recalibrate projected surrogate probabilities to the target cohort with plattCalibration and quantify calibration improvement via getECE/getMCE and logLoss; in LLS_PAROFFs the deltas were ΔECE=-0.063, ΔMCE=-0.013, ΔlogLoss=-0.05 (negative deltas indicating improved calibration). Interpret performance jointly: high AUC confirms the projected score discriminates the endpoint, while negative ECE/MCE/logLoss deltas confirm the Platt step transferred probabilities correctly. Additionally check that the surrogate's underlying metabolic features are represented — e.g. the diabetes surrogate (comp.T2D_Ahola_Olli context) is built from 110 significantly associated metabolic features — and record qualitative observations (such as lipid-medication accuracy caveats) as text rows. Assemble AUCs, LOBOV references, calibration deltas and feature counts into a machine-readable table with metric_name, value, and applies_under columns.

## Related tools

- **MiMIR** (R package providing calculate_surrogate_scores, roc_surro/LOBOV_accuracies, plattCalibration, getECE/getMCE and the acc_LOBOV reference accuracies used to project, calibrate and validate surrogate scores) — https://github.com/DanieleBizzarri/MiMIR
- **pROC** (Statistics/imaging dependency installed by MiMIR used for ROC curve computation behind surrogate AUC evaluation)

## Examples

```
MiMIR::calculate_surrogate_scores(met, pheno, PARAM_surrogates, bin_names=NULL, Nmax_miss=0.05, Nmax_zero=0.2, post=TRUE, roc=TRUE, quiet=FALSE)
```

## Evaluation signals

- Every case-study AUC is paired with its mean LOBOV reference (diabetes 0.85 vs 0.8; high hscrp 0.811 vs 0.81) with no missing metric rows
- Calibration deltas match the reported values exactly: ΔECE=-0.063, ΔMCE=-0.013, ΔlogLoss=-0.05, all negative (improvement direction)
- Feature count for the diabetes surrogate equals 110 (comp.T2D_Ahola_Olli context)
- Output table conforms to machine-readable CSV/TSV with columns metric_name, value, applies_under using R data-frame conventions
- AUC values were computed via the roc_surro/LOBOV_accuracies functions rather than hand-coded ROC logic

## Limitations

- LOBOV reference accuracies are cohort-specific (validated in BBMRI-nl); case-study performance in other populations may deviate and requires the acc_LOBOV comparison to be interpreted cautiously
- Some surrogates have qualitative accuracy caveats (e.g. lipid-medication effects) that cannot be captured by AUC alone and must be recorded as text
- Calibration improvements depend on plattCalibration being run with appropriate nbins and on getECE/getMCE binning choices; deltas are not comparable if binning differs

## Evidence

- [intro] Project previously published metabolic scores; and calibrate the metabolic surrogate values to a desired dataset: "project previously published metabolic scores; and calibrate the metabolic surrogate values to a desired dataset"
- [methods] calculate_surrogate_scores signature: "calculate_surrogate_scores(met, pheno, PARAM_surrogates, bin_names, Nmax_miss, Nmax_zero, post, roc, quiet)"
- [methods] plattCalibration signature: "plattCalibration(r.calib, p.calib, nbins, pl)"
- [methods] Finnish cohorts T2D study context: "a prospective study of 11,896 young adults from four Finnish cohorts"
- [results] LOBOV accuracy comparison finding for LLS_PAROFFs: [evidence span withheld — anchor status: not_found]
- [readme] README citation of the surrogate-score manuscript: "1H-NMR metabolomics-based surrogates to impute common clinical risk factors and endpoints', EBioMedicine, vol. 75, p. 103764"
