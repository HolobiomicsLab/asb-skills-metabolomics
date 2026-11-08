---
name: 1d-cnn-training-on-metabolomic-signals
description: Use when you have preprocessed SCMS metabolomics data (peak tables like
  dataset.csv with TIC-normalized, background-subtracted single-cell profiles) and
  need to classify cell phenotypes from limited samples using a 1D-CNN, optionally
  as a MAML backbone.
when_to_use_negative:
- When the goal is exploratory discrimination of cell groups whose metabolomic profiles
  are highly similar and a linear method would suffice to check separability first
  — note the article reports PCA cannot effectively discriminate similar groups like
  IGR39/IGR37, so confirm PCA fails before committing to CNN training
- When abundant labeled training samples are available and standard transfer learning
  or conventional deep learning would not overfit
- When input data are raw mass spectra that have not undergone the SCMS pretreatment
  protocol (noise/background subtraction, TIC normalization, 10 ppm peak alignment)
edam_operation: http://edamontology.org/operation_2995
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3474
tools:
- name: MetaPhenotype
  role: MAML-based meta-learning framework wrapping the 1D-CNN (LeNet) backbone; code
    lives in meta.py with base.py as the entry point
  repo: https://github.com/songyuan93/MetaPhenotype
- name: Single-probe SCMS
  role: Acquisition technique producing the single-cell metabolomic profiles used
    as training/evaluation input
- name: Geena2
  role: Online tool for peak alignment of SCMS peaks with a 10 ppm mass tolerance
    prior to model training
- name: MetaboAnalyst 6.0
  role: Used for PCA baseline check of whether metabolomic profiles are discriminable
    before CNN training
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1021/acs.analchem.4c02038
    title: 'MetaPhenotype: A Transferable Meta-Learning Model for Single-Cell Mass
      Spectrometry-Based Cell Phenotype Prediction Using Limited Number of Cells'
  source_content_hash: sha256:d1d94b60b3a91b9aabe6a94da489a1f74cc0a2e10e860e2ba148600195b38eb0
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/songyuan93/MetaPhenotype
  tools:
  - Geena2
  - MetaPhenotype
  - MetaboAnalyst 6.0
  - Single-probe SCMS
  edam_operation: http://edamontology.org/operation_2995
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# 1D-CNN training on metabolomic signals

## Summary

Trains a 1D-CNN (LeNet) backbone on preprocessed single-cell mass spectrometry (SCMS) metabolomics profiles to classify cells into primary or metastatic phenotypes, either standalone or as the feature extractor inside the MAML-based MetaPhenotype meta-learning model. Chosen because 1D-CNNs have low computational complexity and are easier to train when SCMS training data are limited.

## When to use

Use when you have preprocessed SCMS metabolomics data (peak tables like dataset.csv with TIC-normalized, background-subtracted single-cell profiles) and need to classify cell phenotypes from limited samples using a 1D-CNN, optionally as a MAML backbone.

## When NOT to use

- When the goal is exploratory discrimination of cell groups whose metabolomic profiles are highly similar and a linear method would suffice to check separability first — note the article reports PCA cannot effectively discriminate similar groups like IGR39/IGR37, so confirm PCA fails before committing to CNN training
- When abundant labeled training samples are available and standard transfer learning or conventional deep learning would not overfit
- When input data are raw mass spectra that have not undergone the SCMS pretreatment protocol (noise/background subtraction, TIC normalization, 10 ppm peak alignment)

## Inputs

- Preprocessed SCMS metabolomics peak table (dataset.csv) from Single-probe SCMS acquisitions on a Thermo LTQ Orbitrap XL
- Cell phenotype labels (primary vs. metastatic) for melanoma cell lines (e.g., IGR39/IGR37, WM115/WM266-4)
- Shot-count configuration (1-, 5-, or 10-shot) and task/adaptation sample definitions

## Outputs

- Trained 1D-CNN (LeNet) classifier or meta-trained MetaPhenotype checkpoint
- Per-cell phenotype predictions with AUROC, F-1 score, and accuracy after 50 epochs
- Selected small set of metabolomic species critical for phenotype classification

## How to apply

Start from SCMS data pretreated per the established protocol: peak export, noise subtraction, background subtraction, TIC normalization, peak alignment (Geena2, 10 ppm mass tolerance), and removal of missing values; the README provides the resulting dataset.csv in the Input data folder, read by dataloade.py. Build the 1D-CNN with a LeNet backbone, since a 1D-CNN is preferable for analyzing 1D signals when limited training data are available due to lower computational complexity. If data are scarce, wrap the backbone in the MAML meta-learning scheme (meta.py): train over tasks using 1/5/10-shot episodes, then adapt via inner-loop gradient descent to new tasks (e.g., drug-treated IGR39/IGR37 or WM115/WM266-4 cells) within a small number of optimization steps. Train for 50 total epochs and evaluate with AUROC, F-1 score, and prediction accuracy on held-out cells. Decision point: expect >90% accuracy with fewer adaptation samples than standard transfer learning; if the model overfits, this signals conventional (non-meta) training is unsuitable for the sample size.

## Related tools

- **MetaPhenotype** (MAML-based meta-learning framework wrapping the 1D-CNN (LeNet) backbone; code lives in meta.py with base.py as the entry point) — https://github.com/songyuan93/MetaPhenotype
- **Single-probe SCMS** (Acquisition technique producing the single-cell metabolomic profiles used as training/evaluation input)
- **Geena2** (Online tool for peak alignment of SCMS peaks with a 10 ppm mass tolerance prior to model training)
- **MetaboAnalyst 6.0** (Used for PCA baseline check of whether metabolomic profiles are discriminable before CNN training)

## Examples

```
python base.py
```

## Evaluation signals

- Adapted MetaPhenotype reaches >90% prediction accuracy on unseen cell groups (treated IGR39/IGR37, untreated/treated WM115/WM266-4) after only five adaptation steps
- AUROC, F-1 score, and accuracy computed after 50 total epochs of training reproduce the comparison against standard transfer-learning baselines (Figure 3, Table S6)
- MetaPhenotype outperforms transfer learning in data efficiency, i.e., achieves comparable accuracy with fewer new training samples
- Model converges without overfitting despite limited SCMS cell counts — overfitting indicates the meta-learning setup was not applied correctly
- Input table loads via dataloade.py from dataset.csv in the Input data folder as documented in the README

## Limitations

- MAML is memory- and computationally intensive for larger models due to multiple inner-loop gradient steps and backpropagation through updates in the outer loop
- MAML is sensitive to hyperparameters and less effective in nonstationary environments
- Conventional deep learning on this data overfits when limited SCMS samples are available; ambient SCMS methods yield relatively small cell numbers, restricting data size
- The model was demonstrated on melanoma SCMS pairs (IGR39/IGR37, WM115/WM266-4, with/without vemurafenib); transfer beyond such binary primary/metastatic tasks is not established here

## Evidence

- [intro] 1D-CNN, which is easier to train due to its less computational complexity, would be clearly preferable for analyzing 1D signals, particularly when limited training data are available: [evidence span withheld — anchor status: not_found]
- [intro] we computed the area under the receiver operating characteristic curve (AUROC), F-1 score, and prediction accuracy after 50 total epochs of training: "we computed the area under the receiver operating characteristic curve (AUROC), F-1 score, and prediction accuracy after 50 total epochs of training"
- [intro] a MAML model is trained for some tasks using a small number of samples, and the trained model is then be adapted to a new task within a small number of optimization (such as gradient descent) steps: "a MAML model is trained for some tasks using a small number of samples, and the trained model is then be adapted to a new task within a small number of optimization (such as gradient descent) steps"
- [readme] The input data were included in the Input data folder which named dataset.csv: [evidence span withheld — anchor status: external_doc]
- [readme] The code for MetaPhenotype model were included in meta.py file.: [evidence span withheld — anchor status: external_doc]
- [intro] The SCMS data pretreatment was performed using our previously established protocol: "The SCMS data pretreatment was performed using our previously established protocol"
