---
name: multivariate-model-validation
description: Use when a supervised multivariate model (e.g. OPLS-DA) has separated
  sample classes (carriage vs controls) and you must quantify its diagnostic performance
  with ROC AUC values and confidence intervals.
when_to_use_negative:
- Input is a single-metabolite feature list rather than cross-validated scores from
  a supervised pattern-recognition model — the skill evaluates the multivariate latent
  biomarker pattern, not individual metabolites.
- The goal is unsupervised exploration of sample-group distribution — PCA is used
  for that; this skill only evaluates class-separating supervised models.
- Comparing carriage profiles against acute enteric fever profiles — the paper treats
  that as a separate heterogeneity question, not a ROC-validated diagnostic model.
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: Matlab
  role: Builds the ROC curves (false positive rate vs true positive rate) and computes
    AUC values with bootstrap percentile confidence intervals
- name: SIMCA
  role: Generates the supervised OPLS-DA cross-validated scores that are converted
    into ROC curves
- name: GCxGC-TOFMS
  role: Two-dimensional gas chromatography coupled with time-of-flight mass spectrometry
    platform producing the 195-metabolite plasma feature matrix underlying the models
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1371/journal.pntd.0006215
    title: Diagnostic metabolite biomarkers of chronic typhoid carriage
  source_content_hash: sha256:25674122da3c4e2d85fb521037d3a9555206d5c74b708e48a90b81015f9e71bf
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - GCxGC-TOFMS
  - Matlab
  - SIMCA
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# multivariate-model-validation

## Summary

Validates supervised chemometric models (OPLS-DA on GCxGC-TOFMS metabolite data) by converting cross-validated scores into ROC curves, computing AUCs, and reporting 95% confidence intervals from 1,000 bootstrap resamples to quantify diagnostic performance of latent metabolite patterns.

## When to use

Use when a supervised multivariate model (e.g. OPLS-DA) has separated sample classes (carriage vs controls) and you must quantify its diagnostic performance with ROC AUC values and confidence intervals.

## When NOT to use

- Input is a single-metabolite feature list rather than cross-validated scores from a supervised pattern-recognition model — the skill evaluates the multivariate latent biomarker pattern, not individual metabolites.
- The goal is unsupervised exploration of sample-group distribution — PCA is used for that; this skill only evaluates class-separating supervised models.
- Comparing carriage profiles against acute enteric fever profiles — the paper treats that as a separate heterogeneity question, not a ROC-validated diagnostic model.

## Inputs

- Cross-validated OPLS-DA score vectors from four pairwise models (carriage vs controls, S. Typhi vs controls, S. Paratyphi A vs controls, S. Typhi vs S. Paratyphi A carriage)
- Class labels for plasma samples from Nepali cholecystectomy patients (confirmed S. Typhi or S. Paratyphi A gallbladder carriage and non-carriage controls)

## Outputs

- ROC curves per pairwise model with false positive rate (1-specificity) vs true positive rate (sensitivity), as in Fig 2A-2D
- AUC values per model (0.5 = random, 1 = perfect classifier)
- Bootstrap 95% confidence intervals per AUC (1,000 resamples)
- Structured results table (auc_results.csv)

## How to apply

Load the cross-validated scores from each pairwise OPLS-DA model (combined Salmonella carriage vs non-carriage controls, S. Typhi vs controls, S. Paratyphi A vs controls, and S. Typhi vs S. Paratyphi A carriage), which are the supervised pattern-recognition models built on 195 metabolites measured by GCxGC-TOFMS. Build ROC curves in Matlab plotting false positive rate (1-specificity) on the x-axis against true positive rate (sensitivity) on the y-axis, as in Fig 2A-2D. Compute the area under each curve, where 0.5 represents a random classifier and 1 a perfect classifier. Estimate 95% confidence intervals for each AUC with the bootstrap percentile resampling method using 1,000 bootstraps. Acceptance criteria follow the paper: AUC >= 0.95 for the three carriage-vs-control models and AUC = 0.833 for the weaker S. Typhi vs S. Paratyphi A comparison.

## Related tools

- **Matlab** (Builds the ROC curves (false positive rate vs true positive rate) and computes AUC values with bootstrap percentile confidence intervals)
- **SIMCA** (Generates the supervised OPLS-DA cross-validated scores that are converted into ROC curves)
- **GCxGC-TOFMS** (Two-dimensional gas chromatography coupled with time-of-flight mass spectrometry platform producing the 195-metabolite plasma feature matrix underlying the models)

## Evaluation signals

- Each reproduced AUC for the carriage-vs-control ROC curves (Fig 2A-2C) must meet the paper's acceptance criterion of >= 0.95.
- The S. Typhi vs S. Paratyphi A serovar-comparison AUC (Fig 2D) must reproduce the reported value of 0.833.
- Bootstrap 95% confidence intervals (1,000 percentile resamples) should contain the reported AUC values.
- ROC axes must match the paper's convention: false positive rate (1-specificity) on x, true positive rate (sensitivity) on y.
- The pairwise separation structure must match the OPLS-DA scores: carriage vs controls along the first component and S. Typhi vs S. Paratyphi A along the second (p = 0.0031).

## Limitations

- Small sample size — described as the major limitation, difficult to overcome given ethical, surgical, and diagnostic issues of identifying and confirming invasive Salmonella carriage.
- No independent validation cohort — metabolite profiles should be validated in an additional independent cohort, but this was not possible in this setting.
- Potential confounding from underlying gallbladder conditions, since all patients presented with gallbladder conditions that may impact the metabolite signatures.

## Evidence

- [abstract] Supervised pattern recognition modeling on GCxGC-TOFMS data: "using two-dimensional gas chromatography coupled with time-of-flight mass spectrometry (GCxGC-TOFMS) and supervised pattern recognition modeling"
- [supplementary] ROC curve axis convention: "Panels A-E are showing ROC curves with false positive rates (i.e. 1-specificity) and true positive rates (i.e. sensitivity) on the x- and y-axes respectively"
- [supplementary] OPLS-DA component separation and serovar discrimination: "the separation of S. Typhi carriage samples from the S. Paratyphi A carriage samples along the second component (p = 0.0031)"
- [abstract] Diagnostic discrimination finding: "We were able to significantly discriminate Salmonella carriage samples from non-carriage control samples."
