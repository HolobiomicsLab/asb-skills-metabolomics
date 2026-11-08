---
name: roc-curve-auc-evaluation
description: Use when a supervised classification model (e.g. OPLS-DA in SIMCA) produces
  cross-validated scores that must be converted into ROC curves and AUC values with
  bootstrap confidence intervals to report diagnostic performance.
when_to_use_negative:
- The model outputs are unsupervised PCA scores with no class labels — ROC/AUC requires
  labeled pairwise comparisons
- The task is comparing acute enteric fever profiles rather than classifying carriage
  vs non-carriage samples
- The goal is to report chemometric separation significance (e.g. CV-ANOVA p = 0.0031)
  rather than classifier diagnostic performance
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0219
tools:
- name: MATLAB
  role: Constructs ROC curves from cross-validated OPLS-DA scores and computes AUC
    values (R2014b)
- name: SIMCA
  role: Fits the two-class OPLS-DA models whose cross-validated scores feed the ROC
    analysis (version 14, Umetrics)
- name: GCxGC-TOFMS
  role: Instrument that generated the 195-metabolite plasma matrix underlying the
    classification models
provenance:
  source_task_ids:
  - task_002
  - task_003
  source_papers:
  - doi: 10.1371/journal.pntd.0006215
    title: Diagnostic metabolite biomarkers of chronic typhoid carriage
  source_content_hash: sha256:a6fffb57bde203387722862c7efd447bbf20a420cecc171b37c16dbbb0c044f0
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - GCxGC-TOFMS
  - MATLAB
  - Matlab
  - SIMCA
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0219
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# roc-curve-auc-evaluation

## Summary

Builds and evaluates ROC curves from cross-validated OPLS-DA model scores of GCxGC-TOFMS plasma metabolite data to quantify diagnostic performance (AUC) for distinguishing Salmonella carriage from non-carriage controls, with 95% confidence intervals from 1,000-percentile bootstraps.

## When to use

Use when a supervised classification model (e.g. OPLS-DA in SIMCA) produces cross-validated scores that must be converted into ROC curves and AUC values with bootstrap confidence intervals to report diagnostic performance.

## When NOT to use

- The model outputs are unsupervised PCA scores with no class labels — ROC/AUC requires labeled pairwise comparisons
- The task is comparing acute enteric fever profiles rather than classifying carriage vs non-carriage samples
- The goal is to report chemometric separation significance (e.g. CV-ANOVA p = 0.0031) rather than classifier diagnostic performance

## Inputs

- Cross-validated OPLS-DA scores (SIMCA v14, Umetrics; 7-fold CV) for pairwise models on the 195-metabolite × 37-sample GCxGC-TOFMS plasma matrix (12 S. Typhi carriers, 5 S. Paratyphi A carriers, 20 non-carriage controls)
- Class labels for the pairwise comparisons (carriage vs control, or S. Typhi vs S. Paratyphi A carriage)
- Five-metabolite subset: glutaric acid, hexanoic acid, Unknown_087, Unknown_118, Unknown_399 (UV-scaled)

## Outputs

- ROC curves with false positive rate (1-specificity) vs true positive rate (sensitivity), as in Fig 2A-2D and Fig 3D
- AUC values with bootstrap-derived 95% confidence intervals
- Structured results table (auc_results.csv)

## How to apply

Load the cross-validated scores from each pairwise OPLS-DA model (e.g. combined carriage vs non-carriage controls, S. Typhi vs controls, S. Paratyphi A vs controls, S. Typhi vs S. Paratyphi A carriage) and plot ROC curves in MATLAB (R2014b) with false positive rate (1-specificity) on the x-axis and true positive rate (sensitivity) on the y-axis. Compute the area under each curve (AUC), where 0.5 represents a random classifier and 1 a perfect classifier. Estimate 95% confidence intervals for each AUC with the bootstrap percentile resampling method using 1,000 bootstraps. Validate against the paper's acceptance criteria: AUC >= 0.95 for carriage-vs-control models and AUC = 0.833 for the weaker S. Typhi vs S. Paratyphi A model; a five-metabolite panel (glutaric acid, hexanoic acid, Unknown_087, Unknown_118, Unknown_399) should reach AUC ≈ 0.935. Record AUC values and confidence intervals in a structured table such as auc_results.csv.

## Related tools

- **MATLAB** (Constructs ROC curves from cross-validated OPLS-DA scores and computes AUC values (R2014b))
- **SIMCA** (Fits the two-class OPLS-DA models whose cross-validated scores feed the ROC analysis (version 14, Umetrics))
- **GCxGC-TOFMS** (Instrument that generated the 195-metabolite plasma matrix underlying the classification models)

## Evaluation signals

- All three carriage-vs-control ROC curves must yield AUC >= 0.95 (Fig 2A-2C)
- The S. Typhi vs S. Paratyphi A ROC curve must reproduce AUC = 0.833 (Fig 2D)
- The five-metabolite OPLS-DA model must yield AUC ≈ 0.935, exceeding every individual metabolite (max individual AUC = 0.876, S2 Fig)
- Each reproduced AUC must fall within its bootstrap-derived 95% confidence interval (1,000 percentile bootstraps)
- auc_results.csv must contain one AUC plus 95% CI per pairwise model, matching reported values

## Limitations

- Small sample size (37 samples) — the major limitation of the study; AUC estimates have wide uncertainty
- No independent validation cohort was available, so AUCs are cross-validated within a single dataset
- All patients had underlying gallbladder conditions, so AUCs may not generalize to non-cholecystectomy populations; further validation in alternative endemic enteric fever settings is required

## Evidence

- [results] AUC 0.95 carriage vs controls: [evidence span withheld — anchor status: not_found]
- [supplementary] ROC axes definition: "Panels A-E are showing ROC curves with false positive rates (i.e. 1-specificity) and true positive rates (i.e. sensitivity) on the x- and y-axes respectively"
- [abstract] Five-metabolite model AUC: "Salmonella carriers could be significantly distinguished from non-carriage controls using only five metabolites"
- [discussion] Small sample size limitation: "The major limitation of this study was the small sample size, which was difficult to overcome given the ethical, surgical, and diagnostic issues of identifying and confirming invasive Salmonella"
- [supplementary] Metabolite matrix and OPLS-DA modeling: "models based on 195 metabolites generated from GCxGC-TOFMS analysis of plasma samples from patients in Nepal undergoing cholecystectomy"
