---
name: platt-probability-calibration
description: Use when raw surrogate scores from 1H-NMR metabolomics models (e.g. 'high
  hscrp') need rescaling to a target cohort, and calibration quality must be quantified
  against binarized phenotypes on a holdout set.
when_to_use_negative:
- The metabolomics data have not yet been projected onto the pre-trained surrogate
  models (run QCprep_surrogates/calculate_surrogate_scores first)
- No binarized phenotype labels exist for the target dataset, so logistic rescaling
  of scores against outcomes cannot be fit
- Only metabolite exploration or MetaboAge/mortality score projection is needed without
  cohort-specific calibration
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: MiMIR
  role: Provides plattCalibration, plattCalib_evaluation, getECE, getMCE and the Shiny
    calibration module for surrogate scores
  repo: https://github.com/DanieleBizzarri/MiMIR
- name: R
  role: Runtime environment (version 3.6+) in which the calibration functions are
    executed
  repo: https://github.com/DanieleBizzarri/MiMIR
- name: shiny
  role: GUI framework exposing the calibration workflow in the MiMIR app without programming
    knowledge
  repo: https://github.com/DanieleBizzarri/MiMIR
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btac388
    title: 'MiMIR: R-shiny application to infer risk factors and endpoints from Nightingale
      Health’s 1H-NMR metabolomics data'
  source_content_hash: sha256:ee61fdc9efc529c9e89446b7c4dc0fbdd47feccb2c543fed44709705675b8ff4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/DanieleBizzarri/MiMIR
  tools:
  - MiMIR
  - R
  - shiny
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# platt-probability-calibration

## Summary

Fits Platt calibration (logistic rescaling) of MiMIR's metabolic surrogate scores against binarized phenotypes on an 80% training split, then quantifies calibration improvement (reliability diagrams, ECE, MCE, log-loss) on the held-out 20%. Grounded in the MiMIR R-shiny application for Nightingale Health 1H-NMR metabolomics.

## When to use

Use when raw surrogate scores from 1H-NMR metabolomics models (e.g. 'high hscrp') need rescaling to a target cohort, and calibration quality must be quantified against binarized phenotypes on a holdout set.

## When NOT to use

- The metabolomics data have not yet been projected onto the pre-trained surrogate models (run QCprep_surrogates/calculate_surrogate_scores first)
- No binarized phenotype labels exist for the target dataset, so logistic rescaling of scores against outcomes cannot be fit
- Only metabolite exploration or MetaboAge/mortality score projection is needed without cohort-specific calibration

## Inputs

- Projected surrogate scores matrix from MiMIR's pre-trained metabolomics-based models (e.g. surrogate clinical variables from Nightingale Health 1H-NMR metabolomics)
- Binarized phenotypes (produced via binarize_all_pheno or matching phenotypic dataset, e.g. synthetic_phenotypic_dataset)
- Number of bins (nbins) for reliability diagrams and ECE/MCE computation

## Outputs

- Calibrated surrogate probability scores per sample
- Reliability diagrams (calibration plots) per surrogate
- Delta table of ΔECE, ΔMCE and log-loss(calibrated - original) per surrogate, e.g. platt_calibration_report.csv

## How to apply

Load the projected surrogate scores and binarized phenotypes into R (use MiMIR's synthetic_metabolic_dataset and synthetic_phenotypic_dataset to test the pipeline when no real cohort is available). Split the samples into an 80% calibration subset and a 20% holdout subset. Fit Platt calibration — a logistic rescaling of the raw surrogate scores against the binarized phenotypes — on the 80% subset with MiMIR::plattCalibration(r.calib, p.calib, nbins, pl), choosing the number of bins for the reliability diagram. Evaluate on the 20% holdout with MiMIR::plattCalib_evaluation(r, p, p.orig, name, nbins, annot_x, annot_y), computing ECE via getECE(actual, predicted, n_bins) and MCE via getMCE(actual, predicted, n_bins). Compute log-loss before and after calibration for each surrogate, tabulate calibrated-vs-original deltas, render per-surrogate reliability diagrams, and write the summary table (e.g. platt_calibration_report.csv). The rationale is that surrogate scores are not intrinsically probabilities; calibrating them to the desired dataset makes them comparable across cohorts, and the 20% holdout provides an unbiased estimate of the calibration gain.

## Related tools

- **MiMIR** (Provides plattCalibration, plattCalib_evaluation, getECE, getMCE and the Shiny calibration module for surrogate scores) — https://github.com/DanieleBizzarri/MiMIR
- **R** (Runtime environment (version 3.6+) in which the calibration functions are executed) — https://github.com/DanieleBizzarri/MiMIR
- **shiny** (GUI framework exposing the calibration workflow in the MiMIR app without programming knowledge) — https://github.com/DanieleBizzarri/MiMIR

## Examples

```
platt <- MiMIR::plattCalibration(r.calib, p.calib, nbins, pl)
```

## Evaluation signals

- On the 20% holdout, each calibrated surrogate shows decreased ECE relative to its original (uncalibrated) scores, e.g. ΔECE(calib-orig)=-0.063 for 'high hscrp' in LLS_PAROFFs
- MCE decreases after calibration (e.g. ΔMCE(calib-orig)=-0.013), confirming improved worst-bin reliability
- Log-loss decreases after calibration (e.g. LogLoss(calib-orig)=-0.05) for every surrogate in the delta table
- Calibrated points in the reliability diagrams lie closer to the diagonal than uncalibrated points
- Report CSV contains one row per surrogate with matched original-vs-calibrated metric deltas and no missing values

## Limitations

- Calibration requires binarized phenotypes; surrogate scores cannot be calibrated to a dataset lacking these labels
- The calibration fit is cohort-specific — models must be recalibrated when applied to a new desired dataset
- Evaluation depends on the choice of nbins; unreliable bins or small holdout sizes can distort ECE/MCE estimates
- The README is the primary documentation; the article provides no discussion-section caveats or changelog, so failure modes beyond those stated are undocumented

## Evidence

- [methods] plattCalibration signature: "`plattCalibration(r.calib, p.calib, nbins, pl)`"
- [readme] calibration purpose: "calibrate the metabolic surrogate values to a desired dataset"
