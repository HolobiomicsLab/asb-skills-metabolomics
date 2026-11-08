---
name: melanoma-phenotype-classification
description: Use when you have few-shot labeled single-cell mass spectrometry (SCMS)
  metabolomic feature vectors from melanoma cell pairs (untreated or drug-treated)
  and need rapid phenotype classification plus selection of essential m/z species.
when_to_use_negative:
- Input is bulk metabolomics or flow-cytometry data rather than single-cell SCMS feature
  vectors — the model is trained and evaluated on Single-probe SCMS cell-level metabolomic
  profiles.
- A large labeled SCMS training set is already available and PCA or conventional deep
  learning suffices — the article motivates MetaPhenotype only when limited data and
  rapid transfer to new cell groups are required.
- The cell pair to classify has very similar metabolomic profiles such that no meaningful
  classification is expected (e.g. PCA cannot discriminate IGR39/IGR37 without the
  meta-learning adaptation).
edam_operation: http://edamontology.org/operation_3936
edam_topics:
- http://edamontology.org/topic_3382
- http://edamontology.org/topic_0154
tools:
- name: MetaPhenotype
  role: MAML-based meta-learning model with 1D-CNN (LeNet) backbone used to train
    on few-shot SCMS data, adapt to new cell pairs, and select essential m/z species
  repo: github.com/songyuan93/MetaPhenotype
- name: Single-probe SCMS
  role: Instrumental method that acquires the single-cell metabolomic profiles fed
    to MetaPhenotype for phenotype classification
- name: Geena2
  role: Online bioinformatics tool used for SCMS peak alignment with a 10 ppm mass
    tolerance before model input
- name: MetaboAnalyst 6.0
  role: Used for principal component analysis (PCA) as the conventional baseline that
    fails to discriminate phenotypically similar cells
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1021/acs.analchem.4c02038
    title: 'MetaPhenotype: A Transferable Meta-Learning Model for Single-Cell Mass
      Spectrometry-Based Cell Phenotype Prediction Using Limited Number of Cells'
  source_content_hash: sha256:8d75dedcffcf5cfd83ab7728d26b7bb07838a232011a4c8cef291e23eca2fe96
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Geena2
  - GitHub (MetaPhenotype repository)
  - MetaPhenotype
  - MetaboAnalyst 6.0
  - Single-probe SCMS
  edam_operation: http://edamontology.org/operation_3936
  edam_topics:
  - http://edamontology.org/topic_3382
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9473684210526315
n_claims: 19
---

# melanoma-phenotype-classification

## Summary

Classifies single-cell SCMS metabolomics data of melanoma cells into primary (IGR39) vs metastatic (IGR37) phenotypes using the MAML-based 1D-CNN MetaPhenotype model, which trains and adapts with as few as 1–5 labeled cells per class. It exists because standard transfer learning and PCA fail when SCMS training data are limited.

## When to use

Use when you have few-shot labeled single-cell mass spectrometry (SCMS) metabolomic feature vectors from melanoma cell pairs (untreated or drug-treated) and need rapid phenotype classification plus selection of essential m/z species.

## When NOT to use

- Input is bulk metabolomics or flow-cytometry data rather than single-cell SCMS feature vectors — the model is trained and evaluated on Single-probe SCMS cell-level metabolomic profiles.
- A large labeled SCMS training set is already available and PCA or conventional deep learning suffices — the article motivates MetaPhenotype only when limited data and rapid transfer to new cell groups are required.
- The cell pair to classify has very similar metabolomic profiles such that no meaningful classification is expected (e.g. PCA cannot discriminate IGR39/IGR37 without the meta-learning adaptation).

## Inputs

- SCMS metabolomics feature matrix (dataset.csv) from Single-probe single-cell mass spectrometry of melanoma cells
- Small labeled training/evaluation cell subsets (1-, 5-, or 10-shot) for the primary/metastatic phenotype classes
- Pretrained MetaPhenotype model checkpoints from github.com/songyuan93/MetaPhenotype (untreated cell-pair model or 1-/5-shot adapted models)
- Peak feature tables after Geena2 alignment (10 ppm) with TIC-normalized intensities

## Outputs

- Per-cell phenotype labels (primary IGR39 vs metastatic IGR37) with AUROC, F1 score, and prediction accuracy
- Significant-peak table (significant_peaks.csv) listing condition, m/z, and the accuracy drop upon peak removal
- Volcano-plot overlay marking essential m/z species as bold dots
- Classification-accuracy curves comparing MetaPhenotype vs standard transfer learning with respect to number of new training samples

## How to apply

Pretreat the SCMS data with the established protocol — noise subtraction, background subtraction, TIC normalization, Geena2 peak alignment at 10 ppm mass tolerance, and removal of missing values — then export the peak feature matrix as dataset.csv for the MetaPhenotype code. Train the MAML model with a 1D-CNN (LeNet) backbone using 1-, 5-, or 10-shot episodes so the model learns an initialization adaptable to new cell pairs within few gradient-descent steps. Record baseline AUROC, F1 score, and accuracy over 50 total epochs, then adapt the trained model to a new task (e.g. vemurafenib-treated IGR39/IGR37 or a different cell-line pair) with 1- or 5-shot adaptation samples. Optionally run the significance test by removing one m/z peak at a time and flagging a peak as essential when its removal drops classification accuracy by ≥1%, reporting counts (14/30 untreated, 3/53 treated) and marking them as bold dots on volcano plots. Save the resulting significant-peak table (condition, m/z, accuracy drop) and use it for biological interpretation of essential metabolites.

## Related tools

- **MetaPhenotype** (MAML-based meta-learning model with 1D-CNN (LeNet) backbone used to train on few-shot SCMS data, adapt to new cell pairs, and select essential m/z species) — github.com/songyuan93/MetaPhenotype
- **Single-probe SCMS** (Instrumental method that acquires the single-cell metabolomic profiles fed to MetaPhenotype for phenotype classification)
- **Geena2** (Online bioinformatics tool used for SCMS peak alignment with a 10 ppm mass tolerance before model input)
- **MetaboAnalyst 6.0** (Used for principal component analysis (PCA) as the conventional baseline that fails to discriminate phenotypically similar cells)

## Examples

```
python base.py
```

## Evaluation signals

- MetaPhenotype reaches over 90% classification accuracy with fewer new training samples than standard transfer learning models.
- Significant-peak counts per condition match the reported values: 14 and 30 for untreated IGR39/IGR37, and 3 and 53 for treated IGR39/IGR37.
- Every flagged significant ion shows an accuracy decrease of at least 1% upon removal from the SCMS feature matrix.
- Metrics are computed after 50 total epochs of training, reporting AUROC, F1 score, and prediction accuracy.
- The adapted model classifies unseen data (drug-treated cells or a different cell-line pair) using only 1- or 5-shot adaptation samples.

## Limitations

- MAML-based training is memory- and computationally intensive because the inner loop needs multiple gradient descent steps with backpropagation through these updates in the outer loop, and it is sensitive to hyperparameters and less effective in nonstationary environments.
- Conventional machine learning models risk overfitting when limited SCMS data are available for training, so few-shot adaptation is required rather than standard retraining.
- Detected species are only tentatively labeled via online databases (METLIN and HMDB), limiting definitive metabolite identification.

## Evidence

- [intro] MAML-based 1D-CNN design rationale: [evidence span withheld — anchor status: not_found]
- [readme] Repository usage instructions: [evidence span withheld — anchor status: external_doc]
