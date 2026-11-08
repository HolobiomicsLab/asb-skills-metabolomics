---
name: cohort-accuracy-benchmarking
description: Use when a candidate lipid biomarker panel must be validated as a case-vs-control
  classifier on a discovery cohort and an independent second cohort, with accuracy/ROC-AUC
  reported per cohort.
when_to_use_negative:
- Input is raw LC-MS data or unannotated features — lipid annotation (LipidIN MS2
  library search) must be completed before feature-table classification
- The goal is annotation-quality benchmarking (false discovery rate, recall) rather
  than clinical case-vs-control prediction accuracy
- Only one cohort is available — the skill's core purpose is assessing generalization
  to an independent second cohort
tools:
- name: LipidIN
  role: produced the annotated lipid feature table and biomarker panel being benchmarked
  repo: github.com/LinShuhaiLAB/LipidIN
- name: pandas
  role: loads the annotated lipid feature table keeping cohort labels and the 10 featured
    lipids
- name: scikit-learn
  role: random-forest Gini-importance lipid selection, roc_curve and roc_auc_score
    for evaluation
- name: LightGBM
  role: classifier on the 10 selected lipids with learning_rate=0.01 and max_depth=5
- name: matplotlib
  role: plots both cohorts' ROC curves in one figure mirroring Fig. 5c
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-025-59683-5
    title: 'LipidIN: a comprehensive repository for flash platform-independent annotation
      and reverse lipidomics'
  source_content_hash: sha256:3148731cb7aab64bfb65c5cf8212ae730b6a6e7bda1f0f7ce6c6dc3b85e1b10f
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
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6923076923076923
n_claims: 13
---

# cohort-accuracy-benchmarking

## Summary

Benchmarks a lipid biomarker panel's classification accuracy across a discovery cohort and an independent validation cohort, mirroring the LipidIN breast-cancer evaluation (LightGBM on random-forest-selected featured lipids). This establishes whether the panel generalizes beyond the training cohort.

## When to use

Use when a candidate lipid biomarker panel must be validated as a case-vs-control classifier on a discovery cohort and an independent second cohort, with accuracy/ROC-AUC reported per cohort.

## When NOT to use

- Input is raw LC-MS data or unannotated features — lipid annotation (LipidIN MS2 library search) must be completed before feature-table classification
- The goal is annotation-quality benchmarking (false discovery rate, recall) rather than clinical case-vs-control prediction accuracy
- Only one cohort is available — the skill's core purpose is assessing generalization to an independent second cohort

## Inputs

- Annotated lipid feature table of two breast cancer cohorts (Supplementary Data 9) with cohort labels and the 10 featured lipids
- Cohort 1 discovery set (1393 samples) with case/healthy labels
- Cohort 2 independent validation set (333 samples) with case/healthy labels

## Outputs

- Per-cohort classification accuracy (target: 96.93% cohort 1, 79.61% cohort 2)
- ROC curves and AUC values computed from LightGBM predicted probabilities per cohort
- Single matplotlib figure with both cohorts' ROC curves mirroring Fig. 5c

## How to apply

Load the annotated lipid feature table (e.g., Supplementary Data 9 from the LipidIN breast-cancer cohorts) with pandas, keeping cohort labels and the featured lipids. On cohort 1 (1393 samples), train a random-forest classifier and rank lipids by Gini importance to select the top 10 featured lipids, reproducing the reported 96.93% accuracy. Train a LightGBM classifier on those 10 lipids with learning_rate=0.01 and max_depth=5, then repeat the identical selection-plus-training procedure on the independent cohort 2 (333 samples), which should yield ~79.61% accuracy. Compute ROC curves and AUC with scikit-learn's roc_curve and roc_auc_score on the LightGBM predicted probabilities, and plot both cohorts' ROC curves in one matplotlib figure mirroring Fig. 5c.

## Related tools

- **LipidIN** (produced the annotated lipid feature table and biomarker panel being benchmarked) — github.com/LinShuhaiLAB/LipidIN
- **pandas** (loads the annotated lipid feature table keeping cohort labels and the 10 featured lipids)
- **scikit-learn** (random-forest Gini-importance lipid selection, roc_curve and roc_auc_score for evaluation)
- **LightGBM** (classifier on the 10 selected lipids with learning_rate=0.01 and max_depth=5)
- **matplotlib** (plots both cohorts' ROC curves in one figure mirroring Fig. 5c)

## Examples

```
from sklearn.ensemble import RandomForestClassifier; from lightgbm import LGBMClassifier; clf = LGBMClassifier(learning_rate=0.01, max_depth=5)
```

## Evaluation signals

- Cohort 1 accuracy must fall within ±2 percentage points of the reported 96.93%
- Cohort 2 accuracy must fall within ±2 percentage points of the reported 79.61%
- Both ROC curves are exported as figure files
- AUC values are computed with scikit-learn's roc_auc_score on LightGBM predicted probabilities, not hard labels
- Random-forest Gini-importance selection recovers the same top-10 featured lipids used for the LightGBM model

## Limitations

- Substantial generalization drop is expected and acceptable: 96.93% on cohort 1 vs 79.61% on the independent cohort 2, so an independent-cohort decline does not invalidate the panel
- Accuracy depends on the upstream annotation quality; ~90% of metabolic features in LC-MS untargeted lipidomics typically remain unannotated without a tool like LipidIN
- The R version of LipidIN is kept for reproducibility only and will no longer be updated; new work should use the Python track

## Evidence

- [abstract] LipidIN in clinical cohorts: "We further demonstrate the utility of LipidIN in multiple tasks for lipid annotation and biomarker discovery in clinical cohorts."
- [results] cohort accuracies: [evidence span withheld — anchor status: not_found]
- [readme] annotation step: "**Annotation** — MS2 library search against the theoretical library (chain compositions + C=C double-bond positions)."
