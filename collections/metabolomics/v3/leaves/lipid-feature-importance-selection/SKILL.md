---
name: lipid-feature-importance-selection
description: Use when LipidIN-annotated lipid feature tables from patient cohorts
  (e.g., breast cancer vs. healthy plasma LC-MS lipidomics, Supplementary Data 9)
  are available and a small diagnostic panel with cross-cohort generalization is needed.
when_to_use_negative:
- Input is raw LC-MS/MS data not yet processed by LipidIN annotation/quantification/QC
  (CV / PCA / PLS-DA); feature importance selection requires an annotated feature
  table first
- Only one cohort is available and the goal includes estimating cross-cohort generalization,
  which this skill's two-cohort validation design addresses
- The task is lipid annotation itself (MS2 library search against the 168.6-million-entry
  theoretical library) rather than downstream biomarker modeling
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_0091
tools:
- name: LipidIN
  role: Upstream annotation framework producing the annotated lipid feature table
    from which the biomarker panel is selected
  repo: github.com/LinShuhaiLAB/LipidIN
- name: pandas
  role: Loads the Supplementary Data 9 feature table, keeping cohort labels and the
    10 featured lipids
- name: scikit-learn
  role: Random-forest classifier for Gini-importance lipid ranking; roc_curve and
    roc_auc_score for ROC/AUC computation
- name: LightGBM
  role: Classifier trained on the 10 selected lipids with learning_rate=0.01 and max_depth=5
    to produce diagnostic probabilities
- name: matplotlib
  role: Plots the ROC curves of both cohorts in one figure mirroring Fig. 5c and exports
    figure files
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-025-59683-5
    title: 'LipidIN: a comprehensive repository for flash platform-independent annotation
      and reverse lipidomics'
  source_content_hash: sha256:cbb6b554532626ef30844c0835c8d3569cf25d377114e84c771a2bc0b669709a
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - LightGBM
  - LipidIN
  - matplotlib
  - pandas
  - scikit-learn
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 15
---

# lipid-feature-importance-selection

## Summary

Selects a compact disease-discriminating lipid biomarker panel by ranking LipidIN-annotated lipid features with random-forest Gini importance, then validates discrimination with a LightGBM classifier across independent clinical cohorts.

## When to use

Use when LipidIN-annotated lipid feature tables from patient cohorts (e.g., breast cancer vs. healthy plasma LC-MS lipidomics, Supplementary Data 9) are available and a small diagnostic panel with cross-cohort generalization is needed.

## When NOT to use

- Input is raw LC-MS/MS data not yet processed by LipidIN annotation/quantification/QC (CV / PCA / PLS-DA); feature importance selection requires an annotated feature table first
- Only one cohort is available and the goal includes estimating cross-cohort generalization, which this skill's two-cohort validation design addresses
- The task is lipid annotation itself (MS2 library search against the 168.6-million-entry theoretical library) rather than downstream biomarker modeling

## Inputs

- LipidIN-annotated lipid feature table (Supplementary Data 9) with cohort labels for two breast cancer cohorts
- Cohort 1 training matrix (1393 samples) with case/healthy class labels
- Independent cohort 2 matrix (333 samples) with class labels

## Outputs

- Ranked lipid importance list with the top 10 featured lipids selected by random-forest Gini importance
- LightGBM classification accuracies per cohort (96.93% cohort 1; 79.61% cohort 2)
- ROC curves and AUC values per cohort computed from LightGBM predicted probabilities
- Exported figure files with both cohorts' ROC curves mirroring Fig. 5c

## How to apply

Load the annotated lipid feature table (e.g., Supplementary Data 9) with pandas, keeping cohort labels and candidate lipid features. Train a random-forest classifier on cohort 1 (1393 samples), rank lipids by Gini importance, and retain the top 10 featured lipids; a LightGBM classifier on these 10 lipids with learning_rate=0.01 and max_depth=5 should reach about 96.93% accuracy on cohort 1. Repeat the random-forest selection and LightGBM training (learning_rate=0.01, max_depth=5) on the independent cohort 2 (333 samples), where the reported accuracy is 79.61%, to test generalization. Compute ROC curves and AUC with scikit-learn's roc_curve and roc_auc_score on LightGBM predicted probabilities, and plot both cohorts' curves in one figure mirroring Fig. 5c with matplotlib.

## Related tools

- **LipidIN** (Upstream annotation framework producing the annotated lipid feature table from which the biomarker panel is selected) — github.com/LinShuhaiLAB/LipidIN
- **pandas** (Loads the Supplementary Data 9 feature table, keeping cohort labels and the 10 featured lipids)
- **scikit-learn** (Random-forest classifier for Gini-importance lipid ranking; roc_curve and roc_auc_score for ROC/AUC computation)
- **LightGBM** (Classifier trained on the 10 selected lipids with learning_rate=0.01 and max_depth=5 to produce diagnostic probabilities)
- **matplotlib** (Plots the ROC curves of both cohorts in one figure mirroring Fig. 5c and exports figure files)

## Examples

```
from lightgbm import LGBMClassifier; model = LGBMClassifier(learning_rate=0.01, max_depth=5); model.fit(X1, y1)
```

## Evaluation signals

- Cohort 1 LightGBM accuracy is within ±2 percentage points of the reported 96.93%
- Cohort 2 LightGBM accuracy is within ±2 percentage points of the reported 79.61%
- ROC curves for both cohorts are exported as figure files and were computed with roc_curve/roc_auc_score on LightGBM predicted probabilities
- The selected panel contains exactly 10 featured lipids ranked by random-forest Gini importance on cohort 1

## Limitations

- Panel accuracy drops from 96.93% on cohort 1 to 79.61% on the independent cohort 2, so generalization across cohorts/batches is substantially weaker than in-cohort performance
- Selection is tied to random-forest Gini importance on cohort 1 (1393 samples); feature rankings may not transfer to other cohorts or platforms
- LipidIN's category-intelligence model carries a 5.7% estimated false discovery rate across 8923 lipids, so misannotated features can propagate into the biomarker panel

## Evidence

- [abstract] We further demonstrate the utility of LipidIN in multiple tasks for lipid annotation and biomarker discovery in clinical cohorts.: "We further demonstrate the utility of LipidIN in multiple tasks for lipid annotation and biomarker discovery in clinical cohorts."
- [methods] Train a random-forest classifier on cohort 1 (1393 samples) to rank lipids by Gini importance and select the top 10 featured lipids, matching the reported 96.93% accuracy.: [evidence span withheld — anchor status: not_found]
- [methods] Train a LightGBM classifier on the 10 selected lipids with learning_rate=0.01 and max_depth=5, evaluating accuracy on cohort 1.: [evidence span withheld — anchor status: not_found]
- [methods] check that cohort 1 accuracy is within ±2 percentage points of the reported 96.93% and cohort 2 within ±2 pp of 79.61%: [evidence span withheld — anchor status: not_found]
