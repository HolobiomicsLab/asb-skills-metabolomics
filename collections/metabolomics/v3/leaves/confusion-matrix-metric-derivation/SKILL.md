---
name: confusion-matrix-metric-derivation
description: Use when you have a trained multi-class classifier's confusion matrix
  (e.g., DeepSAT's 59 NPClassifier superclass predictions on 143,467 HSQC spectra)
  and need per-class precision/recall/F1 to diagnose weak classes.
when_to_use_negative:
- The classifier output is a similarity search (e.g., cosine score over 0.8 against
  Morgan fingerprints) rather than a discrete class prediction, so no confusion matrix
  exists.
- The evaluation target is a regression task such as molecular weight prediction (R-squared
  0.9183/0.9336) or fingerprint prediction (cosine 0.8450/0.8574), where per-class
  F1 does not apply.
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_0154
tools:
- name: DeepSAT
  role: CNN-based multi-task model whose 59-class NPClassifier classification head
    generates the predictions and confusion matrix analyzed here
  repo: github.com/mwang87/DeepSAT
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1186/s13321-023-00738-4
    title: 'DeepSAT: Learning Molecular Structures from Nuclear Magnetic Resonance
      Data'
  source_content_hash: sha256:1963390c461df543719b7be996e3a1266fcbce8a4f84491ea3d9b4d52fe90ddd
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - DeepSAT
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 5
---

# confusion-matrix-metric-derivation

## Summary

Derive per-class precision, recall, and F1-scores from the confusion matrix of DeepSAT's 59-class NPClassifier superclass prediction head, and rank classes to identify the weakest performers (e.g., diazotetronic acids and derivatives F1 ≈ 0.64; naphthalenes F1 ≈ 0.66).

## When to use

Use when you have a trained multi-class classifier's confusion matrix (e.g., DeepSAT's 59 NPClassifier superclass predictions on 143,467 HSQC spectra) and need per-class precision/recall/F1 to diagnose weak classes.

## When NOT to use

- The classifier output is a similarity search (e.g., cosine score over 0.8 against Morgan fingerprints) rather than a discrete class prediction, so no confusion matrix exists.
- The evaluation target is a regression task such as molecular weight prediction (R-squared 0.9183/0.9336) or fingerprint prediction (cosine 0.8450/0.8574), where per-class F1 does not apply.

## Inputs

- Confusion matrix (59 NPClassifier superclass labels x 59 labels) from DeepSAT evaluation
- Class-level prediction results (predicted vs. ground-truth superclass) for 143,467 1H-13C HSQC spectra

## Outputs

- per_class_metrics.csv with per-class precision, recall, and F1 for all 59 NPClassifier superclasses
- Ranked list of weakest-performing classes (e.g., diazotetronic acids and derivatives F1 0.64; naphthalenes F1 0.66)

## How to apply

Load the 59-class NPClassifier superclass confusion matrix produced during evaluation of DeepSAT models trained on 143,467 1H-13C HSQC spectra (normal and multiplicity-edited HSQC). For each class label, compute precision as TP/(TP+FP) and recall as TP/(TP+FN) directly from the confusion-matrix counts, then derive F1 = 2*P*R/(P+R). Rank all 59 classes by F1 and flag the weakest performers, verifying that diazotetronic acids and derivatives score ~0.64 and naphthalenes ~0.66, and that the aggregate metrics remain consistent with the reported top-1 class accuracies of 90.4% (normal HSQC) and 90.8% (edited HSQC). Write per-class precision, recall, and F1 values, with best- and worst-predicted classes annotated, to per_class_metrics.csv. Rationale: per-class F1 exposes proton-deficient or overly broad structural categories that aggregate top-1 accuracy masks.

## Related tools

- **DeepSAT** (CNN-based multi-task model whose 59-class NPClassifier classification head generates the predictions and confusion matrix analyzed here) — github.com/mwang87/DeepSAT

## Examples

```
curl -d @peaks.json -H 'Content-Type: application/json' https://deepsat.ucsd.edu/api/smart3/search  # then recompute per-class F1 from the 59-class confusion matrix
```

## Evaluation signals

- Recomputed per-class F1 reproduces the reported worst-case values: diazotetronic acids and derivatives ≈ 0.64 and naphthalenes ≈ 0.66.
- Macro/micro aggregation of per-class metrics is consistent with reported top-1 class accuracy of 90.4% (normal HSQC) and 90.8% (edited HSQC).
- Exactly 59 class rows appear in per_class_metrics.csv, each with non-null precision, recall, and F1.
- F1 values satisfy F1 ≤ min(2*precision, 2*recall) / (precision + recall) properties for every row; harmonic-mean recomputation from stored P and R matches the stored F1.
- The ranked worst classes are explainable: proton-deficient classes (e.g., diazotetronic acids) and overly broad categories (e.g., naphthalenes) rank lowest.

## Limitations

- Per-class metrics were not reported in the article's main text (deferred to supplementary tables/figures), so worst-class values may only be verifiable against supplementary data.
- Low F1 for some classes is attributed to proton-deficient nature (diazotetronic acids) or overly broad structural categories (naphthalenes) — properties of the ontology itself, not fixable by retraining alone.
- Class imbalance and the broad NPClassifier category definitions can depress F1 independent of model quality; interpret low scores with this in mind.

## Evidence

- [results] Diazotetronic acids and derivatives had F1-score 0.64 and naphthalenes 0.66 in classification: "diazotetronic acids and derivatives (F1-score of 0.64) and naphthalenes (F1-score of 0.66), showed low values"
- [results] Top-1 accuracy reported for both HSQC models: "the Top-1 accuracy of normal and Edited HSQC models were 90.4% and 90.8%, respectively"
- [intro] DeepSAT trained on 143,467 HSQC spectra with class prediction head: "DeepSAT was trained using a convolutional neural network (CNN)-based multi-task supervised learning architecture with 143,467 1H-13C HSQC spectra"
- [discussion] Per-class precision/recall/F1 not reported in main text: [evidence span withheld — anchor status: not_found]
- [readme] README documents the classification API for DeepSAT (SMART 3): "Classify programmatically ```/api/smart3/search```"
