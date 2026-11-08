---
name: retention-time-regression-model-training
description: Use when you have small-molecule structures with measured LC retention
  times (e.g. a METLIN SMRT-style dataset) and need an accurate data-driven retention
  time regression model for structural identification support.
when_to_use_negative:
- When you only have MS/MS spectra or mass spectral features and no molecular structures
  with retention time labels — the GNN-RT model requires molecular graph inputs from
  structures, not spectral features.
- When you need retention time prediction for a chromatographic system fundamentally
  different from the SMRT training system without doing transfer learning — the article
  notes the SMRT-trained model transfers easily only to 'the same type of chromatographic
  systems'.
- When your molecules are outside the small-molecule scope (e.g. large biomolecules),
  since the method is designed for small molecule LC retention time.
edam_topics:
- http://edamontology.org/topic_3336
tools:
- name: PyTorch
  role: Deep learning framework used to train the GNN-RT model end-to-end on molecular
    graph inputs
  repo: https://github.com/Qiong-Yang/GNN-RT
- name: RDKit
  role: Used to preprocess molecular structures into the graph representations consumed
    by GNN-RT
  repo: https://github.com/Qiong-Yang/GNN-RT
- name: GNN-RT repository (Preprocess.py, Train.py, Transferlearning.py)
  role: 'Reference implementation: run Preprocess.py, Train.py, and Transferlearning.py
    from the Qiong-Yang/GNN-RT repo to preprocess data, train the model, and transfer
    it to a new chromatographic system'
  repo: https://github.com/Qiong-Yang/GNN-RT
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/acs.analchem.0c04071
    title: Prediction of Liquid Chromatographic Retention Time with Graph Neural Networks
      to Assist in Small Molecule Identification
  source_content_hash: sha256:36ad69293dac3bfbcdd5ae1c31ec07b5192ce10498cd9546c51fd7177b013b37
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/Qiong-Yang/GNN-RT
  tools:
  - GNN-RT repository (Preprocess.py, Train.py, Transferlearning.py)
  - PyTorch
  - RDKit
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3336
  provenance_tier: literature
license: CC-BY-4.0
merged_aliases:
- retention-time-prediction-model-training
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# retention-time-regression-model-training

## Summary

Train a graph neural network (GNN-RT) that predicts liquid chromatographic retention time directly from molecular graph representations of small molecules, avoiding handcrafted molecular descriptors, and benchmark it against regression baselines (random forest, Bayesian ridge regression, CNN, DLM) on the METLIN SMRT dataset.

## When to use

Use when you have small-molecule structures with measured LC retention times (e.g. a METLIN SMRT-style dataset) and need an accurate data-driven retention time regression model for structural identification support.

## When NOT to use

- When you only have MS/MS spectra or mass spectral features and no molecular structures with retention time labels — the GNN-RT model requires molecular graph inputs from structures, not spectral features.
- When you need retention time prediction for a chromatographic system fundamentally different from the SMRT training system without doing transfer learning — the article notes the SMRT-trained model transfers easily only to 'the same type of chromatographic systems'.
- When your molecules are outside the small-molecule scope (e.g. large biomolecules), since the method is designed for small molecule LC retention time.

## Inputs

- Small-molecule structures paired with measured LC retention times (e.g. the METLIN SMRT dataset)
- Spectra/structure files placed in the repository's 'data' directory for in-house training sets

## Outputs

- Trained GNN-RT retention time prediction model (model checkpoint)
- Predicted retention times for held-out molecules
- Accuracy comparison (mean and median relative error) of GNN-RT vs random forest, Bayesian ridge, CNN, and DLM baselines

## How to apply

Load the METLIN small molecule retention time (SMRT) dataset, preprocess each molecule's structure into a molecular graph with RDKit, and train the GNN-RT model end-to-end in PyTorch so it takes the molecular graph as input and predicts retention time as output, without computing molecular descriptors. Train the baseline regressors — random forest, Bayesian ridge regression, convolutional neural network (CNN), and deep learning model (DLM) — on the same dataset splits. Evaluate all models on the held-out test set using mean relative error and median relative error as the accuracy metrics. Confirm the GNN-RT model reaches a mean relative error of about 4.9% and a median relative error of about 3.2% on SMRT, which is the published reference accuracy for this skill. The resulting SMRT-trained GNN-RT model can be transferred to chromatographic systems of the same type via transfer learning using the repository's Transferlearning.py script.

## Related tools

- **PyTorch** (Deep learning framework used to train the GNN-RT model end-to-end on molecular graph inputs) — https://github.com/Qiong-Yang/GNN-RT
- **RDKit** (Used to preprocess molecular structures into the graph representations consumed by GNN-RT) — https://github.com/Qiong-Yang/GNN-RT
- **GNN-RT repository (Preprocess.py, Train.py, Transferlearning.py)** (Reference implementation: run Preprocess.py, Train.py, and Transferlearning.py from the Qiong-Yang/GNN-RT repo to preprocess data, train the model, and transfer it to a new chromatographic system) — https://github.com/Qiong-Yang/GNN-RT

## Examples

```
conda install -c rdkit rdkit; conda install pytorch; python Preprocess.py; python Train.py; python Transferlearning.py
```

## Evaluation signals

- GNN-RT achieves mean relative error ≈ 4.9% and median relative error ≈ 3.2% on the SMRT test set; larger errors on equivalent splits indicate the model or preprocessing is wrong.
- GNN-RT's mean and median relative error should be lower than those of random forest, Bayesian ridge regression, CNN, and DLM trained on the same splits.
- The model consumes molecular graph inputs derived from structures (via RDKit) and outputs a single retention time value per molecule — check the output shape/dtype.
- Training and evaluation must use consistent dataset splits across GNN-RT and all baselines so relative error comparisons are meaningful.
- For transfer to a new chromatographic system of the same type, the transfer learning step should improve accuracy relative to zero-shot use of the SMRT-trained model.

## Limitations

- The article does not report a changelog or version history for the GNN-RT implementation, so reproducibility depends on pinning the repository commit.
- Transfer of the SMRT-trained model is validated for 'the same type of chromatographic systems'; different stationary phases or gradient programs may require retraining or transfer learning.
- The reported metrics (mean 4.9%, median 3.2% relative error) are on the SMRT dataset; they may not generalize to other datasets without re-verification.
- Retention time prediction accuracy must be high enough to meaningfully assist structural identification; the method's value depends on the identified benchmark accuracy being reproduced.

## Evidence
