---
name: relative-error-accuracy-comparison
description: Decision support for choosing between retention-time prediction models
  on an LC-MS small-molecule dataset when the metrics of interest are mean and median
  relative error and a head-to-head accuracy ranking is needed.
when_to_use_negative:
- Input is a spectral file collection needing preprocessing rather than already-trained
  model predictions to compare
- Goal is transfer learning to a new chromatographic system rather than an accuracy
  ranking on the original SMRT test split
- Retention-time labels or a held-out test split are missing, so no relative error
  can be computed
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0154
tools:
- name: pytorch
  role: Training framework for the GNN-RT model whose accuracy is compared against
    the baselines
  repo: https://github.com/Qiong-Yang/GNN-RT
- name: rdkit
  role: Converts molecular structures into graph inputs for GNN-RT before training
    and prediction
  repo: https://github.com/Qiong-Yang/GNN-RT
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/acs.analchem.0c04071
    title: Prediction of Liquid Chromatographic Retention Time with Graph Neural Networks
      to Assist in Small Molecule Identification
  source_content_hash: sha256:d42ecd7abcb0dfa5e691c7e2800cb4f2e67084c23df77b0802237314b5d13290
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/Qiong-Yang/GNN-RT
  tools:
  - pytorch
  - rdkit
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# relative-error-accuracy-comparison

## Summary

Compares retention-time prediction accuracy across GNN-RT and baseline regressors (random forest, Bayesian ridge regression, CNN, DLM) on the METLIN SMRT dataset using mean relative error and median relative error as the evaluation metrics.

## When to use

Decision support for choosing between retention-time prediction models on an LC-MS small-molecule dataset when the metrics of interest are mean and median relative error and a head-to-head accuracy ranking is needed.

## When NOT to use

- Input is a spectral file collection needing preprocessing rather than already-trained model predictions to compare
- Goal is transfer learning to a new chromatographic system rather than an accuracy ranking on the original SMRT test split
- Retention-time labels or a held-out test split are missing, so no relative error can be computed

## Inputs

- METLIN SMRT dataset (small molecule retention time) with molecular structures
- Trained GNN-RT model checkpoint (PyTorch)
- Baseline model predictions (random forest, Bayesian ridge regression, CNN, DLM) on the same test split

## Outputs

- Per-model mean relative error and median relative error
- Comparison table ranking GNN-RT vs. the four baselines
- Saved GNN-RT checkpoint and accuracy table

## How to apply

First train the GNN-RT model on the METLIN SMRT dataset using the repository's Preprocess.py and Train.py pipeline, converting molecular structures into graph representations with RDKit and learning retention time end-to-end with PyTorch without molecular descriptors. Train the four baselines — random forest, Bayesian ridge regression, CNN, and DLM — on the same SMRT train/test splits so that all five models are evaluated on identical held-out data. On the test set compute mean relative error and median relative error for each model; the paper reports GNN-RT achieving mean relative error 4.9% and median relative error 3.2%. Tabulate the per-model metrics side by side and confirm the GNN-RT model achieves the lowest errors before saving the trained checkpoint and comparison table as deliverables.

## Related tools

- **pytorch** (Training framework for the GNN-RT model whose accuracy is compared against the baselines) — https://github.com/Qiong-Yang/GNN-RT
- **rdkit** (Converts molecular structures into graph inputs for GNN-RT before training and prediction) — https://github.com/Qiong-Yang/GNN-RT

## Examples

```
python Preprocess.py && python Train.py
```

## Evaluation signals

- GNN-RT mean relative error on the SMRT test set falls near the reported 4.9%
- GNN-RT median relative error on the SMRT test set falls near the reported 3.2%
- All five models are evaluated on the same held-out SMRT test split
- Comparison table includes random forest, Bayesian ridge regression, CNN, and DLM alongside GNN-RT
- Trained checkpoint and comparison table are saved as final outputs

## Limitations

- Accuracy figures are specific to the SMRT dataset and its chromatographic system; transferability to other systems is a separate property
- No changelog is provided for the repository, so pipeline behavior may vary across versions
- Baseline models must be retrained on the same splits for a fair comparison; off-the-shelf predictions are not sufficient

## Evidence

- [abstract] GNN-RT achieved the highest predicting accuracy with a mean relative error of 4.9% and a median relative error of 3.2%.: "GNN-RT achieved the highest predicting accuracy with a mean relative error of 4.9% and a median relative error of 3.2%."
- [intro] We present the GNN-RT method, which is proved to be an effective way to predict small molecule LC retention time and improve the accuracy of structural identification of small molecules.: "Hence, we present the GNN-RT method, which is proved to be an effective way to predict small molecule LC  retention time  and improve the accuracy of structural identification of small molecules."
- [readme] Usage instructions for training a model from an in-house database with the provided pipeline scripts.: "please put your spectra files in to **data** directory and run   [Preprocess.py](Preprocess.py) ,  [Train.py](Train.py) and  [Transferlearning.py](Transferlearning.py)"
- [abstract] GNN-RT predicts retention time directly from molecular structures without molecular descriptors.: "to predict the retention time (GNN-RT) from structures of small molecules directly without the requirement of molecular descriptors"
