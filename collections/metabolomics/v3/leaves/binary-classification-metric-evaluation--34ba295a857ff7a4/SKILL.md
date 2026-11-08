---
name: binary-classification-metric-evaluation
description: Use when a CNN or ML model outputs binary class probabilities (e.g. DPPH
  high vs. low activity) and you need AUC, accuracy, precision, recall, and F1 with
  confidence from cross-validation.
when_to_use_negative:
- When the task is a multi-class or continuous bioactivity regression (e.g. TFC or
  TPC quantification) rather than a two-group DPPH classification
- When the input is already a structured feature table from traditional pipelines
  — the CNN baseline comparison (PCA, OPLS-DA, Random Forest) is a separate skill,
  not metric evaluation
- When no labeled bioassay ground truth exists to compute AUC, accuracy, precision,
  recall, or F1
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_3520
tools:
- name: TensorFlow
  role: Runs the MS2toImg-CNN forward pass to obtain sigmoid class probabilities for
    each fold
  repo: https://github.com/jsehhs/MS2toImg-CNN
- name: scikit-learn
  role: Provides StratifiedKFold splitting and ROC-AUC / accuracy / precision / recall
    / F1 computation
  repo: https://github.com/jsehhs/MS2toImg-CNN
provenance:
  source_task_ids:
  - task_003
  source_content_hash: sha256:af5e76631703cd5c03be7ddef2e5996b0f51159a8038b1782d3a5e8dd3954914
  source_papers:
  - doi: 10.48550/arxiv.2510.09716
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/jsehhs/MS2toImg-CNN
  tools:
  - TensorFlow
  - scikit-learn
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# binary-classification-metric-evaluation

## Summary

Evaluates the MS2toImg-CNN binary bioactivity classifier by computing ROC-AUC plus accuracy, precision, recall, and F1 across 5-fold stratified cross-validation on DPPH high/low activity labels.

## When to use

Use when a CNN or ML model outputs binary class probabilities (e.g. DPPH high vs. low activity) and you need AUC, accuracy, precision, recall, and F1 with confidence from cross-validation.

## When NOT to use

- When the task is a multi-class or continuous bioactivity regression (e.g. TFC or TPC quantification) rather than a two-group DPPH classification
- When the input is already a structured feature table from traditional pipelines — the CNN baseline comparison (PCA, OPLS-DA, Random Forest) is a separate skill, not metric evaluation
- When no labeled bioassay ground truth exists to compute AUC, accuracy, precision, recall, or F1

## Inputs

- Grayscale 2D molecular fingerprint images generated from raw LC-MS/MS data (MS1 m/z on x-axis, MS2 m/z on y-axis, masked above the diagonal)
- Binary DPPH activity labels derived from normalized assay values (<0.4 = low, >0.6 = high)
- Trained MS2toImg-CNN model with sigmoid output layer (e.g. checkpoint from python main.py 75 70)

## Outputs

- ROC curve and scalar AUC value (target ≈ 0.96 for DPPH task)
- Per-fold and aggregated accuracy, precision, recall, and F1 scores (box-plotted as in Figure 5)
- 5-fold cross-validation confusion matrices showing balanced, stable classification

## How to apply

Split the wild soybean sample set with 5-fold stratified cross-validation (scikit-learn StratifiedKFold), feeding grayscale 2D molecular fingerprint images into the MS2toImg-CNN sigmoid output layer to get class probabilities. Compute ROC-AUC from the probability scores and plot the ROC curve. Accumulate held-out test predictions across all folds and compute accuracy, precision, recall, and F1 to match Figure 5's box plots. DPPH labels are assigned from normalized assay values: <0.4 = low-activity, >0.6 = high-activity, with mid-range samples excluded. Success is judged by whether metrics meet or exceed the paper's reported values (AUC ≈ 0.96, accuracy/recall/F1 > 0.90) and beat the Random Forest baseline (AUC 0.817).

## Related tools

- **TensorFlow** (Runs the MS2toImg-CNN forward pass to obtain sigmoid class probabilities for each fold) — https://github.com/jsehhs/MS2toImg-CNN
- **scikit-learn** (Provides StratifiedKFold splitting and ROC-AUC / accuracy / precision / recall / F1 computation) — https://github.com/jsehhs/MS2toImg-CNN

## Examples

```
python main.py 75 70
```

## Evaluation signals

- AUC on the DPPH binary task should reach ≈ 0.96 (well above the Random Forest baseline of 0.817)
- Aggregated accuracy, precision, recall, and F1 must each exceed 0.90 across the 5 folds (per Figure 3B and Figure 5)
- Confusion matrices from each fold should show minimal false positives and false negatives with balanced, stable counts
- Label assignment invariants: samples with normalized DPPH < 0.4 are low-activity, > 0.6 are high-activity; 0.4–0.6 band must be excluded from training and evaluation
- Reproducibility check: re-running the evaluation with the same image resolution (e.g. 75×70) and fold seeds must reproduce metric means within a narrow band across folds

## Limitations

- Metrics are only as good as the binarization: normalized DPPH values in the 0.4–0.6 range are unclassified and reduce usable sample count
- Performance values in the manuscript's Table 2 and Supplementary Table 1 hyperparameters are not printed in text, so exact per-metric numbers must be re-derived from figures or re-runs
- The evaluation depends on image resolution choice (75×70 vs 150×140 vs 300×280); resolution changes shift both accuracy and computational cost
- Results are specific to the wild soybean LC-MS/MS dataset (545 accessions) and DPPH assay; transfer to other bioassays requires re-training and re-labeling

## Evidence

- [methods] DPPH activity group assignment: "Samples with normalized values below 0.4 were categorized as the low-activity group, while those above 0.6 were categorized as the high-activity group."
- [results] fivefold cross-validation metrics: [evidence span withheld — anchor status: not_found]
- [abstract] baseline comparison target: "The MS2toImg-CNN model outperformed conventional machine learning baselines (e.g., Random Forest, PCA), demonstrating robust classification accuracy across diverse tasks."
- [results] tool stack for evaluation: "The entire framework was implemented in Python (v3.10) using the Keras (v2.14) and Tensor Flow libraries."
- [readme] image resolution and invocation: "to train model with 75 x 70 image:
```
python main.py 75 70
```"
