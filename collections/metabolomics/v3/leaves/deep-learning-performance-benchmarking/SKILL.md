---
name: deep-learning-performance-benchmarking
description: Use when validating a deep-learning classifier on bioactivity-labeled
  metabolomics samples and comparing its metrics (AUC, accuracy, F1) against classical
  ML baselines across folds.
when_to_use_negative:
- Input data is already an aligned feature table intended for Random Forest, PCA,
  or OPLS-DA baselines rather than the CNN image pipeline
- Samples have normalized bioactivity values between 0.4 and 0.6 (the article's ambiguous
  middle zone is excluded from either class)
- Only a single train/test split is available without enough samples for stratified
  5-fold cross-validation
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_3172
tools:
- name: TensorFlow
  role: Backend for loading the trained MS2toImg-CNN model and running sigmoid probability
    predictions during cross-validation
  repo: https://github.com/jsehhs/MS2toImg-CNN
- name: scikit-learn
  role: Provides StratifiedKFold for 5-fold stratified cross-validation and metric
    computation (ROC-AUC, accuracy, precision, recall, F1)
- name: Keras 2.14
  role: Model API used to implement and evaluate the CNN classifier
  repo: https://github.com/jsehhs/MS2toImg-CNN
provenance:
  source_task_ids:
  - task_003
  source_content_hash: sha256:1c679ca871ec340b5ef0c3c798adc11ddec8735ed222799791c92256e157d354
  source_papers:
  - doi: 10.48550/arxiv.2510.09716
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/jsehhs/MS2toImg-CNN
  tools:
  - Keras 2.14
  - TensorFlow
  - scikit-learn
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6428571428571429
n_claims: 14
---

# deep-learning-performance-benchmarking

## Summary

Rigorous evaluation protocol for a CNN bioactivity classifier (MS2toImg-CNN) using stratified 5-fold cross-validation on grayscale molecular fingerprint images, reporting accuracy, precision, recall, F1, and ROC-AUC against conventional ML baselines (Random Forest, PCA, OPLS-DA).

## When to use

Use when validating a deep-learning classifier on bioactivity-labeled metabolomics samples and comparing its metrics (AUC, accuracy, F1) against classical ML baselines across folds.

## When NOT to use

- Input data is already an aligned feature table intended for Random Forest, PCA, or OPLS-DA baselines rather than the CNN image pipeline
- Samples have normalized bioactivity values between 0.4 and 0.6 (the article's ambiguous middle zone is excluded from either class)
- Only a single train/test split is available without enough samples for stratified 5-fold cross-validation

## Inputs

- Trained MS2toImg-CNN model weights (TensorFlow/Keras 2.14)
- Preprocessed 2D grayscale molecular-fingerprint images of wild soybean LC-MS/MS samples (e.g., 75x70, 150x140, or 300x280 pixel versions)
- Per-sample binary bioactivity labels (DPPH normalized < 0.4 = low, > 0.6 = high)
- Baseline model predictions (Random Forest, PCA, OPLS-DA) for comparison

## Outputs

- ROC curve and ROC-AUC score per bioassay (target ~0.96 for DPPH)
- Per-fold and accumulated test-set confusion matrices (balanced, minimal false positives/negatives)
- Metrics table (accuracy, precision, recall, F1, AUC) saved as CSV
- Metric box plots across 5 folds (analogous to Figure 5)

## How to apply

Load the trained MS2toImg-CNN model and the preprocessed 2D grayscale molecular-fingerprint images of wild soybean (Glycine soja) samples with their DPPH high/low activity labels (normalized DPPH < 0.4 = low, > 0.6 = high). Run 5-fold stratified cross-validation with scikit-learn's StratifiedKFold over the image dataset, predicting class probabilities from the CNN's sigmoid output layer (TensorFlow/Keras 2.14 on Python 3.10). Compute ROC-AUC from the probability scores and generate the ROC curve, then accumulate test-set predictions across all folds to compute accuracy, precision, recall, and F1 (as shown in the article's Figure 5 box plots). Tabulate all five metrics into a CSV results table. Validate against the article's reported values: DPPH AUC of 0.96 with accuracy/recall/F1 > 0.90, which must exceed the Random Forest baseline AUC of 0.817.

## Related tools

- **TensorFlow** (Backend for loading the trained MS2toImg-CNN model and running sigmoid probability predictions during cross-validation) — https://github.com/jsehhs/MS2toImg-CNN
- **scikit-learn** (Provides StratifiedKFold for 5-fold stratified cross-validation and metric computation (ROC-AUC, accuracy, precision, recall, F1))
- **Keras 2.14** (Model API used to implement and evaluate the CNN classifier) — https://github.com/jsehhs/MS2toImg-CNN

## Examples

```
python main.py 75 70
```

## Evaluation signals

- DPPH task ROC-AUC reaches approximately 0.96, matching or exceeding the article's reported value
- Accuracy, precision, recall, and F1 all exceed 0.90 (consistent with Figure 3B and Figure 5)
- 5-fold confusion matrices are balanced and stable with minimal false positives and false negatives across folds
- CNN AUC clearly outperforms the Random Forest baseline AUC of 0.817
- All five metrics (accuracy, precision, recall, F1, AUC) are tabulated and saved as a CSV results table

## Limitations

- Numeric performance values referenced in Table 2 (elastase inhibition, TFC, TPC tasks) are not provided in the article text, limiting direct metric verification for non-DPPH assays
- Hyperparameters are only in Supplementary Table 1, not in the main text, so exact training conditions for reproducing the metrics are underspecified
- The repository is under review with redistribution/use restrictions until formal publication
- Model weights have no stated license or availability guarantee, so metric reproduction depends on retraining via the provided code

## Evidence

- [results] On the DPPH antioxidant task, the MS2toImg-CNN model achieved an AUC of 0.96: [evidence span withheld — anchor status: not_found]
- [methods] Stratified fivefold CV protocol: [evidence span withheld — anchor status: not_found]
