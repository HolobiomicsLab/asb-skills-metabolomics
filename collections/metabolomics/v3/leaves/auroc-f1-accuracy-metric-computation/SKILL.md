---
name: auroc-f1-accuracy-metric-computation
description: Use when meta-evaluating or comparing trained classifiers on held-out
  SCMS tasks (e.g., 1-, 5-, 10-shot MAML adaptation) and quantitative, threshold-free
  metrics are needed to select a model or report performance.
when_to_use_negative:
- Model is still mid-training and has not completed the 50 total epochs used in the
  paper — metrics would not be comparable to reported values
- Evaluation goal is feature/metabolite selection (essential species identification)
  rather than classification performance measurement
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_0121
tools:
- name: MetaPhenotype
  role: MAML-based meta-learning model whose meta-evaluation outputs (per-task predictions
    on held-out SCMS tasks) are scored with AUROC, F1, and accuracy
  repo: github.com/songyuan93/MetaPhenotype
- name: SCMS metabolomics
  role: Source single-cell mass spectrometry data of IGR39/IGR37 melanoma cells on
    which classification metrics are computed
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/acs.analchem.4c02038
    title: 'MetaPhenotype: A Transferable Meta-Learning Model for Single-Cell Mass
      Spectrometry-Based Cell Phenotype Prediction Using Limited Number of Cells'
  source_content_hash: sha256:da5952fb9ddcb725f5d07f7666a05a3dd5499ae5e57c2220dc60a2f9113b9ff9
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - MetaPhenotype
  - SCMS metabolomics
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# auroc-f1-accuracy-metric-computation

## Summary

Evaluate cell-phenotype classifiers (e.g., MetaPhenotype trained on SCMS metabolomics data) by computing AUROC, F1 score, and prediction accuracy after 50 total epochs of training, enabling fair comparison across backbones and shot settings.

## When to use

Use when meta-evaluating or comparing trained classifiers on held-out SCMS tasks (e.g., 1-, 5-, 10-shot MAML adaptation) and quantitative, threshold-free metrics are needed to select a model or report performance.

## When NOT to use

- Model is still mid-training and has not completed the 50 total epochs used in the paper — metrics would not be comparable to reported values
- Evaluation goal is feature/metabolite selection (essential species identification) rather than classification performance measurement

## Inputs

- Trained MetaPhenotype model checkpoint (meta.py) for a given 1D-CNN backbone (Lecun, LeNet, or VGG9) and shot setting
- Held-out query/support splits from pretreated SCMS metabolomics data of the untreated IGR39/IGR37 melanoma cell pair (dataset.csv)

## Outputs

- Per-task AUROC, F1 score, and prediction accuracy values
- Aggregated results table across backbones and shot settings (e.g., meta_training_results.csv)

## How to apply

After meta-training (e.g., MetaPhenotype MAML with a LeNet 1D-CNN backbone, SGD+Adam outer optimizer at lr 1e-3, 50 epochs with 5 inner gradient-descent cycles per task), run the trained initialization on held-out IGR39/IGR37 query tasks. For each backbone and shot setting (1, 5, or 10 shots per class), compute AUROC, F1 score, and prediction accuracy after 50 total epochs of training. Aggregate all metric values across backbones and shot settings into a single results table (e.g., meta_training_results.csv) so that backbone selection (LeNet was chosen over Lecun and VGG9 for efficiency and higher 5-shot accuracy) and reproduction checks against the reported >90% classification accuracy can be performed.

## Related tools

- **MetaPhenotype** (MAML-based meta-learning model whose meta-evaluation outputs (per-task predictions on held-out SCMS tasks) are scored with AUROC, F1, and accuracy) — github.com/songyuan93/MetaPhenotype
- **SCMS metabolomics** (Source single-cell mass spectrometry data of IGR39/IGR37 melanoma cells on which classification metrics are computed)

## Examples

```
python base.py  # train/evaluate MetaPhenotype on dataset.csv and score AUROC, F1, accuracy
```

## Evaluation signals

- LeNet-based MetaPhenotype reaches the reported high (>90%) classification accuracy with limited new training samples (1/5/10-shot)
- LeNet outperforms VGG9 in the 5-shot setting while being roughly comparable in the 1- and 10-shot settings
- All AUROC/F1/accuracy values are aggregated across backbones and shot settings into one results table (meta_training_results.csv) with no missing metric cells
- Metrics are computed after exactly 50 total epochs of training, matching the paper's evaluation protocol

## Limitations

- Accuracy is reported after 50 total epochs of training; earlier-epoch metrics may not reproduce the paper's numbers
- Metrics were established on IGR39/IGR37 melanoma SCMS data with few cells per task (ambient SCMS yields relatively small cell numbers), so performance estimates can be noisy at 1-shot
- Conventional models can overfit when limited data are available, so metric comparisons must hold shot counts and backbone fixed to be meaningful

## Evidence

- [intro] we computed the area under the receiver operating characteristic curve (AUROC), F-1 score, and prediction accuracy after 50 total epochs of training: "we computed the area under the receiver operating characteristic curve (AUROC), F-1 score, and prediction accuracy after 50 total epochs of training"
- [abstract] MetaPhenotype can rapidly predict and achieve a high accuracy of over 90% with fewer new training samples: "MetaPhenotype can rapidly predict and achieve a high accuracy of over 90% with fewer new training samples"
- [other] LeNet was selected as the MetaPhenotype backbone for its efficiency and higher accuracy: [evidence span withheld — anchor status: not_found]
- [readme] python base.py: "Using single cell mass spec dataset to run the MetaPhenotype model.
```
python base.py
```"
