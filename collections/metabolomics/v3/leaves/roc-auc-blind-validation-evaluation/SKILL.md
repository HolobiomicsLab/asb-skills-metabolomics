---
name: roc-auc-blind-validation-evaluation
description: Use when a binary classifier (e.g., qualified vs unqualified LC or MS
  QC labels from DIA features) must be evaluated on held-out blinded data and reported
  as a single ROC AUC per model.
when_to_use_negative:
- Training-time performance is needed instead of held-out generalization — cross-validation
  AUC on the 2110 training files is not a substitute for the blinded evaluation described
  here.
- The task is DDA- or SRM-based QC rather than DIA — the paper states prior SRM/DDA
  QC schemes cannot be directly applied to large-scale DIA experiments and this skill
  is scoped to DIA features.
- Validation labels are unavailable or the model has already seen the evaluation files;
  blinding is essential to the AUC claim and is broken otherwise.
edam_operation: http://edamontology.org/operation_3791
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_3474
tools:
- name: iDIA-QC
  role: End-to-end pipeline whose machine-learning module trains the XGBoost LC/MS
    QC models and applies them, blinded, to validation sets to compute the reported
    AUCs.
  repo: github.com/guomics-lab/iDIA-QC
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-024-54871-1
    title: 'iDIA-QC: AI-empowered data-independent acquisition mass spectrometry-based
      quality control'
  source_content_hash: sha256:2090beb4c449207efeb01f5213cd08ac6ba757d31a8d1149990e5312734eb5ee
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/guomics-lab/iDIA-QC
  tools:
  - iDIA-QC
  edam_operation: http://edamontology.org/operation_3791
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# roc-auc-blind-validation-evaluation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Evaluate trained QC classifiers by applying them, blinded, to held-out DIA validation sets and reporting ROC AUC per model. In the iDIA-QC workflow, XGBoost LC and MS QC models trained on 2110 expert-labeled DIA files are scored on independent validation sets (n=528 and n=116) with reported AUCs of 0.91/0.97 (set 1) and 0.78/0.94 (set 2).

## When to use

Use when a binary classifier (e.g., qualified vs unqualified LC or MS QC labels from DIA features) must be evaluated on held-out blinded data and reported as a single ROC AUC per model.

## When NOT to use

- Training-time performance is needed instead of held-out generalization — cross-validation AUC on the 2110 training files is not a substitute for the blinded evaluation described here.
- The task is DDA- or SRM-based QC rather than DIA — the paper states prior SRM/DDA QC schemes cannot be directly applied to large-scale DIA experiments and this skill is scoped to DIA features.
- Validation labels are unavailable or the model has already seen the evaluation files; blinding is essential to the AUC claim and is broken otherwise.

## Inputs

- Fitted XGBoost LC QC classifier and MS QC classifier (trained via five-fold cross-validation on 2110 DIA files)
- Feature matrices for validation set 1 (n=528 DIA files) and validation set 2 (n=116 QE HF files) built from extracted precursor-related metrics (F2, F13, F14, F1, F3–F12)
- Expert-inspected qualified/unqualified ground-truth labels for the validation files (used only for scoring, not training)

## Outputs

- ROC AUC value for the LC QC model on each validation set (expected 0.91 on set 1, 0.78 on set 2)
- ROC AUC value for the MS QC model on each validation set (expected 0.97 on set 1, 0.94 on set 2)
- Results table of validation AUCs and the two saved fitted XGBoost model files

## How to apply

After training each gradient-boosted (XGBoost) classifier with five-fold cross-validation on the 2110-file training set, freeze the fitted model and apply it, blinded, to validation set 1 (n=528 DIA files) and validation set 2 (n=116 QE HF files). Compute the ROC AUC for the LC model and separately for the MS model on each set; the paper treats the reported AUCs (0.91 LC / 0.97 MS on set 1; 0.78 LC / 0.94 MS on set 2) as the benchmark the evaluation must match. Keep the validation predictions blind — the expert qualified/unqualified labels are only used to score the ROC after all predictions are produced. Save both the fitted model files and a results table of validation AUCs so the evaluation is reproducible.

## Related tools

- **iDIA-QC** (End-to-end pipeline whose machine-learning module trains the XGBoost LC/MS QC models and applies them, blinded, to validation sets to compute the reported AUCs.) — github.com/guomics-lab/iDIA-QC

## Examples

```
python run_iDIAQC.py --input validation_set1.raw --instrument orbitrap --model lc_ms_xgb --output report/
```

## Evaluation signals

- LC model ROC AUC on validation set 1 (n=528) reproduces 0.91
- MS model ROC AUC on validation set 1 (n=528) reproduces 0.97
- LC model ROC AUC on validation set 2 (n=116) reproduces 0.78
- MS model ROC AUC on validation set 2 (n=116) reproduces 0.94
- Predictions on the validation sets were produced without access to expert qualified/unqualified labels during scoring (blinded protocol maintained)

## Limitations

- The two validation sets differ in instrument coverage (set 2 is QE HF only), so AUCs are not directly comparable across sets.
- Generalization to newly emerging LC/MS instruments was not covered in the study and 'further investigation is needed to fully generalize its application'.
- The paper's discussion does not describe hyperparameters or the exact train/test split composition, so reproducing the exact AUCs may require the authors' saved model files.

## Evidence
