---
name: confusion-matrix-and-auc-visualization
description: Use when evaluating a trained DeepMSProfiler (DenseNet121 ensemble) classifier
  on a held-out LC-MS test split and you need ROC/AUC curves and a per-class confusion
  matrix to confirm accuracy, precision, recall, F1, and AUC.
when_to_use_negative:
- Input is raw spectra with no trained model or prediction output yet — run -run_train
  and -run_pred first, since -plot_auc/-plot_cm operate on prediction results of an
  existing job.
- The task is binary or regression modeling of metabolomic data; the flags were exercised
  on the three-class lung cancer / benign nodule / healthy serum LC-MS setup.
- You only need metabolite interpretability outputs (saliency heatmaps / metabolite-protein
  networks) — use -run_feature instead of the plotting flags.
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3474
tools:
- name: DeepMSProfiler
  role: Provides the -plot_auc and -plot_cm command-line flags and the run_predict(plot_auc=True,
    plot_cm=True) API that render AUC curves and confusion matrices from ensemble
    prediction results
  repo: https://github.com/yjdeng9/DeepMSProfiler
- name: TensorFlow
  role: Backend (version 2.2.0) on which the DenseNet121 ensemble predictions feeding
    the plots are computed
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-024-51433-3
    title: An end-to-end deep learning method for mass spectrometry data analysis
      to reveal disease-specific metabolic profiles
  source_content_hash: sha256:e016fdca9322a080f3a47e90facbc26972fbb0b1fe534b211a78e1872c694f64
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
  - DeepMSProfiler
  - Keras
  - Python (>= 3.6)
  - TensorFlow
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
emails_redacted: true
evidence_trimmed: true
triage_status: critique
yes_rate: 0.42105263157894735
n_claims: 19
---

# confusion-matrix-and-auc-visualization

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Generate AUC curves and per-class confusion matrices for a multi-class (healthy/benign nodule/lung cancer) LC-MS deep-learning classifier using DeepMSProfiler's -plot_auc and -plot_cm flags, to verify diagnostic performance against reported metrics (AUC 0.99, accuracy ~0.95).

## When to use

Use when evaluating a trained DeepMSProfiler (DenseNet121 ensemble) classifier on a held-out LC-MS test split and you need ROC/AUC curves and a per-class confusion matrix to confirm accuracy, precision, recall, F1, and AUC.

## When NOT to use

- Input is raw spectra with no trained model or prediction output yet — run -run_train and -run_pred first, since -plot_auc/-plot_cm operate on prediction results of an existing job.
- The task is binary or regression modeling of metabolomic data; the flags were exercised on the three-class lung cancer / benign nodule / healthy serum LC-MS setup.
- You only need metabolite interpretability outputs (saliency heatmaps / metabolite-protein networks) — use -run_feature instead of the plotting flags.

## Inputs

- Trained ensemble job directory (e.g. ../jobs/jobs007) containing saved DenseNet121 models and prediction results
- Sample label file mapping .npy (or auto-converted .mzML) LC-MS spectra to healthy/nodule/cancer classes with train/test splits
- Boolean run flags -plot_auc and -plot_cm

## Outputs

- AUC curve figure for the three-class classification
- Per-class confusion matrix figure (e.g. confusion_matrix.svg)
- Accuracy, precision, recall, F1, and AUC values computable from the matrix (expected ~0.95/0.96/0.95/0.98, AUC 0.99)

## How to apply

After running prediction with the ensemble mode enabled (-run_pred -mode ensemble) on the held-out test split, re-run the job with the -plot_auc and -plot_cm Boolean flags so DeepMSProfiler emits the AUC curve and confusion matrix figures for the three-class classification (healthy, benign nodule, lung cancer). The confusion matrix should be compared to the reported per-class accuracies (~85.7% LUAD, ~90.8% benign nodule, ~97.0% healthy), and the AUC curve to the reported 0.99. In the Python API this corresponds to run_predict(job_dir=..., plot_auc=True, plot_cm=True). Judging correctness: the figures are written to the job's output directory and the confusion matrix should approximate the published one (accuracy ~0.95, precision 0.96, recall 0.95, F1 0.98).

## Related tools

- **DeepMSProfiler** (Provides the -plot_auc and -plot_cm command-line flags and the run_predict(plot_auc=True, plot_cm=True) API that render AUC curves and confusion matrices from ensemble prediction results) — https://github.com/yjdeng9/DeepMSProfiler
- **TensorFlow** (Backend (version 2.2.0) on which the DenseNet121 ensemble predictions feeding the plots are computed)

## Examples

```
python mainRun.py -data ../example/all_data/ -label ../example/all_label.txt -out ../jobs -run_pred -pretrain ../example/pretrain_model -plot_cm
```

## Evaluation signals

- Computed AUC on the independent test set approximates 0.99, consistent with the published three-class ROC curve.
- Confusion matrix yields accuracy ~0.95, precision ~0.96, recall ~0.95, and F1 ~0.98 on the 859-sample cross-hospital testing dataset.
- Per-class accuracies from the matrix match ~85.7% lung adenocarcinoma, ~90.8% benign nodule, ~97.0% healthy.
- Figure files (AUC curve and confusion_matrix.svg) are written to the job's output directory after run_predict with plot_auc=True, plot_cm=True.
- Row sums of the confusion matrix match the test-set class counts (e.g. cancer, health, nodule entries in the label file marked 'test').

## Limitations

- Pre-trained model access (used to reproduce the published figures without retraining) requires contacting the author: <maintainer e-mail, see the repository README>.
- Figures reflect ensemble-mode predictions; single-model or boosted runs may not reproduce the reported metrics.
- Environment pins (Python >= 3.6, TensorFlow 2.2.0, Keras 2.3.1, CUDA 10.1) may be required; CPU-only execution is supported but slower.

## Evidence
