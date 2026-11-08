---
name: metabolomics-diagnostic-performance-reproduction
description: Use when cross-validated supervised model scores (e.g. OPLS-DA scores)
  are available for pairwise sample groups and you must quantify diagnostic performance
  as ROC AUC with bootstrap confidence intervals, reproducing reported classifier
  values.
when_to_use_negative:
- Input is a raw GCxGC-TOFMS peak/feature table rather than cross-validated OPLS-DA
  model scores — model building (e.g. in SIMCA) must happen first
- The goal is unsupervised sample separation (PCA scores) or multiclass OPLS-DA separation
  rather than pairwise diagnostic-performance quantification
- The goal is identifying the five-metabolite minimal biomarker panel or comparing
  carriage profiles with acute infection profiles — those are separate skills
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_3361
- http://edamontology.org/topic_0091
tools:
- name: Matlab
  role: Builds the ROC curves (false positive rate vs true positive rate) and computes
    AUC with 1,000-bootstrap percentile confidence intervals
- name: SIMCA
  role: Produces the cross-validated OPLS-DA model scores for the four pairwise comparisons
    that are fed into the ROC analysis
- name: GCxGC-TOFMS
  role: Instrument platform whose plasma metabolite measurements underlie the 195-metabolite
    data matrix used for the models
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1371/journal.pntd.0006215
    title: Diagnostic metabolite biomarkers of chronic typhoid carriage
  source_content_hash: sha256:6b929ca27fe15ccad3b62420f68b9ff1a41df1ce2bc59ef7d9891784cbb4fb71
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - GCxGC-TOFMS
  - Matlab
  - SIMCA
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_3361
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# metabolomics-diagnostic-performance-reproduction

## Summary

Reproduces the ROC AUC diagnostic performance of the paper's cross-validated pairwise OPLS-DA models built on GCxGC-TOFMS plasma metabolite data, distinguishing Salmonella carriage vs non-carriage controls and S. Typhi vs S. Paratyphi A carriage, with bootstrap 95% confidence intervals.

## When to use

Use when cross-validated supervised model scores (e.g. OPLS-DA scores) are available for pairwise sample groups and you must quantify diagnostic performance as ROC AUC with bootstrap confidence intervals, reproducing reported classifier values.

## When NOT to use

- Input is a raw GCxGC-TOFMS peak/feature table rather than cross-validated OPLS-DA model scores — model building (e.g. in SIMCA) must happen first
- The goal is unsupervised sample separation (PCA scores) or multiclass OPLS-DA separation rather than pairwise diagnostic-performance quantification
- The goal is identifying the five-metabolite minimal biomarker panel or comparing carriage profiles with acute infection profiles — those are separate skills

## Inputs

- Cross-validated score vectors (one per sample) from the four pairwise OPLS-DA models fitted on 195 GCxGC-TOFMS metabolites
- Binary class labels for each pairwise comparison (carriage vs control; S. Typhi vs S. Paratyphi A carriage)
- Bootstrap specification: percentile resampling with 1,000 bootstraps

## Outputs

- ROC curve per pairwise model (false positive rate vs true positive rate), as in Fig 2A-2D
- Point estimate of AUC per model with 95% bootstrap percentile confidence interval
- Structured results table auc_results.csv

## How to apply

Load the cross-validated scores from the four pairwise OPLS-DA models: combined Salmonella carriage vs non-carriage controls, S. Typhi carriage vs controls, S. Paratyphi A carriage vs controls, and S. Typhi carriage vs S. Paratyphi A carriage. For each model, build the ROC curve in Matlab with false positive rate (1-specificity) on the x-axis and true positive rate (sensitivity) on the y-axis, mirroring Fig 2A-2D. Compute the area under each curve, interpreting 0.5 as a random classifier and 1 as a perfect classifier. Estimate 95% confidence intervals with the bootstrap percentile resampling method using 1,000 bootstraps. Verify reproduced values against the acceptance criteria: AUC >= 0.95 for the three carriage-vs-control curves and AUC = 0.833 for the S. Typhi vs S. Paratyphi A comparison, then write AUC values and confidence intervals to auc_results.csv.

## Related tools

- **Matlab** (Builds the ROC curves (false positive rate vs true positive rate) and computes AUC with 1,000-bootstrap percentile confidence intervals)
- **SIMCA** (Produces the cross-validated OPLS-DA model scores for the four pairwise comparisons that are fed into the ROC analysis)
- **GCxGC-TOFMS** (Instrument platform whose plasma metabolite measurements underlie the 195-metabolite data matrix used for the models)

## Evaluation signals

- Reproduced AUC for each of the three carriage-vs-control models (combined, S. Typhi vs controls, S. Paratyphi A vs controls) is >= 0.95
- Reproduced AUC for the S. Typhi vs S. Paratyphi A carriage model equals 0.833 within bootstrap 95% confidence intervals
- ROC curve axes match Fig 2: false positive rate (1-specificity) on x, true positive rate (sensitivity) on y
- Confidence intervals are derived from 1,000 bootstrap percentile resamples, not from a parametric method
- auc_results.csv contains AUC values and confidence intervals for all four pairwise models in structured form

## Limitations

- The major limitation of the underlying study was the small sample size, limiting the stability of the reproduced AUC estimates
- No independent validation cohort was possible, so the reproduced diagnostic performance is not externally validated
- All patients presented with gallbladder conditions, so the primary cause for cholecystectomy may impact the metabolite signatures and hence the achievable AUC

## Evidence

- [supplementary] ROC curve axes: "Panels A-E are showing ROC curves with false positive rates (i.e. 1-specificity) and true positive rates (i.e. sensitivity) on the x- and y-axes respectively"
- [results] reported AUC thresholds: [evidence span withheld — anchor status: not_found]
- [abstract] discrimination claim: "We were able to significantly discriminate Salmonella carriage samples from non-carriage control samples."
- [supplementary] serovar separation: "the separation of S. Typhi carriage samples from the S. Paratyphi A carriage samples along the second component (p = 0.0031)"
