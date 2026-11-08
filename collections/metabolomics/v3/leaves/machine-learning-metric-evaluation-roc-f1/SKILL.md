---
name: machine-learning-metric-evaluation-roc-f1
description: Use when a pixel-wise classifier trained on labeled mass spectra (e.g.,
  massNet or SVM on MALDI FT-ICR MSI data) must be benchmarked on a withheld test
  set with accuracy, precision, recall, F1-score, ROC, and runtime.
when_to_use_negative:
- Data has not been split into training and withheld test sets — metrics computed
  on training data would not reproduce the paper's test-set evaluation
- Only qualitative visualization (e.g., UMAP) is needed rather than quantitative classifier
  benchmarking
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_0121
tools:
- name: massNet
  role: Deep learning classifier whose test-set performance is scored with the metrics
  repo: github.com/wabdelmoula/massNet
- name: support vector machine (SVM)
  role: Classical ML baseline evaluated with the same metrics for comparison
- name: sklearn
  role: Python package (v0.23.2) used in the massNet implementation, providing metric
    evaluation utilities
- name: Python
  role: Implementation language for the model and metric evaluation
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btac032
    title: 'massNet: integrated processing and classification of spatially resolved
      mass spectrometry data using deep learning for rapid tumor delineation'
  source_content_hash: sha256:bcabc0d9c2a763f4e02f3b50f9618ade4bcaf81aded01242aa82e4a9682fb60b
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/wabdelmoula/massNet
  tools:
  - Python
  - massNet
  - sklearn
  - support vector machine (SVM)
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# machine-learning-metric-evaluation-roc-f1

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Evaluates a trained classifier (e.g., massNet or an SVM baseline) on a withheld test set using standard machine learning metrics — accuracy, precision, recall, F1-score, and ROC — and pairs these with runtime measurements for side-by-side benchmarking.

## When to use

Use when a pixel-wise classifier trained on labeled mass spectra (e.g., massNet or SVM on MALDI FT-ICR MSI data) must be benchmarked on a withheld test set with accuracy, precision, recall, F1-score, ROC, and runtime.

## When NOT to use

- Data has not been split into training and withheld test sets — metrics computed on training data would not reproduce the paper's test-set evaluation
- Only qualitative visualization (e.g., UMAP) is needed rather than quantitative classifier benchmarking

## Inputs

- Withheld test set of pixel-labeled mass spectra (9,798 normal and 2,861 tumor spectra; 85,062 m/z variables) from converted HDF5 MSI file
- Trained classifier predictions on the test set (massNet or SVM)
- Classifier runtime measurements on the test set

## Outputs

- Side-by-side benchmark table (Table 1 / svm_benchmark_table.csv) with accuracy, precision, recall, F1-score, ROC, and runtime per classifier
- Computed speed comparison ratio between classifiers (e.g., 174-fold)

## How to apply

Apply the trained classifier (massNet or the SVM baseline trained on 10,024 normal and 3,644 tumor spectra with 85,062 m/z variables) to the withheld test set of 9,798 normal and 2,861 tumor spectra and record predictions. Evaluate predictions with the established metrics used in the paper: accuracy, precision, recall, F1-score, and ROC. Separately measure classification runtime on the test set and compute the speed ratio against the comparator — massNet reported a 174-fold gain over SVM with 95% vs 93.45% accuracy. Assemble both models' metrics and runtimes into a single side-by-side comparison table (Table 1, svm_benchmark_table.csv) so performance and speed are judged together, since the paper's claim rests on higher accuracy AND substantial speed gain.

## Related tools

- **massNet** (Deep learning classifier whose test-set performance is scored with the metrics) — github.com/wabdelmoula/massNet
- **support vector machine (SVM)** (Classical ML baseline evaluated with the same metrics for comparison)
- **sklearn** (Python package (v0.23.2) used in the massNet implementation, providing metric evaluation utilities)
- **Python** (Implementation language for the model and metric evaluation)

## Examples

```
from sklearn.metrics import accuracy_score, precision_score, recall_score, f1_score, roc_curve; acc = accuracy_score(y_test, y_pred)
```

## Evaluation signals

- Reproduced SVM accuracy on the withheld test set matches the reported 93.45% within tolerance, against massNet's 95%
- Computed SVM-to-massNet runtime ratio is consistent with the reported 174-fold gain
- Table includes all five reported metrics (accuracy, precision, recall, F1-score, runtime) for both classifiers
- Test-set composition matches the paper: 9,798 normal and 2,861 tumor spectra with 85,062 m/z variables

## Limitations

- Ground truth annotations transferred from pathologist-annotated H&E images are an approximation and may not be optimal, especially with intra-tumor heterogeneity
- massNet was demonstrated only for binary classification (normal vs tumor); multi-class metric evaluation is left to future development
- No software version numbers or hyperparameter values for the trained model are reported in the discussion, limiting exact reproduction

## Evidence

- [intro] Research objective: assess model stability via 5-fold cross validation and evaluate classification speed and robustness on a test set with standard machine learning metrics (ROC, accuracy, recall, precision, F1-score).: [evidence span withheld — anchor status: not_found]
- [results] massNet slightly outperformed SVM in classification accuracy on the withheld test set (95% vs. 93.45%) while being 174 times faster.: [evidence span withheld — anchor status: not_found]
