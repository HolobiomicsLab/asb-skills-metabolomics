---
name: maml-1d-cnn-backbone-selection
description: Use when building a MAML meta-learning classifier for single-cell mass
  spectrometry (SCMS) metabolomics data and you must decide which 1D-CNN backbone
  to adopt before full-scale meta-training with limited cells.
when_to_use_negative:
- Input data are bulk omics profiles or already-labeled large feature matrices — the
  comparison targets limited-sample SCMS single-cell metabolomics where ambient methods
  yield relatively small cell numbers
- The classification goal involves cell pairs with high metabolomic similarity that
  PCA can discriminate easily — a simpler model comparison may suffice
- You need image (2D) or sequence inputs — these backbones are 1D-CNNs designed for
  1D SCMS spectral signals
edam_operation: http://edamontology.org/operation_3936
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3474
tools:
- name: MetaPhenotype
  role: MAML meta-learning framework in which the Lecun/LeNet/VGG9 1D-CNN backbones
    are instantiated, meta-trained, and meta-evaluated
  repo: github.com/songyuan93/MetaPhenotype
- name: Geena2
  role: Online bioinformatics tool used for peak alignment (10 ppm mass tolerance)
    during upstream SCMS data pretreatment feeding the backbone comparison
- name: SCMS metabolomics
  role: Single-cell mass spectrometry metabolomics data (untreated IGR39/IGR37 pair)
    used as training and evaluation data for backbone selection
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/acs.analchem.4c02038
    title: 'MetaPhenotype: A Transferable Meta-Learning Model for Single-Cell Mass
      Spectrometry-Based Cell Phenotype Prediction Using Limited Number of Cells'
  source_content_hash: sha256:7fcd65753d761fc76a30aea6f00da581d20dc0c4ed03b67f0452531f4a3165d7
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Geena2
  - MetaPhenotype
  - SCMS metabolomics
  edam_operation: http://edamontology.org/operation_3936
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# maml-1d-cnn-backbone-selection

## Summary

Benchmark 1D-CNN backbones (Lecun, LeNet, VGG9) inside the MAML-based MetaPhenotype framework on untreated IGR39/IGR37 SCMS metabolomics data across 1-, 5-, and 10-shot meta-training settings, and select the backbone (LeNet) that best balances meta-evaluation accuracy and computational efficiency.

## When to use

Use when building a MAML meta-learning classifier for single-cell mass spectrometry (SCMS) metabolomics data and you must decide which 1D-CNN backbone to adopt before full-scale meta-training with limited cells.

## When NOT to use

- Input data are bulk omics profiles or already-labeled large feature matrices — the comparison targets limited-sample SCMS single-cell metabolomics where ambient methods yield relatively small cell numbers
- The classification goal involves cell pairs with high metabolomic similarity that PCA can discriminate easily — a simpler model comparison may suffice
- You need image (2D) or sequence inputs — these backbones are 1D-CNNs designed for 1D SCMS spectral signals

## Inputs

- Pretreated SCMS metabolomics feature table of untreated IGR39/IGR37 melanoma cells (dataset.csv from the MetaPhenotype repo)
- MAML meta-training task definitions (support/query splits) at 1, 5, and 10 shots per class
- 1D-CNN backbone definitions: Lecun, LeNet, VGG9 (from meta.py in the MetaPhenotype repository)

## Outputs

- Trained MAML initializations for each backbone (Lecun, LeNet, VGG9) and shot setting
- Meta-evaluation metrics (AUROC, F1 score, prediction accuracy) per backbone per shot setting
- Aggregated comparison table meta_training_results.csv
- Selected backbone (LeNet) for the final MetaPhenotype model

## How to apply

Load pretreated SCMS metabolomics data (peak export, noise/background subtraction, TIC normalization, Geena2 alignment at 10 ppm mass tolerance, missing-value removal) for the untreated IGR39/IGR37 melanoma cell pair and construct MAML meta-training tasks with support/query splits at 1, 5, and 10 shots per class. Instantiate the Lecun, LeNet, and VGG9 1D-CNN backbones from github.com/songyuan93/MetaPhenotype (meta.py) with the outer-loop optimizer initialized as SGD plus Adam at learning rate 1e-3, and meta-train for 50 total epochs with 5 inner gradient-descent cycles per task for each backbone/shot combination. Meta-evaluate each trained initialization on held-out IGR39/IGR37 tasks, computing AUROC, F1 score, and prediction accuracy after 50 total epochs of training. Aggregate all metrics into a single results table (meta_training_results.csv); select LeNet because it matched or exceeded VGG9 (notably in the 5-shot setting) at lower computational complexity. Rationale: 1D-CNNs are easier to train with limited SCMS data, and LeNet's simplicity yields >90% classification accuracy with fewer new training samples.

## Related tools

- **MetaPhenotype** (MAML meta-learning framework in which the Lecun/LeNet/VGG9 1D-CNN backbones are instantiated, meta-trained, and meta-evaluated) — github.com/songyuan93/MetaPhenotype
- **Geena2** (Online bioinformatics tool used for peak alignment (10 ppm mass tolerance) during upstream SCMS data pretreatment feeding the backbone comparison)
- **SCMS metabolomics** (Single-cell mass spectrometry metabolomics data (untreated IGR39/IGR37 pair) used as training and evaluation data for backbone selection)

## Examples

```
python base.py  # meta.py defines the MetaPhenotype MAML model; dataloade.py reads dataset.csv
```

## Evaluation signals

- LeNet-based MetaPhenotype reaches >90% classification accuracy on held-out IGR39/IGR37 tasks with limited new training samples
- LeNet outperforms VGG9 specifically in the 5-shot meta-evaluation setting, with comparable performance in 1- and 10-shot settings
- AUROC, F1 score, and prediction accuracy are all computed after 50 total epochs of training for every backbone/shot combination
- All metric values across backbones and shot settings are aggregated into a single meta_training_results.csv without missing cells
- Outer-loop optimizer settings (SGD plus Adam, learning rate 1e-3) and 5 inner gradient-descent cycles per task are identical across all compared backbones

## Limitations

- MAML is memory- and computationally intensive due to multiple inner-loop gradient steps and outer-loop backpropagation through updates, constraining backbone size
- MAML performance is sensitive to hyperparameters (e.g., learning rate, inner-loop step count) and less effective in nonstationary environments
- The comparison is grounded on one untreated primary/metastatic melanoma cell pair (IGR39/IGR37) with limited cells, as ambient SCMS yields relatively small cell numbers
- PCA cannot effectively discriminate IGR39/IGR37 cells with highly similar metabolomic profiles, so backbone selection cannot be shortcut by simple statistical screening

## Evidence

- [intro] 1D-CNN, which is easier to train due to its less computational complexity, would be clearly preferable for analyzing 1D signals, particularly when limited training data are available: [evidence span withheld — anchor status: not_found]
- [readme] README run instruction: [evidence span withheld — anchor status: not_found]
