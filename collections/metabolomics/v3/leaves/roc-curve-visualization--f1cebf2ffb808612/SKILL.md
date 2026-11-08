---
name: roc-curve-visualization
description: Use when a binary lipid-biomarker classifier (e.g., LightGBM on RF-selected
  featured lipids) has produced predicted probabilities for case/control cohorts and
  you must quantify and visualize discrimination per cohort, as in reproducing Fig.
  5c.
when_to_use_negative:
- Input is raw LC-MS/MS spectra or unannotated features — run LipidIN annotation (MS2
  library search against the theoretical library) first; ROC visualization operates
  on a labeled feature table
- The classification task is multiclass without a defined case/control binarization
  — roc_curve and roc_auc_score require binary labels or one-vs-rest probabilities
- No class labels or predicted probabilities exist (e.g., only unsupervised PCA/PLS-DA
  QC output) — ROC curves cannot be computed
edam_topics:
- http://edamontology.org/topic_3524
- http://edamontology.org/topic_0092
tools:
- name: LightGBM
  role: Gradient-boosting classifier (learning_rate=0.01, max_depth=5) trained on
    the 10 RF-selected featured lipids; its predicted probabilities feed the ROC computation
- name: scikit-learn
  role: Provides roc_curve and roc_auc_score used to compute the ROC points and AUC
    from LightGBM predicted probabilities; also supplies the random-forest feature
    selection
- name: matplotlib
  role: Plots the ROC curves of both cohorts in one figure mirroring Fig. 5c and exports
    the figure files
- name: LipidIN
  role: Source of the annotated lipid feature table on which the biomarker panel is
    selected and classified
  repo: github.com/LinShuhaiLAB/LipidIN
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-025-59683-5
    title: 'LipidIN: a comprehensive repository for flash platform-independent annotation
      and reverse lipidomics'
  source_content_hash: sha256:df8e48c7552f04dbb2a8dbe72a65f1ac398ca3d7daf68fc45f5128369ff3483a
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - LightGBM
  - LipidIN
  - matplotlib
  - pandas
  - scikit-learn
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3524
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# roc-curve-visualization

## Summary

Computes and plots ROC curves with AUC for lipid-biomarker classifiers (e.g., LightGBM on random-forest-selected lipids) across two breast cancer cohorts, mirroring Fig. 5c of the LipidIN paper to demonstrate panel discrimination between patients and healthy subjects.

## When to use

Use when a binary lipid-biomarker classifier (e.g., LightGBM on RF-selected featured lipids) has produced predicted probabilities for case/control cohorts and you must quantify and visualize discrimination per cohort, as in reproducing Fig. 5c.

## When NOT to use

- Input is raw LC-MS/MS spectra or unannotated features — run LipidIN annotation (MS2 library search against the theoretical library) first; ROC visualization operates on a labeled feature table
- The classification task is multiclass without a defined case/control binarization — roc_curve and roc_auc_score require binary labels or one-vs-rest probabilities
- No class labels or predicted probabilities exist (e.g., only unsupervised PCA/PLS-DA QC output) — ROC curves cannot be computed

## Inputs

- Annotated lipid feature table (e.g., Supplementary Data 9) with cohort labels (breast cancer vs. healthy) and the 10 random-forest-selected featured lipids
- LightGBM predicted probabilities for cohort 1 (1393 samples) and cohort 2 (333 samples)
- Binary ground-truth class labels per cohort (patient vs. healthy subject)

## Outputs

- ROC curve data (FPR/TPR arrays) per cohort computed via sklearn roc_curve
- AUC score per cohort computed via sklearn roc_auc_score
- A single matplotlib figure with both cohorts' ROC curves, mirroring Fig. 5c, exported as an image file

## How to apply

Start from the annotated lipid feature table (e.g., Supplementary Data 9) with cohort labels and the 10 random-forest-selected featured lipids. Train the LightGBM classifier per cohort (learning_rate=0.01, max_depth=5) — cohort 1 with 1393 samples reaching ~96.93% accuracy, cohort 2 with 333 samples reaching ~79.61% — and extract predicted probabilities. Feed the true labels and LightGBM predicted probabilities into scikit-learn's roc_curve and roc_auc_score to compute FPR/TPR points and AUC for each cohort independently. Plot both ROC curves in a single matplotlib figure mirroring Fig. 5c, and export the figure to a file. Rationale: ROC/AUC on held-out probabilities captures panel discrimination beyond raw accuracy and generalization to the independent cohort validates the LipidIN-annotated biomarker panel.

## Related tools

- **LightGBM** (Gradient-boosting classifier (learning_rate=0.01, max_depth=5) trained on the 10 RF-selected featured lipids; its predicted probabilities feed the ROC computation)
- **scikit-learn** (Provides roc_curve and roc_auc_score used to compute the ROC points and AUC from LightGBM predicted probabilities; also supplies the random-forest feature selection)
- **matplotlib** (Plots the ROC curves of both cohorts in one figure mirroring Fig. 5c and exports the figure files)
- **LipidIN** (Source of the annotated lipid feature table on which the biomarker panel is selected and classified) — github.com/LinShuhaiLAB/LipidIN

## Examples

```
from sklearn.metrics import roc_curve, roc_auc_score; fpr, tpr, _ = roc_curve(y_true, y_prob); auc = roc_auc_score(y_true, y_prob)
```

## Evaluation signals

- Cohort 1 accuracy falls within ±2 percentage points of the reported 96.93% (1393 samples)
- Cohort 2 (independent validation, 333 samples) accuracy falls within ±2 percentage points of the reported 79.61%
- Both ROC curves are exported as figure files
- AUC per cohort is consistent with the reported accuracies (AUC should substantially exceed 0.5 for both cohorts)

## Limitations

- The panel's discrimination dropped from 96.93% on cohort 1 to 79.61% on the independent cohort 2, so cross-cohort generalization is imperfect
- Accuracy tolerances of ±2 percentage points mean exact reproduction depends on RF feature-selection stability and LightGBM hyperparameters (learning_rate=0.01, max_depth=5)
- ROC visualization validates the specific LipidIN-annotated, RF-selected 10-lipid panel; other feature sets are not covered by these reported metrics

## Evidence

- [other] LightGBM classifier reached 96.93% and 79.61% accuracy: [evidence span withheld — anchor status: not_found]
- [other] ROC computation method: [evidence span withheld — anchor status: not_found]
- [other] Figure mirroring instruction: [evidence span withheld — anchor status: not_found]
- [abstract] LipidIN demonstrated in clinical biomarker discovery: "We further demonstrate the utility of LipidIN in multiple tasks for lipid annotation and biomarker discovery in clinical cohorts."
