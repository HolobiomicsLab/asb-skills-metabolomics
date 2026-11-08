---
name: melanoma-primary-vs-metastatic-classification
description: Use when you have pretreated SCMS metabolomics feature data (e.g., dataset.csv)
  from a primary/metastatic melanoma cell pair, PCA fails to discriminate the phenotypes,
  and only limited cells are available for training a classifier.
when_to_use_negative:
- Input is raw, unprocessed single-cell spectra — the model expects pretreated data
  (noise/background subtraction, TIC normalization, 10 ppm peak alignment, missing-value
  removal)
- Cell populations whose metabolomic profiles are well separated by simpler methods
  such as PCA, where a deep meta-learning model is unnecessary
- Tasks with abundant training cells where standard supervised or transfer-learning
  models do not suffer from limited-sample overfitting — note MAML is memory/computationally
  intensive and sensitive to hyperparameters
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_3473
- http://edamontology.org/topic_0092
tools:
- name: MetaPhenotype
  role: MAML-based meta-learning model (meta.py, base.py) with 1D-CNN LeNet backbone
    used for phenotype classification
  repo: github.com/songyuan93/MetaPhenotype
- name: Single-probe SCMS
  role: Acquires the single-cell metabolomic profiles of melanoma cell lines that
    serve as model input
- name: Geena2
  role: Online tool for peak alignment of SCMS peaks with a 10 ppm mass tolerance
    during data pretreatment
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/acs.analchem.4c02038
    title: 'MetaPhenotype: A Transferable Meta-Learning Model for Single-Cell Mass
      Spectrometry-Based Cell Phenotype Prediction Using Limited Number of Cells'
  source_content_hash: sha256:0b5024fb335b934d9fc51f3123b9c714b21fc924ea3a864b38db0bad2e515719
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Geena2
  - MetaPhenotype
  - Single-probe SCMS
  edam_operation: http://edamontology.org/operation_2990
  edam_topics:
  - http://edamontology.org/topic_3473
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# melanoma-primary-vs-metastatic-classification

## Summary

Classifies single melanoma cells (e.g., IGR39 primary vs IGR37 metastatic) into primary or metastatic phenotypes from Single-probe SCMS metabolomics data using the MAML-based MetaPhenotype meta-learning model with a 1D-CNN (LeNet) backbone, achieving >90% accuracy from limited training samples.

## When to use

Use when you have pretreated SCMS metabolomics feature data (e.g., dataset.csv) from a primary/metastatic melanoma cell pair, PCA fails to discriminate the phenotypes, and only limited cells are available for training a classifier.

## When NOT to use

- Input is raw, unprocessed single-cell spectra — the model expects pretreated data (noise/background subtraction, TIC normalization, 10 ppm peak alignment, missing-value removal)
- Cell populations whose metabolomic profiles are well separated by simpler methods such as PCA, where a deep meta-learning model is unnecessary
- Tasks with abundant training cells where standard supervised or transfer-learning models do not suffer from limited-sample overfitting — note MAML is memory/computationally intensive and sensitive to hyperparameters

## Inputs

- Pretreated SCMS metabolomics feature table (dataset.csv) of primary/metastatic melanoma cells, e.g., untreated IGR39/IGR37 acquired on a Thermo LTQ Orbitrap XL
- MAML meta-training task definitions with support/query splits at 1, 5, and 10 shots per class
- Trained MetaPhenotype initialization for transfer to new cell groups (drug-treated cells, unseen cell-line pair)

## Outputs

- Per-cell phenotype labels (primary vs metastatic)
- Meta-evaluation metrics: AUROC, F1 score, and prediction accuracy after 50 total epochs (aggregated as meta_training_results.csv across backbones and shot settings)
- Selected backbone model: LeNet-based MAML initialization

## How to apply

Pretreat the SCMS data (peak export, noise/background subtraction, TIC normalization, Geena2 peak alignment at 10 ppm mass tolerance, missing-value removal), then construct MAML meta-training tasks with support/query splits at 1, 5, and 10 shots per class. Instantiate 1D-CNN backbones (Lecun, LeNet, VGG9) inside the MetaPhenotype MAML framework and meta-train for 50 total epochs with 5 inner gradient-descent cycles per task, initializing the outer-loop optimizer as SGD plus Adam with learning rate 1e-3. Select the backbone by meta-evaluation on held-out IGR39/IGR37 tasks — LeNet was chosen for comparable performance to VGG9 but better 5-shot results and lower computational complexity on 1D spectra. After training, adapt the model to new cell groups (drug-treated cells or a different cell-line pair) with 1- or 5-shot gradient updates, and report AUROC, F1 score, and prediction accuracy after 50 total epochs.

## Related tools

- **MetaPhenotype** (MAML-based meta-learning model (meta.py, base.py) with 1D-CNN LeNet backbone used for phenotype classification) — github.com/songyuan93/MetaPhenotype
- **Single-probe SCMS** (Acquires the single-cell metabolomic profiles of melanoma cell lines that serve as model input)
- **Geena2** (Online tool for peak alignment of SCMS peaks with a 10 ppm mass tolerance during data pretreatment)

## Examples

```
python base.py
```

## Evaluation signals

- Meta-evaluation accuracy exceeds 90% with fewer new training samples than standard transfer-learning models
- AUROC, F1, and accuracy computed after 50 total epochs of training are reproducible across 1-, 5-, and 10-shot settings
- LeNet-based model outperforms VGG9 in the 5-shot setting while remaining computationally cheaper
- All AUROC/F1/accuracy values across backbones and shot settings aggregate into a single meta_training_results.csv table
- Pipeline runs end-to-end with the README entry point (python base.py) reading dataset.csv from the Input data folder

## Limitations

- MAML requires multiple inner-loop gradient steps and outer-loop backpropagation, making it memory- and computationally intensive for larger models, sensitive to hyperparameters, and less effective in nonstationary environments
- Conventional deep learning and this meta-learner alike can overfit when limited SCMS data are available if not trained as few-shot tasks
- Performance depends on the pretreatment protocol (noise/background subtraction, TIC normalization, peak alignment); PCA-like methods cannot separate closely related phenotypes such as IGR39/IGR37, motivating but also constraining this skill to SCMS feature data

## Evidence

- [readme] Repository usage: [evidence span withheld — anchor status: external_doc]
