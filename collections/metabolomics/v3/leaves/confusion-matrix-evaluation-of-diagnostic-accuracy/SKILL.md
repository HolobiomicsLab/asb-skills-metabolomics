---
name: confusion-matrix-evaluation-of-diagnostic-accuracy
description: Use when an end-to-end LC-MS metabolomics classifier must be validated
  on held-out testing samples and its per-class accuracy visualized as a confusion
  matrix.
when_to_use_negative:
- Input is already a peak-picked, aligned and annotated feature table — DeepMSProfiler
  is designed for end-to-end analysis of raw LC-MS signals without conventional feature
  extraction.
- The goal is metabolite identification or quantification rather than sample-level
  disease classification across the three serum classes.
- Only a single-model prediction without ensemble aggregation is needed, since the
  accuracy claim depends on ensemble mode over the 10-run model set.
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: DeepMSProfiler
  role: Provides the -run_pred / -plot_cm pipeline that runs ensemble inference over
    tri-class labels and renders the confusion matrix (confusion_matrix.svg).
  repo: https://github.com/yjdeng9/DeepMSProfiler
- name: TensorFlow
  role: Runtime backend (version 2.2.0) on which the pre-trained DenseNet121 ensemble
    models execute during prediction.
- name: Keras
  role: Model API layer (version 2.3.1) used to load and apply the pre-trained DenseNet121
    ensemble.
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-024-51433-3
    title: An end-to-end deep learning method for mass spectrometry data analysis
      to reveal disease-specific metabolic profiles
  source_content_hash: sha256:aaca49a0f57ae7fd619c8bea964fb883c5c44b5ee8bedb13b7a9f2061e452bea
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# confusion-matrix-evaluation-of-diagnostic-accuracy

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Evaluates a trained DeepMSProfiler ensemble on independent testing samples by aggregating tri-class predictions (healthy / lung nodule / lung adenocarcinoma) and rendering a confusion matrix, the primary evidence for the reported 96.1% early-stage detection accuracy.

## When to use

Use when an end-to-end LC-MS metabolomics classifier must be validated on held-out testing samples and its per-class accuracy visualized as a confusion matrix.

## When NOT to use

- Input is already a peak-picked, aligned and annotated feature table — DeepMSProfiler is designed for end-to-end analysis of raw LC-MS signals without conventional feature extraction.
- The goal is metabolite identification or quantification rather than sample-level disease classification across the three serum classes.
- Only a single-model prediction without ensemble aggregation is needed, since the accuracy claim depends on ensemble mode over the 10-run model set.

## Inputs

- Independent testing LC-MS metabolomics data in .npy format (auto-converted from mzML if raw files are supplied), e.g. ../example/all_data/
- Tri-class sample label file (healthy / nodule / cancer), e.g. ../example/all_label.txt
- Pre-trained DenseNet121 ensemble model directory (10 runs), e.g. ../example/pretrain_model

## Outputs

- Aggregated ensemble classification accuracy on the testing samples (expected 0.961 for stage-I lung adenocarcinoma)
- Tri-class AUC (~0.99) when -plot_auc is enabled
- Confusion matrix figure saved as confusion_matrix.svg in the job output directory

## How to apply

Run DeepMSProfiler in prediction-only mode by pointing -data at the independent testing .npy files (auto-converted from mzML if raw), -label at the tri-class label file, and -pretrain at the pre-trained DenseNet121 ensemble trained on 859 serum metabolomics samples (210 healthy, 323 lung nodules, 326 lung cancer). Execute with -run_pred -plot_cm so inference runs in ensemble mode over 10 DenseNet121 runs (adam optimizer, lr 1e-04); the tool aggregates the ensemble predictions per sample before scoring. Compute the classification accuracy for the lung cancer (stage-I adenocarcinoma) class from the aggregated predictions, and optionally pass -plot_auc for the tri-class AUC. The confusion matrix is saved as confusion_matrix.svg in the output job directory, providing the visual check that errors concentrate between nodule and cancer classes rather than collapsing into the healthy background category.

## Related tools

- **DeepMSProfiler** (Provides the -run_pred / -plot_cm pipeline that runs ensemble inference over tri-class labels and renders the confusion matrix (confusion_matrix.svg).) — https://github.com/yjdeng9/DeepMSProfiler
- **TensorFlow** (Runtime backend (version 2.2.0) on which the pre-trained DenseNet121 ensemble models execute during prediction.)
- **Keras** (Model API layer (version 2.3.1) used to load and apply the pre-trained DenseNet121 ensemble.)

## Examples

```
python mainRun.py -data ../example/all_data/ -label ../example/all_label.txt -out ../jobs -run_pred -pretrain ../example/pretrain_model -plot_cm
```

## Evaluation signals

- Aggregated testing-sample accuracy should reproduce the reported 0.961 for early-stage lung adenocarcinoma detection.
- Tri-class AUC should be at the reported 0.99 level.
- confusion_matrix.svg must be generated in the -out job directory after the -plot_cm run completes.
- Predictions must come from ensemble mode over 10 DenseNet121 runs, not a single run.
- Input labels must resolve to the tri-class scheme (healthy, nodule, cancer) matching the 859-sample pre-trained model.

## Limitations

- The pre-trained serum model required for the 0.961 accuracy benchmark is distributed for academic use via author request ('Please contact the author for access').
- Requires pinned dependency versions (Python >= 3.6, TensorFlow 2.2.0, Keras 2.3.1) and CUDA 10.1 capability, though CPU-only environments are also supported.
- The confusion matrix reflects performance on the independent testing samples of the serum dataset; it does not by itself demonstrate generalization to new hospitals (inter-hospital variability was addressed separately in ablation experiments).

## Evidence
