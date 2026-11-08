---
name: chemical-superclass-classification-evaluation
description: Use when evaluating a chemical structure-class classifier (e.g., DeepSAT
  on HSQC spectra) and you need per-superclass F1-scores to identify the lowest-performing
  NPClassifier structure classes and their causes.
when_to_use_negative:
- You need whole-model summary metrics (top-1 accuracy, fingerprint cosine score,
  MW R-squared) rather than per-structure-class breakdowns.
- The classifier output is a molecular fingerprint for similarity search (cosine >
  0.8) rather than discrete NPClassifier superclass labels.
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_3292
tools:
- name: DeepSAT
  role: CNN-based multi-task model whose 59-class NPClassifier superclass prediction
    head is evaluated per-class in this skill; code available at github.com/mwang87/DeepSAT
    (MIT license, models and ontology CC0)
  repo: github.com/mwang87/DeepSAT
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1186/s13321-023-00738-4
    title: 'DeepSAT: Learning Molecular Structures from Nuclear Magnetic Resonance
      Data'
  source_content_hash: sha256:00bf320e05084d0e5d4f2d5ed0653bd5e2db0778d552bf6fb23792f0fdfc42eb
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - DeepSAT
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_3292
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# chemical-superclass-classification-evaluation

## Summary

Computes per-class precision, recall, and F1-scores for each of the 59 NPClassifier superclass labels predicted by a multi-task CNN (DeepSAT) trained on 143,467 1H-13C HSQC spectra, ranking classes to expose the weakest structural categories and diagnosing why they underperform.

## When to use

Use when evaluating a chemical structure-class classifier (e.g., DeepSAT on HSQC spectra) and you need per-superclass F1-scores to identify the lowest-performing NPClassifier structure classes and their causes.

## When NOT to use

- You need whole-model summary metrics (top-1 accuracy, fingerprint cosine score, MW R-squared) rather than per-structure-class breakdowns.
- The classifier output is a molecular fingerprint for similarity search (cosine > 0.8) rather than discrete NPClassifier superclass labels.

## Inputs

- 59-class NPClassifier superclass confusion matrix from DeepSAT evaluation on 1H-13C HSQC spectra
- Class-level prediction results (TP/FP/FN counts per superclass label) from the normal and multiplicity edited HSQC models

## Outputs

- per_class_metrics.csv with per-class precision, recall, and F1-score for all 59 NPClassifier superclasses, with best- and worst-predicted classes annotated

## How to apply

Load the 59-class NPClassifier superclass confusion matrix or class-level predictions generated during evaluation of DeepSAT models trained on 143,467 1H-13C HSQC spectra. For each superclass label, compute precision as TP/(TP+FP) and recall as TP/(TP+FN) directly from the confusion-matrix counts, then derive F1 as the harmonic mean F1 = 2*P*R/(P+R). Rank all 59 classes by F1 and flag the weakest performers, verifying that diazotetronic acids and derivatives score approximately 0.64 and naphthalenes approximately 0.66. Interpret the failure modes: diazotetronic acids and derivatives suffer from their proton-deficient nature (few HSQC-visible signals), while naphthalenes score low because the structural category is overly broad. Write per_class_metrics.csv with per-class precision, recall, and F1, annotating the best- and worst-predicted classes, and sanity-check consistency against the reported top-1 class accuracies of 90.4% (normal HSQC) and 90.8% (multiplicity edited HSQC).

## Related tools

- **DeepSAT** (CNN-based multi-task model whose 59-class NPClassifier superclass prediction head is evaluated per-class in this skill; code available at github.com/mwang87/DeepSAT (MIT license, models and ontology CC0)) — github.com/mwang87/DeepSAT

## Examples

```
python eval_per_class.py --confusion_matrix deepsat_59class_cm.csv --labels npclassifier_superclasses.txt --out per_class_metrics.csv
```

## Evaluation signals

- Recomputed per-class F1 for diazotetronic acids and derivatives must reproduce approximately 0.64.
- Recomputed per-class F1 for naphthalenes must reproduce approximately 0.66.
- Per-class metrics must remain consistent with reported top-1 class accuracy of 90.4% (normal HSQC) and 90.8% (edited HSQC).
- per_class_metrics.csv must contain exactly 59 superclass rows with finite precision, recall, and F1 values (harmonic mean of precision and recall).
- The weakest-ranked classes should be explainable (e.g., proton-deficient diazotetronic acids; overly broad naphthalene category), matching the article's interpretation.

## Limitations

- Diazotetronic acids and derivatives score low (F1 ~0.64) partly because they are proton-deficient, so few 1H-13C HSQC signals are available to the model; other proton-deficient classes may share this failure mode.
- Naphthalenes score low (F1 ~0.66) partly because the structural category is overly broad, inflating within-class diversity; broad NPClassifier categories generally depress per-class F1.
- Interpretability analyses in the article (occlusion sensitivity) were limited to a single compound (quercetin) and 'do not fully explain the decision-making process of DeepSAT'.

## Evidence

- [results] Per-class F1 from confusion matrix: "diazotetronic acids and derivatives (F1-score of 0.64) and naphthalenes (F1-score of 0.66), showed low values"
- [results] Top-1 class accuracy context: "the Top-1 accuracy of normal and Edited HSQC models were 90.4% and 90.8%, respectively"
- [intro] Training scale and architecture: "DeepSAT was trained using a convolutional neural network (CNN)-based multi-task supervised learning architecture with 143,467 1H-13C HSQC spectra"
- [readme] README: classification API and licensing: "You can put in your peaks as a json list of dicts, with 1H,13C as headers."
