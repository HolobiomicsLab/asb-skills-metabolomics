---
name: binary-classification-metric-reporting
description: Use when a binary-classification MB-PLS model (e.g., encoded outcome
  such as Female=1/Male=0 in alz_metadata.csv) has been fitted to multiassay LC-MS
  feature blocks and its predictive performance must be quantified on an independent
  test set or via repeated resampling.
when_to_use_negative:
- The outcome is continuous — use evaluate_regression_model instead of evaluate_class_model
  for regression tasks
- The number of latent variables has not yet been selected via estimate_lv k-fold
  cross-validation (metric='auc'); premature evaluation reflects an arbitrary model
  complexity
- The goal is feature prioritization or structural clustering (mb_vip, mb_vip_permtest,
  MamsiStructSearch) rather than model performance reporting
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: mamsi
  role: Provides MamsiPls.evaluate_class_model and MamsiPls.montecarlo_cv for classification
    metric computation on test and resampled data
  repo: https://github.com/kopeckylukas/py-mamsi
- name: mbpls
  role: mbpls==1.0.4 underlies the MamsiPls MB-PLS decomposition being evaluated
  repo: https://github.com/kopeckylukas/py-mamsi
- name: scikit-learn
  role: train_test_split (random_state=42) for the 90:10 held-out split and underlying
    k-fold cross-validation used for LV selection
- name: pandas
  role: Loading assay feature tables and writing aggregated CV metrics (e.g., mbpls_cv_metrics.csv)
provenance:
  source_task_ids:
  - task_003
  - task_008
  source_papers:
  - doi: 10.1021/acs.analchem.5c01327
    title: 'MAMSI: Integration of Multiassay Liquid Chromatography–Mass Spectrometry
      Metabolomics Data Using Multiview Machine Learning'
  source_content_hash: sha256:90285a9afa75ea6ad50c16751e66a60fdb212d7392d5338201bbb908498dd87d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/kopeckylukas/py-mamsi
  tools:
  - Python
  - joblib
  - mamsi
  - matplotlib
  - mbpls
  - networkx
  - numpy
  - pandas
  - pyvis
  - scikit-learn
  - scipy
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 18
---

# binary-classification-metric-reporting

## Summary

Evaluates a fitted MB-PLS binary classifier on a held-out test split and via Monte Carlo cross-validation, reporting accuracy, recall, specificity, F1, precision and AUC. In the MAMSI workflow this establishes that the multiblock model outperforms single-assay baselines and is not overfitted.

## When to use

Use when a binary-classification MB-PLS model (e.g., encoded outcome such as Female=1/Male=0 in alz_metadata.csv) has been fitted to multiassay LC-MS feature blocks and its predictive performance must be quantified on an independent test set or via repeated resampling.

## When NOT to use

- The outcome is continuous — use evaluate_regression_model instead of evaluate_class_model for regression tasks
- The number of latent variables has not yet been selected via estimate_lv k-fold cross-validation (metric='auc'); premature evaluation reflects an arbitrary model complexity
- The goal is feature prioritization or structural clustering (mb_vip, mb_vip_permtest, MamsiStructSearch) rather than model performance reporting

## Inputs

- Sample-aligned multiassay LC-MS feature tables (e.g., alz_hpos.csv, alz_lpos.csv, alz_lneg.csv)
- Binary outcome metadata numerically encoded (e.g., Gender: 1=Female, 0=Male from alz_metadata.csv)
- Training/testing split (e.g., 90:10, random_state=42)
- Fitted MamsiPls model with NIPALS algorithm, standardize=True, max_tol=1e-14, and a selected number of LVs (e.g., 6)

## Outputs

- Held-out test-set classification metrics: accuracy, recall, specificity, F1 score, AUC
- Per-repeat MCCV train and test metrics (AUC, precision, recall, specificity, F1, accuracy)
- Aggregated CV metrics table (e.g., mbpls_cv_metrics.csv)

## How to apply

Split the sample-aligned assay blocks 90:10 with scikit-learn train_test_split (random_state=42), keeping the other blocks (e.g., LPOS, LNEG) indexed to the first block (e.g., HPOS). Select the number of latent variables first via MamsiPls.estimate_lv with k-fold cross-validation and metric='auc' (the AddNeuroMed reference model used 6 LVs), then call evaluate_class_model([hpos_test, lpos_test, lneg_test], y_test) on the held-out 10% split to obtain accuracy, recall, specificity, F1 and AUC. For robustness, run MamsiPls.montecarlo_cv on the training data with repeats=1000, test_size=0.2, classification=True and return_train=True to collect per-repeat train and test metrics (AUC, precision, recall, specificity, F1, accuracy). Aggregate the k-fold and MCCV metrics (e.g., into mbpls_cv_metrics.csv) and judge correctness by consistency: the mean MCCV validation AUC should agree with the LV-estimate cross-validated performance and with the paper's held-out test AUC of 0.933 (accuracy 0.966, recall 1.0, specificity 0.943, F1 0.971) for the final 6-LV model. Suspiciously large train-vs-test gaps or metrics far above the reference indicate overfitting.

## Related tools

- **mamsi** (Provides MamsiPls.evaluate_class_model and MamsiPls.montecarlo_cv for classification metric computation on test and resampled data) — https://github.com/kopeckylukas/py-mamsi
- **mbpls** (mbpls==1.0.4 underlies the MamsiPls MB-PLS decomposition being evaluated) — https://github.com/kopeckylukas/py-mamsi
- **scikit-learn** (train_test_split (random_state=42) for the 90:10 held-out split and underlying k-fold cross-validation used for LV selection)
- **pandas** (Loading assay feature tables and writing aggregated CV metrics (e.g., mbpls_cv_metrics.csv))

## Examples

```
predicted = mamsipls.evaluate_class_model([hpos_test, lpos_test, lneg_test], y_test.array)
```

## Evaluation signals

- The reported metric set contains all five classification metrics (accuracy, recall, specificity, F1, AUC); the reference table shows accuracy 0.966, recall 1.0, specificity 0.943, F1 0.971, AUC 0.933
- Mean MCCV validation AUC is consistent with the estimate_lv k-fold cross-validated AUC at the chosen number of LVs (6 in the reference)
- Train metrics from montecarlo_cv (return_train=True) are not substantially higher than test metrics, indicating a non-overfitted model
- Test evaluation is performed only on the untouched 10% split that was excluded from both LV selection and fitting
- Metrics are computed with the outcome as a numeric binary variable (1=Female, 0=Male), since the PLS algorithm requires a numeric response

## Limitations

- A single 90:10 held-out split yields a small test set with high-variance metrics; the article relies on MCCV with 1000 random 20% test splits for robustness
- Reported metrics depend on the chosen number of latent variables; the reference performance (AUC 0.933) corresponds specifically to the 6-LV model
- Metrics assume a numeric binary outcome encoding; non-numeric class labels must be converted beforehand
- For grouped or repeated-measure designs, an ungrouped train_test_split can leak related samples; use group_train_test_split in that case

## Evidence

- [results] Final classification model achieved accuracy 0.966, recall 1.0, specificity 0.943, F1 0.971, AUC 0.933 on test set: "| Accuracy            | 0.966 |
| Recall              | 1.0   |
| Specificity         | 0.943 |
| F<sub>1</sub> Score | 0.971 |
| AUC                 | 0.933 |"
- [methods] evaluate_class_model evaluates the classification model on a testing dataset: "Evaluate classification MB-PLS model using a **testing** dataset."
- [methods] montecarlo_cv evaluates the MB-PLS model using MCCV: "Evaluate MB-PLS model using Monte Carlo Cross-Validation (MCCV)."
- [readme] README quickstart shows estimate_lv with metric='auc' before VIP/permutation analysis: [evidence span withheld — anchor status: external_doc]
- [readme] The PLS algorithm requires the response variable to be numeric; outcome encoded as 1 for female and 0 for male: "The PLS algorithm requires the response variable to be numeric. 
# We will encode the outcome "Gender" (Biological Sex) as 1 for female and 0 for male subjects."
