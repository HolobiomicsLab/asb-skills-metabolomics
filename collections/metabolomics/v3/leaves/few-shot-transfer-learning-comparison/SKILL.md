---
name: few-shot-transfer-learning-comparison
description: Use when few (1-5) labeled single-cell mass spectrometry (SCMS) samples
  are available for a new cell group and you need to adapt a pre-trained meta-learning
  classifier and compare it with transfer-learning baselines.
when_to_use_negative:
- PCA already discriminates the cell groups — the article notes PCA cannot separate
  phenotypically similar metabolomic profiles such as IGR39/IGR37, so if simpler methods
  suffice, meta-learning is unnecessary
- Large labeled training sets are available for the new cell group — conventional
  deep learning is appropriate and the MAML inner/outer loop overhead is not warranted
- Data are bulk (population-level) measurements rather than single-cell metabolomic
  profiles, since the model is trained on SCMS single-cell feature tables
edam_operation: http://edamontology.org/operation_3936
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3474
tools:
- name: MetaPhenotype
  role: MAML-based meta-learning model with 1D-CNN (LeNet) backbone adapted to unseen
    cell groups in 1- or 5-shot settings
  repo: https://github.com/songyuan93/MetaPhenotype
- name: Single-probe SCMS
  role: Technique that produced the single-cell metabolomics profiles of the melanoma
    cell pairs used for adaptation
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1021/acs.analchem.4c02038
    title: 'MetaPhenotype: A Transferable Meta-Learning Model for Single-Cell Mass
      Spectrometry-Based Cell Phenotype Prediction Using Limited Number of Cells'
  source_content_hash: sha256:938277e78551367acdea469bf78932e4d8d0077b5970729496f17b7ec6d8bfec
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/songyuan93/MetaPhenotype
  tools:
  - MetaPhenotype
  - Single-probe SCMS
  edam_operation: http://edamontology.org/operation_3936
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# few-shot-transfer-learning-comparison

## Summary

Rapidly adapt a meta-trained MetaPhenotype model (MAML with a 1D-CNN/LeNet backbone) to classify previously unseen melanoma cell groups into primary vs metastatic phenotypes using only 1- or 5-shot adaptation, and benchmark against standard transfer-learning baselines.

## When to use

Use when few (1-5) labeled single-cell mass spectrometry (SCMS) samples are available for a new cell group and you need to adapt a pre-trained meta-learning classifier and compare it with transfer-learning baselines.

## When NOT to use

- PCA already discriminates the cell groups — the article notes PCA cannot separate phenotypically similar metabolomic profiles such as IGR39/IGR37, so if simpler methods suffice, meta-learning is unnecessary
- Large labeled training sets are available for the new cell group — conventional deep learning is appropriate and the MAML inner/outer loop overhead is not warranted
- Data are bulk (population-level) measurements rather than single-cell metabolomic profiles, since the model is trained on SCMS single-cell feature tables

## Inputs

- MetaPhenotype meta-trained checkpoint (meta.py model code)
- Pretreated SCMS metabolomics data as dataset.csv from the Input data folder (unseen cell groups: treated IGR39/IGR37, untreated/treated WM115/WM266-4)
- Small adaptation sample sets (1- or 5-shot) per new task
- Baseline transfer-learning model configurations for comparison

## Outputs

- Adapted model predictions for held-out cells of each unseen group
- Per-method AUROC, F-1 score, and accuracy values after 50 epochs
- Comparison table of AUROC/accuracy across three cell groups and both shot settings (Figure 3 / Table S6)

## How to apply

Load the meta-trained MetaPhenotype checkpoint and the pretreated SCMS metabolomics data (peaks exported, noise/background subtracted, TIC-normalized, peak-aligned with Geena2 at 10 ppm mass tolerance, missing values removed) for each unseen cell group (treated IGR39/IGR37, untreated and treated WM115/WM266-4). For each shot setting (1-shot, 5-shot), perform MAML inner-loop gradient-descent adaptation of the LeNet 1D-CNN backbone on the small adaptation sample set, then evaluate on held-out cells after 50 total epochs of training. Compute AUROC, F-1 score, and prediction accuracy, and run the identical protocol for the paper's standard transfer-learning baseline methods. Aggregate per-method AUROC and accuracy across the three cell groups and both shot settings into a comparison table matching Figure 3 and Table S6, validating that MetaPhenotype exceeds 90% accuracy with the limited samples.

## Related tools

- **MetaPhenotype** (MAML-based meta-learning model with 1D-CNN (LeNet) backbone adapted to unseen cell groups in 1- or 5-shot settings) — https://github.com/songyuan93/MetaPhenotype
- **Single-probe SCMS** (Technique that produced the single-cell metabolomics profiles of the melanoma cell pairs used for adaptation)

## Examples

```
python base.py
```

## Evaluation signals

- MetaPhenotype adaptation accuracy exceeds 90% for treated IGR39/IGR37 and untreated/treated WM115/WM266-4 after only five adaptation training steps
- AUROC, F-1, and accuracy are computed after 50 total epochs of training on held-out cells of each group
- MetaPhenotype reaches >90% accuracy with fewer new training samples than the standard transfer-learning baselines run under the same protocol
- Per-method results aggregated across the three cell groups and both shot settings reproduce the pattern in Figure 3 and Table S6

## Limitations

- MAML is memory and computationally intensive for larger backbones due to multiple inner-loop gradient steps and backpropagation through updates, and it is sensitive to hyperparameters and less effective in nonstationary environments
- Ambient SCMS methods yield relatively small cell numbers, so adaptation performance depends on the limited sample counts available
- Conventional machine learning models can overfit when limited data are available for training; the meta-learning approach is designed specifically for this regime

## Evidence

- [results] In meta-evaluation, MetaPhenotype's prediction accuracy for untreated/treated WM115/WM266-4 and treated IGR39/IGR37 cells exceeded 90% after only five adaptation training steps on unseen cell groups: [evidence span withheld — anchor status: not_found]
- [readme] compared with standard transfer learning models, MetaPhenotype can more rapidly achieve a high accuracy of over 90% with fewer new training samples: [evidence span withheld — anchor status: external_doc]
- [readme] The code for MetaPhenotype model were included in meta.py file: [evidence span withheld — anchor status: external_doc]
