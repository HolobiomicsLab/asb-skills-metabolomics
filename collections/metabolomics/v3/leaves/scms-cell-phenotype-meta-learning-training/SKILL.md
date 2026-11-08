---
name: scms-cell-phenotype-meta-learning-training
description: Use when SCMS metabolomics data (e.g., Single-probe profiles of melanoma
  cell pairs) are too limited for conventional deep learning and the goal is few-shot
  primary/metastatic cell phenotype classification via meta-learning.
when_to_use_negative:
- The input is already bulk omics data or cell populations rather than single-cell
  SCMS profiles — bulk analysis cannot evaluate cell heterogeneity, which is the motivation
  for this skill.
- A large SCMS training dataset is available and standard supervised deep learning
  or transfer learning would not suffer from overfitting.
- The goal is unsupervised exploratory separation of metabolomic profiles (e.g., PCA
  on MetaboAnalyst 6.0) rather than supervised phenotype classification.
edam_operation: http://edamontology.org/operation_3936
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3474
tools:
- name: MetaPhenotype
  role: MAML-based meta-learning model with 1D-CNN backbone trained on limited SCMS
    data for phenotype classification
  repo: github.com/songyuan93/MetaPhenotype
- name: SCMS metabolomics
  role: Source of single-cell training/evaluation data (Single-probe SCMS on melanoma
    cell lines)
- name: Geena2
  role: Online bioinformatics tool used for peak alignment of SCMS peaks with a mass
    tolerance of 10 ppm
- name: MetaboAnalyst 6.0
  role: Used for PCA during data pretreatment/checking; PCA cannot discriminate highly
    similar IGR39/IGR37 profiles, motivating the model
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/acs.analchem.4c02038
    title: 'MetaPhenotype: A Transferable Meta-Learning Model for Single-Cell Mass
      Spectrometry-Based Cell Phenotype Prediction Using Limited Number of Cells'
  source_content_hash: sha256:66813f9309c81f153f7efa2d7803d241360b6f969145c38a5b714ae9f5269034
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Geena2
  - MetaPhenotype
  - MetaboAnalyst 6.0
  - SCMS metabolomics
  edam_operation: http://edamontology.org/operation_3936
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 15
---

# scms-cell-phenotype-meta-learning-training

## Summary

Trains the MAML-based MetaPhenotype model with a 1D-CNN backbone (LeNet) on limited single-cell mass spectrometry (SCMS) metabolomics data (1/5/10-shot tasks) to classify cell phenotypes such as primary vs metastatic melanoma, achieving >90% accuracy with few new training samples.

## When to use

Use when SCMS metabolomics data (e.g., Single-probe profiles of melanoma cell pairs) are too limited for conventional deep learning and the goal is few-shot primary/metastatic cell phenotype classification via meta-learning.

## When NOT to use

- The input is already bulk omics data or cell populations rather than single-cell SCMS profiles — bulk analysis cannot evaluate cell heterogeneity, which is the motivation for this skill.
- A large SCMS training dataset is available and standard supervised deep learning or transfer learning would not suffer from overfitting.
- The goal is unsupervised exploratory separation of metabolomic profiles (e.g., PCA on MetaboAnalyst 6.0) rather than supervised phenotype classification.

## Inputs

- dataset.csv — pretreated SCMS metabolomics feature table (e.g., untreated IGR39/IGR37 melanoma single-cell profiles)
- MAML task specification: shot settings (1, 5, or 10 shots per class) with support/query splits

## Outputs

- Trained MetaPhenotype MAML initialization (meta.py model weights)
- meta_training_results.csv — table of AUROC, F1 score, and prediction accuracy across backbones (Lecun, LeNet, VGG9) and shot settings
- Cell phenotype predictions (primary vs metastatic) for held-out SCMS tasks

## How to apply

First pretreat SCMS data (peak export, noise/background subtraction, TIC normalization, peak alignment at 10 ppm mass tolerance via Geena2, missing-value removal) and load the resulting dataset.csv via dataloade.py. Construct MAML meta-training tasks with support/query splits at 1, 5, and 10 shots per class, instantiate the LeNet 1D-CNN backbone inside meta.py, and initialize the outer-loop optimizer as SGD plus Adam with learning rate 1e-3. Run meta-training for 50 total epochs with 5 inner gradient-descent cycles per task; 1D-CNN is preferred for 1D spectra because of lower computational complexity when training data are scarce. Meta-evaluate on held-out IGR39/IGR37 tasks after 50 total epochs, computing AUROC, F1 score, and prediction accuracy; LeNet was selected over Lecun and VGG9 for its efficiency and higher 5-shot accuracy. Aggregate all AUROC/F1/accuracy values across backbones and shot settings into a single results table (meta_training_results.csv).

## Related tools

- **MetaPhenotype** (MAML-based meta-learning model with 1D-CNN backbone trained on limited SCMS data for phenotype classification) — github.com/songyuan93/MetaPhenotype
- **SCMS metabolomics** (Source of single-cell training/evaluation data (Single-probe SCMS on melanoma cell lines))
- **Geena2** (Online bioinformatics tool used for peak alignment of SCMS peaks with a mass tolerance of 10 ppm)
- **MetaboAnalyst 6.0** (Used for PCA during data pretreatment/checking; PCA cannot discriminate highly similar IGR39/IGR37 profiles, motivating the model)

## Examples

```
python base.py  # dataloade.py reads Input data/dataset.csv; model code in meta.py
```

## Evaluation signals

- Meta-evaluation on held-out IGR39/IGR37 tasks achieves >90% prediction accuracy, matching the reported MetaPhenotype performance with limited new training samples.
- AUROC, F1 score, and prediction accuracy are all computed after 50 total epochs of training, as specified in the protocol.
- LeNet backbone outperforms or matches VGG9 and Lecun across 1-, 5-, and 10-shot settings, with the reported advantage in the 5-shot setting.
- meta_training_results.csv contains AUROC/F1/accuracy rows for every backbone × shot-setting combination (Lecun/LeNet/VGG9 × 1/5/10-shot).
- Reproduction confirms MetaPhenotype reaches high accuracy with fewer new training samples than standard transfer learning models.

## Limitations

- MAML is memory- and computationally intensive due to multiple inner-loop gradient descent steps and backpropagation through these updates in the outer loop, and it is sensitive to hyperparameters.
- Ambient SCMS methods yield relatively small numbers of cells, limiting training data; PCA cannot effectively discriminate phenotypically similar cells such as IGR39/IGR37, so baseline separability may be absent.
- Conventional deep learning applied to the same limited data can encounter overfitting — do not substitute a standard classifier without the meta-learning task construction.

## Evidence

- [intro] 1D-CNN preferred for 1D signals: [evidence span withheld — anchor status: not_found]
