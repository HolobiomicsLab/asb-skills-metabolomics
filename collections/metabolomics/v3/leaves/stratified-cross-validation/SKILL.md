---
name: stratified-cross-validation
description: Use when training a CNN classifier on grayscale molecular fingerprint
  images with imbalanced or skewed high/low bioactivity labels (normalized DPPH <
  0.4 low, > 0.6 high) and you need stable fold-balanced metrics like AUC.
when_to_use_negative:
- Input is already a feature table from aligned peak picking where the Random Forest
  baseline (AUC 0.817) with standard cross-validation suffices
- The task is regression of continuous bioactivity values rather than high/low class
  discrimination
- Labels have not been binarized (mid-range normalized values 0.4–0.6 included), so
  stratified class proportions are undefined
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_0154
tools:
- name: scikit-learn
  role: Provides StratifiedKFold for the fivefold class-balanced data splits and metric
    computation (ROC-AUC, accuracy, precision, recall, F1)
- name: TensorFlow
  role: Trains the CNN within each cross-validation fold and produces sigmoid class
    probabilities for ROC-AUC scoring
  repo: https://github.com/jsehhs/MS2toImg-CNN
- name: Keras 2.14
  role: Defines the CNN architecture (two conv layers with 48 3x3 filters, five dense
    layers with dropout, sigmoid output) retrained identically per fold
  repo: https://github.com/jsehhs/MS2toImg-CNN
provenance:
  source_task_ids:
  - task_003
  - task_006
  - task_007
  source_content_hash: sha256:1a21d61bd0e997507a793b5d913485feee6ff5bc08e27e33ec2bc969ab4ce4ef
  source_papers:
  - doi: 10.48550/arxiv.2510.09716
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/jsehhs/MS2toImg-CNN
  tools:
  - Keras
  - Keras 2.14
  - MS-DIAL
  - MS2toImg
  - Python
  - Python 3.10
  - TensorFlow
  - matplotlib
  - pandas
  - scikit-learn
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 15
---

# stratified-cross-validation

## Summary

Applies fivefold StratifiedKFold cross-validation (scikit-learn) to train and evaluate the MS2toImg-CNN on binarized bioactivity labels (e.g., DPPH high/low), accumulating per-fold predictions to compute ROC-AUC, accuracy, precision, recall, and F1 with confusion matrices.

## When to use

Use when training a CNN classifier on grayscale molecular fingerprint images with imbalanced or skewed high/low bioactivity labels (normalized DPPH < 0.4 low, > 0.6 high) and you need stable fold-balanced metrics like AUC.

## When NOT to use

- Input is already a feature table from aligned peak picking where the Random Forest baseline (AUC 0.817) with standard cross-validation suffices
- The task is regression of continuous bioactivity values rather than high/low class discrimination
- Labels have not been binarized (mid-range normalized values 0.4–0.6 included), so stratified class proportions are undefined

## Inputs

- Grayscale 2D molecular fingerprint images (e.g., 150x140 pixels) resized with the TensorFlow resize function
- Binarized bioactivity labels (e.g., DPPH normalized values < 0.4 = low, > 0.6 = high) for the 545 wild soybean accessions
- Trained or trainable MS2toImg-CNN model (sigmoid output layer)

## Outputs

- Per-fold and mean ROC-AUC scores with ROC curves for the DPPH classification
- Accumulated cross-validated accuracy, precision, recall, and F1 scores (box plots per fold)
- Fivefold confusion matrices showing true/false positive and negative counts per fold
- Results table of all five metrics saved as CSV

## How to apply

Split the labeled image dataset using scikit-learn's StratifiedKFold with k=5 so each fold preserves the proportion of low- vs. high-activity samples (labels binarized at normalized-value thresholds: below 0.4 = low activity, above 0.6 = high activity). In each fold, train the identical CNN (two convolutional layers with 48 3x3 filters plus max pooling, five dense layers each followed by dropout, sigmoid output, binary cross-entropy loss, Adam optimizer) in TensorFlow/Keras. Predict class probabilities from the sigmoid output layer, then compute ROC-AUC and generate ROC curves from probability scores, and accumulate held-out predictions across folds to compute accuracy, precision, recall, and F1 (as in the paper's Figure 5 box plots). Generate per-fold confusion matrices and check that diagonal values are high with low standard deviation across folds, indicating balanced, stable classification. Stratification is essential here because the DPPH label binarization discards mid-range samples (0.4–0.6), and unstratified folds could skew the minority class.

## Related tools

- **scikit-learn** (Provides StratifiedKFold for the fivefold class-balanced data splits and metric computation (ROC-AUC, accuracy, precision, recall, F1))
- **TensorFlow** (Trains the CNN within each cross-validation fold and produces sigmoid class probabilities for ROC-AUC scoring) — https://github.com/jsehhs/MS2toImg-CNN
- **Keras 2.14** (Defines the CNN architecture (two conv layers with 48 3x3 filters, five dense layers with dropout, sigmoid output) retrained identically per fold) — https://github.com/jsehhs/MS2toImg-CNN

## Examples

```
python main.py 150 140
```

## Evaluation signals

- Reported ROC-AUC should be ≈ 0.96 for the DPPH high/low task, exceeding the Random Forest baseline AUC of 0.817
- Accuracy, recall, and F1 should each exceed 0.90, consistent with Figure 3B and Figure 5 of the paper
- Per-fold confusion matrices should show high diagonal values with low standard deviation and minimal false positives and false negatives
- Each fold must preserve the class proportions of the binarized labels (StratifiedKFold, k=5), verifiable from fold label counts
- All five metrics (accuracy, precision, recall, F1, AUC) must be present in the saved CSV results table

## Limitations

- Only samples with normalized bioactivity below 0.4 or above 0.6 are used; mid-range samples are excluded, reducing the effective dataset size
- Reported metrics depend on the specific 150x140 image resolution choice; smaller (75x70) resolution underperforms and larger (300x280) overfits with a large training-validation gap
- Per-fold hyperparameters are summarized only in Supplementary Table 1 and numeric Table 2 values are not in the text, limiting exact replication

## Evidence

- [results] Stratified fivefold CV preserving label balance: [evidence span withheld — anchor status: not_found]
- [results] Reported performance benchmark: [evidence span withheld — anchor status: not_found]
- [results] CNN architecture retrained per fold: [evidence span withheld — anchor status: not_found]
