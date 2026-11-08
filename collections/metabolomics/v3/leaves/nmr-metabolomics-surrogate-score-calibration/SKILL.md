---
name: nmr-metabolomics-surrogate-score-calibration
description: Use when projected 1H-NMR surrogate scores (e.g. 'high hscrp') from MiMIR
  need recalibration to a desired cohort, and calibration gain must be quantified
  on a held-out fraction of samples.
when_to_use_negative:
- The metabolites matrix has not yet been projected to surrogate scores (run QCprep_surrogates
  / calculate_surrogate_scores first — calibration operates on scores, not raw metabolite
  concentrations).
- No binarized phenotype labels are available for the target dataset, since Platt
  rescaling requires binary outcomes to fit against.
- The goal is computing the surrogate scores themselves (mortality score, MetaboAge,
  COVID-severity, T2D) rather than recalibrating existing surrogate values.
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3517
tools:
- name: MiMIR
  role: Provides plattCalibration, plattCalib_evaluation, getECE, getMCE and the Shiny
    app calibration tab
  repo: https://github.com/DanieleBizzarri/MiMIR
- name: R
  role: Runtime environment (version 3.6+) in which the MiMIR calibration functions
    are executed
  repo: https://github.com/DanieleBizzarri/MiMIR
- name: R-shiny
  role: GUI front-end exposing upload of metabolite/phenotype CSV-TSV data, calibration
    and reliability-diagram figures without programming
  repo: https://github.com/DanieleBizzarri/MiMIR
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btac388
    title: 'MiMIR: R-shiny application to infer risk factors and endpoints from Nightingale
      Health’s 1H-NMR metabolomics data'
  source_content_hash: sha256:40ec73c76a9c0ad4f25de4e7ada08b282f05232f60b52bb89fd19f82da89cdb2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/DanieleBizzarri/MiMIR
  tools:
  - MiMIR
  - R
  - R-shiny
  - shiny
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3517
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# nmr-metabolomics-surrogate-score-calibration

## Summary

Platt (logistic-rescaling) calibration of MiMIR metabolic surrogate scores derived from Nightingale Health 1H-NMR metabolomics, fitted on an 80% calibration subset and evaluated on a 20% holdout using reliability diagrams, ECE, MCE and log-loss deltas.

## When to use

Use when projected 1H-NMR surrogate scores (e.g. 'high hscrp') from MiMIR need recalibration to a desired cohort, and calibration gain must be quantified on a held-out fraction of samples.

## When NOT to use

- The metabolites matrix has not yet been projected to surrogate scores (run QCprep_surrogates / calculate_surrogate_scores first — calibration operates on scores, not raw metabolite concentrations).
- No binarized phenotype labels are available for the target dataset, since Platt rescaling requires binary outcomes to fit against.
- The goal is computing the surrogate scores themselves (mortality score, MetaboAge, COVID-severity, T2D) rather than recalibrating existing surrogate values.

## Inputs

- Projected MiMIR surrogate scores matrix (from calculate_surrogate_scores on Nightingale Health 1H-NMR metabolites)
- Binarized phenotypic dataset (CSV/TSV; MiMIR::binarize_all_pheno output, or the packaged synthetic_phenotypic_dataset)
- Number of bins (nbins) for calibration/evaluation reliability diagrams
- 80/20 calibration/holdout sample split

## Outputs

- Platt-calibrated surrogate scores per sample
- Reliability diagrams (calibration plots) per surrogate
- Summary table platt_calibration_report.csv with ΔECE(calib-orig), ΔMCE(calib-orig) and LogLoss(calib-orig) per surrogate

## How to apply

Load the projected surrogate scores together with binarized phenotypes (via MiMIR::binarize_all_pheno, or the packaged synthetic_metabolic_dataset and synthetic_phenotypic_dataset when no real cohort is available) into R 3.6+. Split samples into an 80% calibration subset and a 20% holdout subset. Fit Platt calibration — a logistic rescaling of raw scores against binarized phenotypes — on the calibration subset with MiMIR::plattCalibration(r.calib, p.calib, nbins, pl). On the holdout, run MiMIR::plattCalib_evaluation(r, p, p.orig, name, nbins, annot_x, annot_y) to render per-surrogate reliability diagrams, and compute ECE with getECE(actual, predicted, n_bins) and MCE with getMCE(actual, predicted, n_bins). Tabulate calibrated-vs-original deltas per surrogate (ΔECE, ΔMCE, LogLoss(calib-orig), e.g. ΔECE=-0.063, ΔMCE=-0.013, log-loss decrease of -0.05 for 'high hscrp' in LLS_PAROFFs) and write them to platt_calibration_report.csv. Validation requires each calibrated surrogate to show decreased ECE and log-loss versus its uncalibrated scores, with reliability-diagram points lying closer to the diagonal.

## Related tools

- **MiMIR** (Provides plattCalibration, plattCalib_evaluation, getECE, getMCE and the Shiny app calibration tab) — https://github.com/DanieleBizzarri/MiMIR
- **R** (Runtime environment (version 3.6+) in which the MiMIR calibration functions are executed) — https://github.com/DanieleBizzarri/MiMIR
- **R-shiny** (GUI front-end exposing upload of metabolite/phenotype CSV-TSV data, calibration and reliability-diagram figures without programming) — https://github.com/DanieleBizzarri/MiMIR

## Examples

```
library(MiMIR); calib <- plattCalibration(r.calib, p.calib, nbins=10, pl=NULL); plattCalib_evaluation(r, p, p.orig, name, nbins=10, annot_x, annot_y)
```

## Evaluation signals

- On the 20% holdout, every calibrated surrogate shows decreased ECE relative to its uncalibrated scores (e.g. ΔECE(calib-orig) = -0.063 for 'high hscrp' in LLS_PAROFFs).
- MCE decreases after calibration (ΔMCE(calib-orig) = -0.013 for 'high hscrp' in LLS_PAROFFs).
- Log-loss decreases after calibration (LogLoss(calib-orig) = -0.05 for 'high hscrp').
- Reliability-diagram points of calibrated scores lie closer to the diagonal than those of original scores.
- platt_calibration_report.csv is written with a per-surrogate row of delta statistics.

## Limitations

- Calibration quality is only verified on a 20% holdout; only one worked example ('high hscrp' in LLS_PAROFFs) is documented with delta values, so per-surrogate gains vary and must be re-checked per dataset.
- The app cannot proceed if the uploaded CSV/TSV metabolites lack column names matching the example dataset; required-metabolite checks gate the workflow.
- The discussion-section extraction found no additional content (no changelog), so no longitudinal/robustness caveats are documented in the paper.

## Evidence

- [methods] plattCalibration function: "`plattCalibration(r.calib, p.calib, nbins, pl)`"
- [intro] calibration to desired dataset: "and calibrate the metabolic surrogate values to a desired dataset"
- [abstract] GUI without programming: "allows for the projection and calibration of 24 pre-trained metabolomics-based models, without any pre-required programming knowledge"
- [readme] example datasets: "containing 2 files: the metabolic synthetic dataset, the phenotypic synthetic dataset. These example dataset can be used to test the App"
- [intro] calibration supported by paper: "project previously published metabolic scores; and calibrate the metabolic surrogate values to a desired dataset"
