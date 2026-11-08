---
name: lightgbm-classifier-tuning
description: Use when a lipid (or other small-feature) biomarker panel must be validated
  as a classifier across independent clinical cohorts, with accuracy and ROC curve
  reporting.
when_to_use_negative:
- Input is already a trained model or prediction output — no selection or fitting
  needed.
- The feature set is large and unfiltered — random-forest Gini-based panel selection
  should be applied first.
- No independent cohort is available for external validation — only in-sample accuracy
  can be reported.
edam_operation: http://edamontology.org/operation_3936
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_3360
tools:
- name: LightGBM
  role: gradient-boosted classifier trained on the selected lipid panel with learning_rate=0.01,
    max_depth=5
- name: scikit-learn
  role: random forest for Gini-importance feature selection, plus roc_curve and roc_auc_score
    for evaluation
- name: pandas
  role: loading and reshaping the annotated lipid feature table (Supplementary Data
    9)
- name: matplotlib
  role: plotting the ROC curves for both cohorts in one figure (Fig. 5c)
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-025-59683-5
    title: 'LipidIN: a comprehensive repository for flash platform-independent annotation
      and reverse lipidomics'
  source_content_hash: sha256:1cfb47c385e706d3738717c3597644fbd1e3e8001a001de5150cdac3a4892b77
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - LightGBM
  - LipidIN
  - matplotlib
  - pandas
  - scikit-learn
  edam_operation: http://edamontology.org/operation_3936
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_3360
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# lightgbm-classifier-tuning

## Summary

Train a LightGBM classifier on a small set of lipid features (e.g. the 10 featured lipids selected by random-forest Gini importance) to classify clinical cohorts, tuned with learning_rate=0.01 and max_depth=5, and evaluate via accuracy and ROC/AUC.

## When to use

Use when a lipid (or other small-feature) biomarker panel must be validated as a classifier across independent clinical cohorts, with accuracy and ROC curve reporting.

## When NOT to use

- Input is already a trained model or prediction output — no selection or fitting needed.
- The feature set is large and unfiltered — random-forest Gini-based panel selection should be applied first.
- No independent cohort is available for external validation — only in-sample accuracy can be reported.

## Inputs

- annotated lipid feature table (Supplementary Data 9) with cohort labels (cohort 1: 1393 samples; cohort 2: 333 samples)
- the 10 random-forest-selected featured lipids

## Outputs

- classifier accuracy per cohort (96.93% cohort 1; 79.61% cohort 2)
- ROC curves and AUC values per cohort
- a single ROC figure combining both cohorts (mirroring Fig. 5c)

## How to apply

Load the annotated lipid feature table (e.g. Supplementary Data 9) keeping cohort labels and the featured lipids. First run a random-forest classifier on the training cohort to rank lipids by Gini importance and select the top panel (here the 10 featured lipids). Train LightGBM with learning_rate=0.01 and max_depth=5 on those features, reporting accuracy on cohort 1 (1393 samples) and then repeat the same selection-and-training on an independent cohort 2 (333 samples) to test generalization. Compute ROC curves and AUC with scikit-learn's roc_curve and roc_auc_score on the LightGBM predicted probabilities, and plot both cohorts' ROC curves together (mirroring Fig. 5c).

## Related tools

- **LightGBM** (gradient-boosted classifier trained on the selected lipid panel with learning_rate=0.01, max_depth=5)
- **scikit-learn** (random forest for Gini-importance feature selection, plus roc_curve and roc_auc_score for evaluation)
- **pandas** (loading and reshaping the annotated lipid feature table (Supplementary Data 9))
- **matplotlib** (plotting the ROC curves for both cohorts in one figure (Fig. 5c))

## Examples

```
import lightgbm as lgb; model = lgb.LGBMClassifier(learning_rate=0.01, max_depth=5); model.fit(X_train, y_train); acc = model.score(X_test, y_test)
```

## Evaluation signals

- Cohort-1 accuracy within ±2 percentage points of the reported 96.93%.
- Cohort-2 accuracy within ±2 percentage points of the reported 79.61%.
- ROC curves for both cohorts exported as figure files (Fig. 5c style).
- The same 10-featured-lipid panel is used in both cohorts to confirm transferability.
- Gini importance ranking from the random forest selects exactly the top-10 panel used downstream.

## Limitations

- Only two cohorts (1393 and 333 samples) were used; generalization beyond them is untested.
- Panel selection via random-forest Gini importance is a heuristic — alternative selection schemes may yield different panels.
- Estimated recall/coverage gains of the underlying annotation (+20% for WMYn) may affect which lipids enter the feature table.

## Evidence

- [results] Train a LightGBM classifier on the 10 selected lipids with learning_rate=0.01 and max_depth=5, evaluating accuracy on cohort 1.: [evidence span withheld — anchor status: not_found]
- [results] A LightGBM classifier built on 10 random-forest-selected featured lipids reached 96.93% accuracy on cohort 1 (1393 samples) and 79.61% accuracy on the independent second cohort of 333 cases.: [evidence span withheld — anchor status: not_found]
- [methods] Compute ROC curves and AUC for each cohort using scikit-learn's roc_curve and roc_auc_score on LightGBM predicted probabilities.: [evidence span withheld — anchor status: not_found]
- [methods] Check that cohort 1 accuracy is within ±2 percentage points of the reported 96.93% and cohort 2 within ±2 pp of 79.61%.: [evidence span withheld — anchor status: not_found]
