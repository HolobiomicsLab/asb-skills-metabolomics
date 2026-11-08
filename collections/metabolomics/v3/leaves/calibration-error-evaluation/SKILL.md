---
name: calibration-error-evaluation
description: Use when binary surrogate or risk scores have been (or will be) recalibrated
  to a target cohort and you must verify that predicted probabilities match observed
  phenotype frequencies, quantitatively via ECE/MCE/log-loss on a holdout and visually
  via reliability diagrams.
when_to_use_negative:
- When the goal is discrimination (AUC/ROC) rather than probability calibration —
  MiMIR computes ROC separately and calibration error does not assess ranking ability.
- When no binarized phenotype/ground truth is available to compare scores against,
  since ECE/MCE/log-loss all require observed binary outcomes.
- When the entire dataset was used to fit the calibration — evaluating on the same
  samples would overstate calibration quality; a held-out subset is required.
edam_operation: http://edamontology.org/operation_3664
edam_topics:
- http://edamontology.org/topic_3518
- http://edamontology.org/topic_0203
tools:
- name: MiMIR
  role: Provides plattCalibration, plattCalib_evaluation, getECE, getMCE and reliability-diagram
    rendering for calibrating and evaluating surrogate scores on Nightingale Health
    1H-NMR metabolomics data
  repo: https://github.com/DanieleBizzarri/MiMIR
- name: R
  role: Runtime environment for the MiMIR package functions (R version 3.6+)
- name: shiny
  role: GUI layer of MiMIR through which the calibration evaluation can be performed
    without programming knowledge
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btac388
    title: 'MiMIR: R-shiny application to infer risk factors and endpoints from Nightingale
      Health’s 1H-NMR metabolomics data'
  source_content_hash: sha256:50e9227d1c05d4649ebd60868c1cd47fce7498acfea4b98e2c30bfc4a08d7cb9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/DanieleBizzarri/MiMIR
  tools:
  - MiMIR
  - R
  - shiny
  edam_operation: http://edamontology.org/operation_3664
  edam_topics:
  - http://edamontology.org/topic_3518
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7647058823529411
n_claims: 17
---

# calibration-error-evaluation

## Summary

Quantifies calibration quality of surrogate scores (e.g. MiMIR's metabolomics-based surrogates such as 'high hscrp') after Platt calibration by computing expected/maximum calibration error (ECE/MCE) and log-loss on a held-out subset and rendering reliability diagrams against the diagonal.

## When to use

Use when binary surrogate or risk scores have been (or will be) recalibrated to a target cohort and you must verify that predicted probabilities match observed phenotype frequencies, quantitatively via ECE/MCE/log-loss on a holdout and visually via reliability diagrams.

## When NOT to use

- When the goal is discrimination (AUC/ROC) rather than probability calibration — MiMIR computes ROC separately and calibration error does not assess ranking ability.
- When no binarized phenotype/ground truth is available to compare scores against, since ECE/MCE/log-loss all require observed binary outcomes.
- When the entire dataset was used to fit the calibration — evaluating on the same samples would overstate calibration quality; a held-out subset is required.

## Inputs

- Raw surrogate scores projected from Nightingale Health 1H-NMR metabolomics data (e.g. MiMIR synthetic_metabolic_dataset or a real cohort)
- Binarized phenotypes (e.g. via MiMIR::binarize_all_pheno on the synthetic_phenotypic_dataset)
- Platt-calibrated scores fitted on the 80% calibration subset
- Number of bins (nbins) for binned reliability/computation

## Outputs

- Per-surrogate reliability diagrams with annotations (from MiMIR::plattCalib_evaluation)
- ECE and MCE values per surrogate (original vs calibrated)
- Log-loss before and after calibration per surrogate
- Delta table written to platt_calibration_report.csv

## How to apply

Split the scored samples into an 80% calibration subset (used to fit the Platt logistic rescaling of raw scores against binarized phenotypes, e.g. via MiMIR::plattCalibration(r.calib, p.calib, nbins, pl)) and a 20% holdout subset reserved strictly for evaluation. On the holdout, compare the original (uncalibrated) and calibrated scores against the binarized phenotypes using MiMIR::plattCalib_evaluation(r, p, p.orig, name, nbins, annot_x, annot_y), computing ECE with getECE(actual, predicted, n_bins) and MCE with getMCE(actual, predicted, n_bins) over the chosen number of bins. Compute log-loss before and after calibration per surrogate and tabulate the calibrated-vs-original deltas (e.g. for 'high hscrp' in LLS_PAROFFs: ΔECE=-0.063, ΔMCE=-0.013, LogLoss(calib-orig)=-0.05). Render per-surrogate reliability diagrams and write the summary table to platt_calibration_report.csv. Correctness is judged by ΔECE and Δlog-loss being negative (i.e., calibration improved error) and calibrated points lying closer to the diagonal in the reliability diagrams.

## Related tools

- **MiMIR** (Provides plattCalibration, plattCalib_evaluation, getECE, getMCE and reliability-diagram rendering for calibrating and evaluating surrogate scores on Nightingale Health 1H-NMR metabolomics data) — https://github.com/DanieleBizzarri/MiMIR
- **R** (Runtime environment for the MiMIR package functions (R version 3.6+))
- **shiny** (GUI layer of MiMIR through which the calibration evaluation can be performed without programming knowledge)

## Examples

```
library(MiMIR); calib <- plattCalibration(r.calib, p.calib, nbins); plattCalib_evaluation(r, p, p.orig, name, nbins, annot_x, annot_y)
```

## Evaluation signals

- On the 20% holdout, each calibrated surrogate must show decreased ECE relative to its original (uncalibrated) scores (ΔECE < 0, e.g. -0.063 for 'high hscrp' in LLS_PAROFFs).
- MCE must also decrease after calibration (ΔMCE < 0, e.g. -0.013).
- Log-loss on the holdout must decrease after calibration (LogLoss(calib-orig) < 0, e.g. -0.05).
- Reliability diagrams: calibrated binned mean predictions must lie closer to the diagonal than the original scores.
- platt_calibration_report.csv exists and contains calibrated-vs-original delta rows for every surrogate.

## Limitations

- Evaluation depends on the chosen number of bins (nbins) for binning predicted probabilities; ECE/MCE values are bin-sensitive.
- Only 20% of data is reserved for evaluation, so holdout-based deltas are noisy for small cohorts.
- Log-loss and calibration errors require reliable binarized phenotypes; mislabeled or missing phenotypes invalidate the comparison.
- Calibration improves probability estimates but does not change the ranking/discrimination of the surrogate scores.

## Evidence

- [results] Platt Calibration is applied to the surrogate scores on 80% of the uploaded data, and calibration quality is assessed on the 20% holdout using reliability diagrams plus ECE, MCE and decrease in log-loss statistics: [evidence span withheld — anchor status: not_found]
- [intro] MiMIR calibrates surrogate scores to a desired dataset: "and calibrate the metabolic surrogate values to a desired dataset"
- [methods] plattCalibration function signature: "`plattCalibration(r.calib, p.calib, nbins, pl)`"
- [readme] Synthetic example datasets for testing the app: "the metabolic synthetic dataset, the phenotypic synthetic dataset. These example dataset can be used to test the App and to understand how the variables in your own dataset should be named."
- [abstract] MiMIR projects and calibrates 24 pre-trained models without programming knowledge: "allows for the projection and calibration of 24 pre-trained metabolomics-based models, without any pre-required programming knowledge"
