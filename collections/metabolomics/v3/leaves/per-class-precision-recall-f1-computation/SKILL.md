---
name: per-class-precision-recall-f1-computation
description: Use when you have a multiclass confusion matrix or class-level prediction
  counts and need to identify which class labels (e.g. NPClassifier superclasses)
  a model like DeepSAT predicts worst.
when_to_use_negative:
- Only an aggregate top-1 accuracy is available with no confusion matrix or per-class
  counts — precision/recall/F1 cannot be reconstructed
- The task is binary classification or the goal is overall model performance, not
  diagnosing which individual class labels fail
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_0157
tools:
- name: DeepSAT
  role: Source of the multiclass classification predictions (NPClassifier superclass
    head trained on 143,467 1H-13C HSQC spectra) whose confusion matrix is decomposed
    into per-class metrics
  repo: https://github.com/mwang87/DeepSAT
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1186/s13321-023-00738-4
    title: 'DeepSAT: Learning Molecular Structures from Nuclear Magnetic Resonance
      Data'
  source_content_hash: sha256:d40803103e2669598f8f1c5834721c048a7adc96f1ad4e4e9ba209b2b0d5dd3a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mwang87/DeepSAT
  tools:
  - DeepSAT
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_0157
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# per-class-precision-recall-f1-computation

## Summary

Computes per-class precision, recall, and F1 from a multiclass confusion matrix (e.g. the 59 NPClassifier superclass labels predicted by DeepSAT) to expose weakly predicted structure classes that aggregate top-1 accuracy hides.

## When to use

Use when you have a multiclass confusion matrix or class-level prediction counts and need to identify which class labels (e.g. NPClassifier superclasses) a model like DeepSAT predicts worst.

## When NOT to use

- Only an aggregate top-1 accuracy is available with no confusion matrix or per-class counts — precision/recall/F1 cannot be reconstructed
- The task is binary classification or the goal is overall model performance, not diagnosing which individual class labels fail

## Inputs

- Multiclass confusion matrix or class-level true-positive/false-positive/false-negative counts for the 59 NPClassifier superclass labels from DeepSAT's classification head
- Per-sample predicted and true superclass labels from evaluation of the 143,467 1H-13C HSQC spectra models (normal HSQC and multiplicity edited HSQC)

## Outputs

- per_class_metrics.csv containing per-class precision, recall, and F1 with best- and worst-predicted classes annotated
- Ranked list of the 59 superclass labels by F1-score with weakest performers flagged

## How to apply

Load the 59-class NPClassifier superclass confusion matrix produced during evaluation of DeepSAT's classification head on the 143,467 1H-13C HSQC spectra. For each class, compute precision as TP/(TP+FP) and recall as TP/(TP+FN) directly from the confusion-matrix counts, then derive F1 = 2*P*R/(P+R). Rank all 59 classes by F1 and flag the weakest performers — in DeepSAT's evaluation, diazotetronic acids and derivatives (F1 ≈ 0.64, attributed to a proton-deficient structure with few HSQC signals) and naphthalenes (F1 ≈ 0.66, attributed to an overly broad structural category). Write per-class precision, recall, and F1 to per_class_metrics.csv, annotating the best- and worst-predicted classes.

## Related tools

- **DeepSAT** (Source of the multiclass classification predictions (NPClassifier superclass head trained on 143,467 1H-13C HSQC spectra) whose confusion matrix is decomposed into per-class metrics) — https://github.com/mwang87/DeepSAT

## Examples

```
from sklearn.metrics import precision_recall_fscore_support; P,R,F,_ = precision_recall_fscore_support(y_true, y_pred, labels=range(59), zero_division=0)
```

## Evaluation signals

- Recomputed F1 for diazotetronic acids and derivatives reproduces ≈0.64 and for naphthalenes ≈0.66
- Per-class F1 values remain consistent with the reported top-1 class accuracy of 90.4% (normal HSQC) and 90.8% (edited HSQC)
- Output CSV covers exactly the 59 NPClassifier superclass labels with non-null precision and recall
- Each F1 equals the harmonic mean of its own precision and recall (F1 = 2*P*R/(P+R)), checkable row-wise in per_class_metrics.csv
- Ranked worst classes are explained by identifiable causes (proton-deficient scaffolds with sparse HSQC peaks; overly broad categories)

## Limitations

- Low F1 can stem from different causes per class: proton-deficient structures like diazotetronic acids yield few 1H-13C HSQC signals, while overly broad categories like naphthalenes dilute class-specific features — causes must be interpreted individually
- Detailed per-class precision/recall/F1 values were deferred to supplementary tables/figures rather than the main text
- Metrics apply to the NPClassifier superclass classification head only; fingerprint cosine (average 0.8450 normal / 0.8574 edited) and molecular-weight R² (0.9183 / 0.9336) are separate evaluation tasks

## Evidence

- [results] diazotetronic acids and derivatives (F1-score of 0.64) and naphthalenes (F1-score of 0.66), showed low values: "diazotetronic acids and derivatives (F1-score of 0.64) and naphthalenes (F1-score of 0.66), showed low values"
- [results] Top-1 class prediction accuracy 90.4/90.8: "the Top-1 accuracy of normal and Edited HSQC models were 90.4% and 90.8%, respectively"
- [readme] DeepSAT API input format: "You can put in your peaks as a json list of dicts, with 1H,13C as headers."
