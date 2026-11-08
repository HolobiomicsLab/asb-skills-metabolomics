---
name: reliability-diagram-construction
description: Use when projected surrogate scores (e.g. 'high hscrp') from Nightingale
  Health 1H-NMR metabolomics must be checked for calibration against binarized phenotypes,
  using reliability diagrams plus ECE/MCE and log-loss deltas.
when_to_use_negative:
- The dataset lacks the required Nightingale metabolite columns — the surrogate scores
  (and hence calibration curves) cannot be computed until the upload passes the metabolite-presence
  check
- No binarized phenotype ground truth is available for the surrogate of interest,
  since Platt calibration and reliability diagrams require actual 0/1 labels
- The goal is regression calibration of continuous outcomes (e.g. MetaboAge), not
  calibration of probabilistic surrogate scores
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_2269
tools:
- name: MiMIR
  role: Provides plattCalibration (fit on the 80% calibration subset), plattCalib_evaluation
    (renders reliability diagrams on the 20% holdout), and getECE/getMCE statistics
  repo: https://github.com/DanieleBizzarri/MiMIR
- name: R
  role: Runtime environment (R 3.6+) in which the calibration, diagram rendering,
    and report generation are executed
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btac388
    title: 'MiMIR: R-shiny application to infer risk factors and endpoints from Nightingale
      Health’s 1H-NMR metabolomics data'
  source_content_hash: sha256:382a7b71d529e4023aa02bc9463d5afc5645d63cf328a7b3e641e07dcb5dfe02
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/DanieleBizzarri/MiMIR
  tools:
  - MiMIR
  - R
  - shiny
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# reliability-diagram-construction

## Summary

Construction of reliability diagrams to visually assess calibration of MiMIR metabolic surrogate scores, paired with ECE/MCE/log-loss statistics computed on a 20% holdout after fitting Platt calibration on an 80% calibration subset.

## When to use

Use when projected surrogate scores (e.g. 'high hscrp') from Nightingale Health 1H-NMR metabolomics must be checked for calibration against binarized phenotypes, using reliability diagrams plus ECE/MCE and log-loss deltas.

## When NOT to use

- The dataset lacks the required Nightingale metabolite columns — the surrogate scores (and hence calibration curves) cannot be computed until the upload passes the metabolite-presence check
- No binarized phenotype ground truth is available for the surrogate of interest, since Platt calibration and reliability diagrams require actual 0/1 labels
- The goal is regression calibration of continuous outcomes (e.g. MetaboAge), not calibration of probabilistic surrogate scores

## Inputs

- Projected MiMIR surrogate scores (numeric matrix of raw surrogate values, e.g. from calculate_surrogate_scores)
- Binarized phenotypes (0/1 vectors from binarize_all_pheno, one per surrogate)
- Number of bins (nbins) for reliability diagram binning and ECE/MCE computation
- MiMIR synthetic_metabolic_dataset / synthetic_phenotypic_dataset as stand-in cohort when no real data are uploaded

## Outputs

- Reliability diagram per surrogate (calibrated vs original curves plotted against the diagonal)
- ECE and MCE values per surrogate (original and calibrated) from getECE/getMCE
- Log-loss before and after calibration with per-surrogate deltas
- Summary table written to platt_calibration_report.csv

## How to apply

Load the projected surrogate scores and binarized phenotypes (fall back to MiMIR's synthetic_metabolic_dataset and synthetic_phenotypic_dataset when no real cohort is available) into R. Split samples into an 80% calibration subset and a 20% holdout subset, then fit Platt calibration — a logistic rescaling of raw surrogate scores against binarized phenotypes — with MiMIR::plattCalibration(r.calib, p.calib, nbins, pl). On the holdout, run MiMIR::plattCalib_evaluation(r, p, p.orig, name, nbins, annot_x, annot_y) to render a reliability diagram per surrogate; binned predicted probabilities are plotted against observed frequencies so that perfect calibration lies on the diagonal. Quantify mis-calibration with getECE(actual, predicted, n_bins) and getMCE(actual, predicted, n_bins), and compute log-loss before and after calibration for each surrogate. Tabulate calibrated-vs-original deltas per surrogate and write the summary to platt_calibration_report.csv; e.g. for 'high hscrp' in LLS_PAROFFs the calibrated values lie closer to the diagonal with ΔECE(calib-orig)=-0.063, ΔMCE(calib-orig)=-0.013, LogLoss(calib-orig)=-0.05.

## Related tools

- **MiMIR** (Provides plattCalibration (fit on the 80% calibration subset), plattCalib_evaluation (renders reliability diagrams on the 20% holdout), and getECE/getMCE statistics) — https://github.com/DanieleBizzarri/MiMIR
- **R** (Runtime environment (R 3.6+) in which the calibration, diagram rendering, and report generation are executed)

## Examples

```
MiMIR::plattCalib_evaluation(r, p, p.orig, name, nbins, annot_x, annot_y)
```

## Evaluation signals

- On the holdout, each calibrated surrogate shows decreased ECE relative to its original (uncalibrated) scores (ΔECE = calib − orig < 0)
- MCE decreases after calibration (ΔMCE < 0); reference run: ΔMCE(calib-orig) = -0.013 for 'high hscrp' in LLS_PAROFFs
- Log-loss on the holdout decreases after calibration (LogLoss(calib-orig) < 0; reference: -0.05 for 'high hscrp')
- In the reliability diagram the calibrated curve lies closer to the diagonal than the original curve
- platt_calibration_report.csv contains one row per surrogate with matching ECE/MCE/log-loss deltas consistent with the plotted diagrams

## Limitations

- Calibration quality is assessed on only a 20% holdout, so estimates of ECE/MCE/log-loss deltas depend on the split and bin count (nbins)
- Requires the App to find all necessary metabolites in the uploaded dataset; missing metabolites block score projection and thus calibration
- MiMIR is designed for GUI-based, pre-trained model calibration (24 models) rather than custom model development

## Evidence

- [methods] plattCalibration: "`plattCalibration(r.calib, p.calib, nbins, pl)`"
- [results] reliability diagrams on holdout: [evidence span withheld — anchor status: not_found]
- [readme] calibrate surrogate values: "project previously published metabolic scores; and calibrate the metabolic surrogate values to a desired dataset."
- [abstract] 24 pre-trained models: "allows for the projection and calibration of 24 pre-trained metabolomics-based models, without any pre-required programming knowledge"
