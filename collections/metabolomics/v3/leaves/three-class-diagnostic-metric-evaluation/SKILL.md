---
name: three-class-diagnostic-metric-evaluation
description: Use when a multi-class LC-MS/metabolomics classifier (e.g., 10-run DenseNet121
  ensemble) must be benchmarked on held-out .npy spectra with disease-type labels,
  and accuracy/precision/recall/F1/AUC plus AUC-curve and confusion-matrix figures
  are the deliverable.
when_to_use_negative:
- Input is already a peak-picked/annotated metabolite feature table — this skill evaluates
  end-to-end raw-spectrum classification, not conventional peak-table pipelines
- The task is binary or uses disease groups other than healthy/nodule/lung cancer,
  so the paper's per-class accuracy targets (85.7%/90.8%/97.0%) are not meaningful
  comparators
- The model is still training — metric evaluation requires a completed ensemble and
  a fixed held-out test split
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3474
tools:
- name: DeepMSProfiler
  role: Executes ensemble prediction on raw .npy spectra and generates the AUC curve
    and confusion matrix via -plot_auc / -plot_cm flags
  repo: https://github.com/yjdeng9/DeepMSProfiler
- name: TensorFlow
  role: Backend (v2.2.0) running the 10-run DenseNet121 ensemble inference whose aggregated
    predictions are scored
- name: Keras
  role: Model API (v2.3.1) defining the DenseNet121 architecture used by the ensemble
    being evaluated
- name: Python (>= 3.6)
  role: Runtime for the CLI entry point mainRun.py and the PyPI API (run_predict with
    plot_auc=True, plot_cm=True)
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-024-51433-3
    title: An end-to-end deep learning method for mass spectrometry data analysis
      to reveal disease-specific metabolic profiles
  source_content_hash: sha256:d34e12a4226dc00a46157625434082f5828a4ce2563f86b628a09d36b1d73c49
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
  edam_operation: http://edamontology.org/operation_2990
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4
n_claims: 15
---

# three-class-diagnostic-metric-evaluation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Evaluates a trained tri-class diagnostic classifier (lung adenocarcinoma vs benign lung nodule vs healthy) on raw serum LC-MS samples by computing accuracy, precision, recall, F1, and AUC on an independent test set and visualizing the AUC curve and confusion matrix. It establishes whether the model reproduces the reported diagnostic performance (AUC 0.99, accuracy 95%).

## When to use

Use when a multi-class LC-MS/metabolomics classifier (e.g., 10-run DenseNet121 ensemble) must be benchmarked on held-out .npy spectra with disease-type labels, and accuracy/precision/recall/F1/AUC plus AUC-curve and confusion-matrix figures are the deliverable.

## When NOT to use

- Input is already a peak-picked/annotated metabolite feature table — this skill evaluates end-to-end raw-spectrum classification, not conventional peak-table pipelines
- The task is binary or uses disease groups other than healthy/nodule/lung cancer, so the paper's per-class accuracy targets (85.7%/90.8%/97.0%) are not meaningful comparators
- The model is still training — metric evaluation requires a completed ensemble and a fixed held-out test split

## Inputs

- Raw serum LC-MS spectra as .npy arrays (.mzML files are auto-converted to .npy by the script)
- Label file mapping each sample to healthy/nodule/cancer and marking train vs test split
- Trained ensemble of 10 DenseNet121 models (job directory, or -pretrain model path)
- Package environment: Python >= 3.6, TensorFlow 2.2.0, Keras 2.3.1, optional CUDA 10.1

## Outputs

- Aggregate classification metrics on the independent test set: accuracy, precision, recall, F1, AUC
- Three-class AUC curve figure (via -plot_auc)
- Confusion matrix figure, e.g. confusion_matrix.svg (via -plot_cm)
- Per-class diagnostic accuracies for LUAD, benign nodule, and healthy groups

## How to apply

Split the .npy serum LC-MS spectra according to the label file (sample → health/nodule/cancer, train/test columns), then run prediction on the held-out split with the 10-model DenseNet121 ensemble in ensemble mode. Enable the -plot_auc and -plot_cm flags so the package emits the three-class AUC curve and confusion matrix into the job's output directory. Compute accuracy, precision, recall, F1, and AUC from the ensemble's aggregated predictions rather than any single run, since the ensemble strategy is what removes background-category phenomena in multi-class deep-learning models. Validate against the paper's independent testing set of 859 cross-hospital samples (210 healthy, 323 lung nodules, 326 lung cancer): expected values are AUC ~0.99, accuracy ~95%, precision ~96%, recall ~95%, F1 ~98%, with per-class accuracies ~85.7% (LUAD), ~90.8% (benign nodule), and ~97.0% (healthy). If deviating materially, check train/test leakage in the datalist and confirm the ensemble (not single-run) prediction mode was used.

## Related tools

- **DeepMSProfiler** (Executes ensemble prediction on raw .npy spectra and generates the AUC curve and confusion matrix via -plot_auc / -plot_cm flags) — https://github.com/yjdeng9/DeepMSProfiler
- **TensorFlow** (Backend (v2.2.0) running the 10-run DenseNet121 ensemble inference whose aggregated predictions are scored)
- **Keras** (Model API (v2.3.1) defining the DenseNet121 architecture used by the ensemble being evaluated)
- **Python (>= 3.6)** (Runtime for the CLI entry point mainRun.py and the PyPI API (run_predict with plot_auc=True, plot_cm=True))

## Examples

```
python mainRun.py -data ../example/all_data/ -label ../example/all_label.txt -out ../jobs -run_pred -mode ensemble -pretrain ../example/pretrain_model -plot_auc -plot_cm
```

## Evaluation signals

- Independent-test-set metrics fall near reported values: AUC ~0.99, accuracy ~95%, precision ~96%, recall ~95%, F1 ~98%
- Per-class accuracies approximate 85.7% (LUAD), 90.8% (benign nodule), 97.0% (healthy), consistent with the published confusion matrix
- Confusion matrix plot is generated in the job output directory (confusion_matrix.svg) with majority of mass on the diagonal
- AUC curve figure produced by -plot_auc covers all three classes, not a single one-vs-rest binary curve only
- Predictions come from ensemble mode over 10 runs — single-run scores below the ensemble baseline indicate the wrong -mode was used

## Limitations

- The pre-trained 859-sample model (210 healthy, 323 lung nodules, 326 lung cancer) is not openly downloadable; access requires contacting the author
- Reported metrics were obtained on a specific cross-hospital serum cohort; performance on other instruments, cohorts, or sample types is not guaranteed
- The demo dataset is very small (a handful of test samples per class), so metrics computed on demo data will be unstable and will not reproduce the 859-sample results

## Evidence
