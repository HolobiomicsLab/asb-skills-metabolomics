---
name: cnn-image-classification-inference
description: Use when raw LC-MS/MS samples have been encoded as single-channel grayscale
  molecular-fingerprint images and a trained CNN must predict a binary bioactivity
  label (e.g., DPPH high/low) with AUC/F1-style evaluation.
when_to_use_negative:
- The input is already a curated feature table or aligned peak matrix (e.g., the Random
  Forest/PCA/OPLS-DA baseline setting) rather than 2D grayscale fingerprint images.
- The task requires structural identification or annotation of individual molecular
  features — the CNN deliberately bypasses compound identification.
- Bioactivity labels cannot be assigned to the required high/low groups (e.g., samples
  with intermediate normalized DPPH values between 0.4 and 0.6).
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_0092
tools:
- name: TensorFlow
  role: Deep-learning backend for loading and running the trained MS2toImg-CNN and
    obtaining sigmoid class probabilities
  repo: https://github.com/jsehhs/MS2toImg-CNN
- name: Keras 2.14
  role: Model-definition API for the CNN architecture used for bioactivity classification
  repo: https://github.com/jsehhs/MS2toImg-CNN
- name: scikit-learn
  role: Provides StratifiedKFold (k=5) for cross-validation and metric computation
    (accuracy, precision, recall, F1, ROC-AUC)
provenance:
  source_task_ids:
  - task_003
  source_content_hash: sha256:61b50069227cd233c89cd5d373b911c2f4319bcf2ba905d478488073a9827e14
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# cnn-image-classification-inference

## Summary

Runs a trained convolutional neural network (MS2toImg-CNN, Keras/TensorFlow) on 2D grayscale molecular-fingerprint images built from raw LC-MS/MS data to predict binary bioactivity class, then scores predictions (accuracy, precision, recall, F1, ROC-AUC) under stratified 5-fold cross-validation.

## When to use

Use when raw LC-MS/MS samples have been encoded as single-channel grayscale molecular-fingerprint images and a trained CNN must predict a binary bioactivity label (e.g., DPPH high/low) with AUC/F1-style evaluation.

## When NOT to use

- The input is already a curated feature table or aligned peak matrix (e.g., the Random Forest/PCA/OPLS-DA baseline setting) rather than 2D grayscale fingerprint images.
- The task requires structural identification or annotation of individual molecular features — the CNN deliberately bypasses compound identification.
- Bioactivity labels cannot be assigned to the required high/low groups (e.g., samples with intermediate normalized DPPH values between 0.4 and 0.6).

## Inputs

- Preprocessed 2D single-channel grayscale molecular-fingerprint images of LC-MS/MS samples (75x70, 150x140, or 300x280 pixels)
- Binary bioactivity labels (e.g., normalized DPPH < 0.4 = low, > 0.6 = high)
- Trained MS2toImg-CNN model weights (Keras 2.14 / TensorFlow)

## Outputs

- Per-fold class probability predictions from the CNN sigmoid output
- ROC curve and ROC-AUC for the bioactivity classification
- Aggregated accuracy, precision, recall, F1, and AUC across 5 stratified folds
- Per-fold confusion matrices
- Results table saved in CSV format

## How to apply

Load the trained MS2toImg-CNN (implemented in Python 3.10/3.11 with Keras 2.14 and TensorFlow) and the preprocessed 2D fingerprint images, choosing one of the tested input resolutions (75x70, 150x140, or 300x280) that balances predictive accuracy against compute cost. Assign binary labels from the normalized bioassay values — for the DPPH task, normalized activity < 0.4 is the low-activity group and > 0.6 is the high-activity group (intermediate samples excluded). Run 5-fold stratified cross-validation (scikit-learn StratifiedKFold, k=5), taking class probabilities from the CNN's sigmoid output layer, and compute ROC-AUC from the probability scores plus accuracy, precision, recall, and F1 by accumulating test-set predictions across folds. The rationale is that the image-based CNN leverages the overall fragmentation landscape (with non-informative pixels above the MS2>MS1 diagonal masked to zero) rather than precise feature alignment, making it robust to alignment errors and requiring no prior compound identification. Tabulate all five metrics and save as a results CSV, then benchmark against baselines (e.g., Random Forest, AUC 0.817).

## Related tools

- **TensorFlow** (Deep-learning backend for loading and running the trained MS2toImg-CNN and obtaining sigmoid class probabilities) — https://github.com/jsehhs/MS2toImg-CNN
- **Keras 2.14** (Model-definition API for the CNN architecture used for bioactivity classification) — https://github.com/jsehhs/MS2toImg-CNN
- **scikit-learn** (Provides StratifiedKFold (k=5) for cross-validation and metric computation (accuracy, precision, recall, F1, ROC-AUC))

## Examples

```
python main.py 75 70
```

## Evaluation signals

- Reported ROC-AUC should reach ~0.96 on the DPPH task, exceeding the Random Forest baseline AUC of 0.817.
- Accuracy, recall, and F1 should each exceed 0.90, consistent with the article's Figure 3B and Figure 5 box plots.
- Confusion matrices across the 5 stratified folds should show balanced, stable classification with minimal false positives and false negatives.
- Label assignment must respect the normalized-DPPH thresholds (< 0.4 low, > 0.6 high) before splitting into folds.
- All five metrics (accuracy, precision, recall, F1, AUC) should be tabulated and saved to a CSV results table.

## Limitations

- Repository code is under review with a NOASSERTION license — redistribution, modification, or reuse for other research/commercial purposes is not permitted until formal publication; authors must be contacted for permission.
- Exact numeric performance values (Table 2) and training hyperparameters (Supplementary Table 1) are not provided in the article text, so exact metric parity may not be verifiable without contacting authors.
- Input image resolution trades predictive accuracy against computational efficiency; predictions depend on the chosen resolution (75x70, 150x140, or 300x280).
- The pipeline is validated only on wild soybean (Glycine soja Sieb. & Zucc.) bioassay data; generalization to other extract types or assays is asserted but not independently verified here.

## Evidence

- [results] we applied a masking procedure in which all pixels within this non-informative region were set to zero (black): [evidence span withheld — anchor status: not_found]
