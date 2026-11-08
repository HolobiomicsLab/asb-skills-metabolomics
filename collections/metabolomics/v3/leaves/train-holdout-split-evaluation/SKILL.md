---
name: train-holdout-split-evaluation
description: Use when surrogate scores (e.g. MiMIR 'high hscrp' phenotypes) must be
  calibrated to a new cohort and calibration quality must be assessed out-of-sample
  on a held-out fraction of the data.
when_to_use_negative:
- When the goal is to evaluate the discriminative ability (e.g. ROC/AUC) of the uncalibrated
  surrogate scores themselves rather than calibration quality after rescaling
- When the cohort is too small for a meaningful 20% holdout — the split leaves too
  few events per bin for stable ECE/MCE estimates
- When no binarized phenotypes (ground-truth labels) are available, since Platt rescaling
  and log-loss deltas both require them
edam_operation: http://edamontology.org/operation_3658
edam_topics:
- http://edamontology.org/topic_3518
- http://edamontology.org/topic_0091
tools:
- name: MiMIR
  role: Provides plattCalibration, plattCalib_evaluation, getECE, getMCE and binarize_all_pheno
    functions for the 80/20 split, calibration and holdout evaluation
  repo: https://github.com/DanieleBizzarri/MiMIR
- name: R
  role: Runtime environment (version 3.6+) in which the split, calibration and evaluation
    functions are executed
- name: shiny
  role: GUI layer of the MiMIR app where the calibration tab and reliability-diagram
    figures are rendered without programming knowledge
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btac388
    title: 'MiMIR: R-shiny application to infer risk factors and endpoints from Nightingale
      Health’s 1H-NMR metabolomics data'
  source_content_hash: sha256:2c92c446372c7675487a840a44adc49a4de18c26b27b2be573b3f8192dfa5dbc
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/DanieleBizzarri/MiMIR
  tools:
  - MiMIR
  - R
  - shiny
  edam_operation: http://edamontology.org/operation_3658
  edam_topics:
  - http://edamontology.org/topic_3518
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# train-holdout-split-evaluation

## Summary

Splits a cohort into an 80% calibration subset and a 20% holdout subset so that Platt-calibrated MiMIR metabolic surrogate scores can be evaluated on unseen samples using reliability diagrams, ECE, MCE and log-loss deltas rather than in-sample fit.

## When to use

Use when surrogate scores (e.g. MiMIR 'high hscrp' phenotypes) must be calibrated to a new cohort and calibration quality must be assessed out-of-sample on a held-out fraction of the data.

## When NOT to use

- When the goal is to evaluate the discriminative ability (e.g. ROC/AUC) of the uncalibrated surrogate scores themselves rather than calibration quality after rescaling
- When the cohort is too small for a meaningful 20% holdout — the split leaves too few events per bin for stable ECE/MCE estimates
- When no binarized phenotypes (ground-truth labels) are available, since Platt rescaling and log-loss deltas both require them

## Inputs

- Projected MiMIR surrogate scores (numeric vector per surrogate) from Nightingale Health 1H-NMR metabolomics data (CSV/TSV upload)
- Binarized phenotypes from MiMIR::binarize_all_pheno(data) (e.g. synthetic_phenotypic_dataset)
- Number of bins (nbins) for reliability diagrams and calibration-bin statistics
- Plot annotations annot_x and annot_y for the reliability diagrams

## Outputs

- Reliability diagrams per surrogate showing calibrated vs original scores against the diagonal
- ECE and MCE statistics per surrogate on the 20% holdout (getECE/getMCE with n_bins)
- Pre- vs post-calibration log-loss per surrogate
- Delta table (ΔECE, ΔMCE, ΔLogLoss per surrogate) written to platt_calibration_report.csv

## How to apply

Load the projected surrogate scores and binarized phenotypes in R (MiMIR::synthetic_metabolic_dataset and synthetic_phenotypic_dataset serve as example inputs when no real cohort is uploaded). Randomly split samples into 80% for fitting and 20% for holdout evaluation. On the calibration subset, fit Platt calibration (logistic rescaling of raw scores against binarized phenotypes) via MiMIR::plattCalibration(r.calib, p.calib, nbins, pl). On the 20% holdout, run MiMIR::plattCalib_evaluation(r, p, p.orig, name, nbins, annot_x, annot_y), computing ECE with getECE(actual, predicted, n_bins) and MCE with getMCE(actual, predicted, n_bins). Compute log-loss before and after calibration per surrogate, tabulate calibrated-vs-original deltas, and render reliability diagrams; e.g. for 'high hscrp' in LLS_PAROFFs the calibrated points lie closer to the diagonal with ΔECE=-0.063, ΔMCE=-0.013 and LogLoss(calib-orig)=-0.05.

## Related tools

- **MiMIR** (Provides plattCalibration, plattCalib_evaluation, getECE, getMCE and binarize_all_pheno functions for the 80/20 split, calibration and holdout evaluation) — https://github.com/DanieleBizzarri/MiMIR
- **R** (Runtime environment (version 3.6+) in which the split, calibration and evaluation functions are executed)
- **shiny** (GUI layer of the MiMIR app where the calibration tab and reliability-diagram figures are rendered without programming knowledge)

## Examples

```
MiMIR::plattCalib_evaluation(r, p, p.orig, name, nbins, annot_x, annot_y)
```

## Evaluation signals

- On the 20% holdout, each calibrated surrogate shows a decreased ECE relative to its original (uncalibrated) score — e.g. ΔECE(calib-orig)=-0.063 for 'high hscrp' in LLS_PAROFFs
- Each calibrated surrogate shows a decreased log-loss relative to the original scores (e.g. LogLoss(calib-orig)=-0.05)
- MCE also decreases after calibration (e.g. ΔMCE=-0.013 for 'high hscrp')
- Reliability diagrams show calibrated points lying closer to the diagonal than the original points
- The summary table is fully populated with one row per surrogate and is written to platt_calibration_report.csv

## Limitations

- Evaluation is only as good as the 20% holdout; calibration is fit on 80% of the uploaded data, so very small cohorts give unstable ECE/MCE bins
- Metrics require binarized phenotypes; the App must find all necessary metabolites in the uploaded dataset before scores can be projected
- No discussion-section content or changelog was available for this article, so long-term behavior or versioning caveats of the calibration module are not documented

## Evidence

- [other] Platt Calibration is applied to the surrogate scores on 80% of the uploaded data, and calibration quality is assessed on the 20% holdout using reliability diagrams plus ECE, MCE and decrease in log-loss statistics: [evidence span withheld — anchor status: not_found]
- [methods] plattCalibration signature: "`plattCalibration(r.calib, p.calib, nbins, pl)`"
- [intro] calibration findings: "and calibrate the metabolic surrogate values to a desired dataset"
- [methods] synthetic example data: "the metabolic synthetic dataset, the phenotypic synthetic dataset. These example dataset can be used to test the App"
- [readme] app installation: "devtools::install_github("DanieleBizzarri/MiMIR")"
