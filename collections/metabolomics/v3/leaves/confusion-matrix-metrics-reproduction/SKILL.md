---
name: confusion-matrix-metrics-reproduction
description: Use when validating a re-trained DeepMSProfiler model on an independent
  test split of raw LC-MS data and you need to confirm reported accuracy/precision/recall/F1
  and reproduce confusion_matrix.svg.
when_to_use_negative:
- You only need exploratory feature heatmaps or metabolite-protein networks (run_feature/show_feature),
  not held-out classification metrics
- There is no independent test split — evaluating on the discovery/training split
  would not reproduce the reported test-set figures
- Input is already a feature/peak table rather than raw mzML/.npy LC-MS signals, which
  the end-to-end pipeline expects
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3474
tools:
- name: DeepMSProfiler
  role: Provides run_predict with plot_auc/plot_cm flags that generate ensemble predictions,
    the AUC curve, and the confusion matrix used to derive test metrics
  repo: https://github.com/yjdeng9/DeepMSProfiler
- name: Python (>= 3.6)
  role: Runtime for the PyPI-installed DeepMSProfiler package and metric computation
  repo: https://pypi.org/project/DeepMSProfiler/
- name: TensorFlow 2.2.0
  role: Backend executing the DenseNet121 ensemble used for prediction
- name: Keras 2.3.1
  role: Model API layer over TensorFlow for the trained ensemble
- name: CUDA 10.1
  role: GPU acceleration for ensemble inference (CPU-only environments also compatible)
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-024-51433-3
    title: An end-to-end deep learning method for mass spectrometry data analysis
      to reveal disease-specific metabolic profiles
  source_content_hash: sha256:12cc7b51bbecb9fc8f0d9a040d929b35ce6af7c2948ce823a4c49ddea4c5d131
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/yjdeng9/DeepMSProfiler
  repo_url: https://github.com/yjdeng9/DeepMSProfiler
  tools:
  - CUDA
  - CUDA 10.1
  - DeepMSProfiler
  - Keras
  - Keras 2.3.1
  - Python (>= 3.6)
  - TensorFlow
  - TensorFlow 2.2.0
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# confusion-matrix-metrics-reproduction

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Reproduces held-out test-set classification metrics (accuracy, precision, recall, F1) and a confusion matrix from DeepMSProfiler ensemble predictions, e.g. the MTBLS1129 colon-cancer benchmark reporting 97.9% accuracy / 98.7% precision / 93.4% recall / 95.8% F1.

## When to use

Use when validating a re-trained DeepMSProfiler model on an independent test split of raw LC-MS data and you need to confirm reported accuracy/precision/recall/F1 and reproduce confusion_matrix.svg.

## When NOT to use

- You only need exploratory feature heatmaps or metabolite-protein networks (run_feature/show_feature), not held-out classification metrics
- There is no independent test split — evaluating on the discovery/training split would not reproduce the reported test-set figures
- Input is already a feature/peak table rather than raw mzML/.npy LC-MS signals, which the end-to-end pipeline expects

## Inputs

- Trained DeepMSProfiler job directory (e.g. example/out/jobs007) containing the 10-run DenseNet121 ensemble
- Held-out test split samples as .npy files (auto-converted from mzML)
- Sample label file assigning disease-type labels and train/test split assignment

## Outputs

- confusion_matrix.svg from -plot_cm
- AUC curve from -plot_auc
- test_metrics.csv with accuracy, precision, recall, and F1 on the independent test set

## How to apply

After training the 10-run DenseNet121 ensemble (adam, lr 1e-4, batch size 8) on the discovery split, call run_predict on the job directory with plot_auc=True and plot_cm=True so the ensemble predicts every held-out sample and writes the confusion matrix to confusion_matrix.svg. Then compute accuracy, precision, recall, and F1 from the ensemble predictions on the test set and write them to test_metrics.csv. For the MTBLS1129 colon-cancer dataset (236 samples, 4:1 discovery/testing split), success means the metrics land at approximately accuracy 97.9%, precision 98.7%, recall 93.4%, F1 95.8%, tolerating small run-to-run variation from the 10 independent training runs.

## Related tools

- **DeepMSProfiler** (Provides run_predict with plot_auc/plot_cm flags that generate ensemble predictions, the AUC curve, and the confusion matrix used to derive test metrics) — https://github.com/yjdeng9/DeepMSProfiler
- **Python (>= 3.6)** (Runtime for the PyPI-installed DeepMSProfiler package and metric computation) — https://pypi.org/project/DeepMSProfiler/
- **TensorFlow 2.2.0** (Backend executing the DenseNet121 ensemble used for prediction)
- **Keras 2.3.1** (Model API layer over TensorFlow for the trained ensemble)
- **CUDA 10.1** (GPU acceleration for ensemble inference (CPU-only environments also compatible))

## Examples

```
python mainRun.py -data ../example/data/ -label ../example/label.txt -out ../jobs -run_pred -plot_cm -plot_auc
```

## Evaluation signals

- test-set accuracy reproduces ~97.9% on MTBLS1129 (allowing small run-to-run variation across the 10 training runs)
- Precision ~98.7%, recall ~93.4%, and F1 ~95.8% match the reported values within tolerance
- confusion_matrix.svg is produced in the job output directory
- AUC curve file is generated when -plot_auc is enabled
- Predictions cover every held-out test sample defined in the label file's 4:1 discovery/testing split

## Limitations

- Run-to-run variation across the 10 independent training runs means metrics will not match reported values exactly
- Pre-trained model access requires contacting the author; full reproduction requires re-training from scratch
- Requires the exact environment (Python >= 3.6, TensorFlow == 2.2.0, Keras == 2.3.1, CUDA 10.1) for faithful reproduction

## Evidence

- [results] reported MTBLS1129 test metrics: [evidence span withheld — anchor status: not_found]
