---
name: random-forest-classifier-training
description: Use when a labeled, high-dimensional lipid feature table (e.g., annotated
  by LipidIN, such as Supplementary Data 9 breast cancer cohorts) needs feature ranking
  and biomarker panel selection for patient-vs-healthy classification.
when_to_use_negative:
- Input is already a small, pre-selected lipid panel and the goal is only model fitting
  — no Gini-importance ranking step is needed.
- The task is spectral annotation or library search (e.g., LipidIN MS2 library search,
  Flash Entropy Search) rather than supervised classification of a feature table.
- Samples lack disease/healthy class labels, so no supervised random-forest training
  is possible.
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: scikit-learn
  role: Random-forest training and Gini-importance ranking of lipids; roc_curve and
    roc_auc_score for ROC/AUC computation
- name: LightGBM
  role: Final classifier trained on the 10 RF-selected featured lipids (learning_rate=0.01,
    max_depth=5), producing predicted probabilities for ROC analysis
- name: pandas
  role: Loading the annotated lipid feature table of the two breast cancer cohorts
    while keeping cohort labels and the 10 featured lipids
- name: matplotlib
  role: Plotting the ROC curves for both cohorts in one figure mirroring Fig. 5c
- name: LipidIN
  role: Upstream source of the annotated lipid feature table used as classifier input
  repo: github.com/LinShuhaiLAB/LipidIN
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-025-59683-5
    title: 'LipidIN: a comprehensive repository for flash platform-independent annotation
      and reverse lipidomics'
  source_content_hash: sha256:8c73bc2e626c05747a91bf096ec623ad73b43267544cd47d1cf4faf76f34bf77
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
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 18
---

# random-forest-classifier-training

## Summary

Trains a random-forest classifier on an annotated lipid feature table to rank lipids by Gini importance, selecting a small biomarker panel (top 10 featured lipids) that downstream classifiers such as LightGBM use for disease-vs-control prediction across independent clinical cohorts.

## When to use

Use when a labeled, high-dimensional lipid feature table (e.g., annotated by LipidIN, such as Supplementary Data 9 breast cancer cohorts) needs feature ranking and biomarker panel selection for patient-vs-healthy classification.

## When NOT to use

- Input is already a small, pre-selected lipid panel and the goal is only model fitting — no Gini-importance ranking step is needed.
- The task is spectral annotation or library search (e.g., LipidIN MS2 library search, Flash Entropy Search) rather than supervised classification of a feature table.
- Samples lack disease/healthy class labels, so no supervised random-forest training is possible.

## Inputs

- Annotated lipid feature table with cohort labels (e.g., Supplementary Data 9 from LipidIN-annotated breast cancer cohorts 1 and 2)
- Binary class labels (breast cancer patient vs. healthy subject) per sample
- Cohort split indicator (cohort 1, n=1393; independent cohort 2, n=333)

## Outputs

- Ranked lipid list with random-forest Gini importance scores and the top-10 featured-lipid panel
- Trained LightGBM classifier (learning_rate=0.01, max_depth=5) per cohort with reported accuracies (96.93% cohort 1; 79.61% cohort 2)
- ROC curves and AUC values per cohort, plotted together (Fig. 5c-style) and exported as figure files

## How to apply

Load the annotated lipid feature table (e.g., Supplementary Data 9) with pandas, keeping cohort labels and lipid features. Train a random-forest classifier on cohort 1 (1393 breast cancer/healthy samples) with scikit-learn, rank lipids by Gini importance, and select the top 10 featured lipids. Feed only those 10 selected lipids into a LightGBM classifier (learning_rate=0.01, max_depth=5) and record accuracy on cohort 1, then repeat random-forest selection plus LightGBM training on the independent cohort 2 (333 samples). Compute ROC curves and AUC from LightGBM predicted probabilities with roc_curve and roc_auc_score, and plot both cohorts' curves in one figure mirroring Fig. 5c with matplotlib. The rationale is that Gini-importance ranking from the RF reduces the ~90%-unannotated, high-redundancy lipid feature space to a compact, credible biomarker panel before model training.

## Related tools

- **scikit-learn** (Random-forest training and Gini-importance ranking of lipids; roc_curve and roc_auc_score for ROC/AUC computation)
- **LightGBM** (Final classifier trained on the 10 RF-selected featured lipids (learning_rate=0.01, max_depth=5), producing predicted probabilities for ROC analysis)
- **pandas** (Loading the annotated lipid feature table of the two breast cancer cohorts while keeping cohort labels and the 10 featured lipids)
- **matplotlib** (Plotting the ROC curves for both cohorts in one figure mirroring Fig. 5c)
- **LipidIN** (Upstream source of the annotated lipid feature table used as classifier input) — github.com/LinShuhaiLAB/LipidIN

## Examples

```
from sklearn.ensemble import RandomForestClassifier; rf = RandomForestClassifier().fit(X_c1, y_c1); top10 = X_c1.columns[rf.feature_importances_.argsort()[-10:]]
```

## Evaluation signals

- Cohort 1 LightGBM accuracy is within ±2 percentage points of the reported 96.93% (1393 samples).
- Independent cohort 2 accuracy is within ±2 percentage points of the reported 79.61% (333 cases).
- The RF Gini-importance ranking selects exactly 10 featured lipids, and downstream accuracy is computed only on those lipids.
- ROC curves for both cohorts are generated from LightGBM predicted probabilities via roc_curve/roc_auc_score and exported as figure files.
- The accuracy drop between cohort 1 and cohort 2 (≈17 percentage points) is consistent with a genuine independent-cohort generalization test.

## Limitations

- Generalization drops substantially from 96.93% on cohort 1 to 79.61% on the independent cohort 2, so within-cohort accuracy overstates panel performance.
- Classifier hyperparameters are fixed (learning_rate=0.01, max_depth=5); no broader tuning or cross-validation regime is specified.
- Results depend on the LipidIN-annotated feature table; annotation false positives (estimated 5.7% FDR across 8923 lipids) can propagate into the selected panel.

## Evidence

- [methods] RF selection and top-10 panel: [evidence span withheld — anchor status: not_found]
- [results] Reported cohort accuracies: [evidence span withheld — anchor status: not_found]
- [methods] Validation tolerance: [evidence span withheld — anchor status: not_found]
- [abstract] Upstream annotation FDR context: "developed using three relative retention time rules, reducing false positive annotations and predicting unannotated lipids with a 5.7% estimated false discovery rate"
