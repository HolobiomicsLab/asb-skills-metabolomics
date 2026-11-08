---
name: meta-learning-model-adaptation
description: Use when SCMS single-cell metabolomic peak tables are limited (1–10 adaptation
  samples per class) and a meta-trained model must be transferred via few-shot gradient
  descent to classify new cell groups (e.g., drug-treated IGR39/IGR37 or WM115/WM266-4
  melanoma cells) as primary vs metastatic.
when_to_use_negative:
- A large number of labeled training cells is available, so standard transfer learning
  or conventional deep learning is adequate and MAML's inner/outer loop overhead is
  unnecessary.
- The input is bulk metabolomics data or an uncurated raw spectral file that has not
  gone through the SCMS pretreatment protocol (noise/background subtraction, TIC normalization,
  peak alignment).
- The goal is metabolite identification or annotation (METLIN/HMDB labeling) rather
  than phenotype classification.
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_0091
tools:
- name: MetaPhenotype
  role: MAML meta-learning model with 1D-CNN (LeNet) backbone adapted to unseen cell
    groups via few-shot inner-loop gradient descent
  repo: github.com/songyuan93/MetaPhenotype
- name: Single-probe SCMS
  role: Source technique producing the single-cell metabolomic profiles used for adaptation
    and evaluation
provenance:
  source_task_ids:
  - task_002
  - task_006
  source_papers:
  - doi: 10.1021/acs.analchem.4c02038
    title: 'MetaPhenotype: A Transferable Meta-Learning Model for Single-Cell Mass
      Spectrometry-Based Cell Phenotype Prediction Using Limited Number of Cells'
  source_content_hash: sha256:6405ed4eaa90b145b6443070dd1e27bfbf2982a66e7f7179e72a92a25adec643
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - GitHub (MetaPhenotype repository)
  - MetaPhenotype
  - Single-probe SCMS
  edam_operation: http://edamontology.org/operation_2990
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# meta-learning-model-adaptation

## Summary

Rapidly adapt the MAML-based MetaPhenotype model (1D-CNN/LeNet backbone) to previously unseen single-cell mass spectrometry (SCMS) cell groups with only 1- or 5-shot adaptation samples, classifying melanoma cells into primary vs metastatic phenotypes at >90% accuracy.

## When to use

Use when SCMS single-cell metabolomic peak tables are limited (1–10 adaptation samples per class) and a meta-trained model must be transferred via few-shot gradient descent to classify new cell groups (e.g., drug-treated IGR39/IGR37 or WM115/WM266-4 melanoma cells) as primary vs metastatic.

## When NOT to use

- A large number of labeled training cells is available, so standard transfer learning or conventional deep learning is adequate and MAML's inner/outer loop overhead is unnecessary.
- The input is bulk metabolomics data or an uncurated raw spectral file that has not gone through the SCMS pretreatment protocol (noise/background subtraction, TIC normalization, peak alignment).
- The goal is metabolite identification or annotation (METLIN/HMDB labeling) rather than phenotype classification.

## Inputs

- Meta-trained MetaPhenotype checkpoint (meta.py model)
- Pretreated SCMS metabolomics feature matrix (dataset.csv) for unseen cell groups
- Small adaptation sample set (1-shot or 5-shot) from each unseen cell group
- Held-out evaluation cells from the same unseen groups

## Outputs

- Adapted MetaPhenotype model per cell group and shot setting
- Per-group phenotype predictions (primary vs metastatic)
- Performance metrics: AUROC, F-1 score, and prediction accuracy after 50 total epochs
- Comparison table of AUROC/accuracy vs baseline transfer-learning methods (Figure 3 / Table S6 analogue)

## How to apply

Load the meta-trained MetaPhenotype checkpoint (code in meta.py of github.com/songyuan93/MetaPhenotype) and the pretreated SCMS feature matrix (dataset.csv, read by dataloade.py), which must have undergone noise/background subtraction, TIC normalization, and peak alignment (Geena2, 10 ppm tolerance). Perform MAML inner-loop gradient-descent adaptation of the 1D-CNN (LeNet) backbone on the small adaptation sample set for each unseen cell group (drug-treated IGR39/IGR37, untreated/treated WM115/WM266-4) under 1-shot and 5-shot settings. Rationale: ambient SCMS yields relatively few cells, so a MAML model trained on one primary/metastatic pair transfers to new tasks within a small number of optimization steps rather than retraining from scratch. After adaptation, train for 50 total epochs and evaluate on held-out cells of each group, computing AUROC, F-1 score, and prediction accuracy. Aggregate per-method results across the three unseen cell groups and both shot settings and compare against baseline transfer-learning models to confirm adaptation accuracy exceeds 90% with the limited sample counts.

## Related tools

- **MetaPhenotype** (MAML meta-learning model with 1D-CNN (LeNet) backbone adapted to unseen cell groups via few-shot inner-loop gradient descent) — github.com/songyuan93/MetaPhenotype
- **Single-probe SCMS** (Source technique producing the single-cell metabolomic profiles used for adaptation and evaluation)

## Examples

```
python base.py
```

## Evaluation signals

- Adapted MetaPhenotype prediction accuracy on unseen cell groups (treated IGR39/IGR37, untreated/treated WM115/WM266-4) exceeds 90% with only five adaptation training steps.
- AUROC, F-1 score, and accuracy are computed after exactly 50 total epochs of training, matching the paper's evaluation protocol.
- Reported metrics for 1-shot and 5-shot settings reproduce the paper's trend that MetaPhenotype outperforms baseline transfer-learning models at equal sample counts.
- Run structure matches the README: dataloade.py reads dataset.csv and meta.py contains the MetaPhenotype model, executed via python base.py.
- Per-cell-group AUROC/accuracy values across the three unseen groups and both shot settings aggregate into a table matching Figure 3 / Table S6.

## Limitations

- MAML requires multiple inner-loop gradient descent steps and backpropagation through the updates in the outer loop, making it memory and computationally intensive for larger models.
- MAML is sensitive to hyperparameters and less effective in nonstationary environments.
- Conventional machine learning models can overfit when limited data are available; ambient SCMS methods inherently yield relatively small cell numbers, so sample size remains a bottleneck.

## Evidence

- [readme] Repository run entry point: [evidence span withheld — anchor status: external_doc]
