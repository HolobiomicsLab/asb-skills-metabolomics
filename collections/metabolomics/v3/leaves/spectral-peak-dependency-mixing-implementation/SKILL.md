---
name: spectral-peak-dependency-mixing-implementation
description: Use when extending MLP or GNN spectral predictors so outputs capture
  dependencies among co-occurring spectral peaks via low-rank label mixing and LDA-topic
  multi-task heads.
when_to_use_negative:
- Input is already an ensemble ranking task handled by ens_train_canopus.py — this
  skill only modifies the base MLP/GNN peak-dependency modules, not the ESP weighting
  layer.
- Data are EI/GC-MS spectra from the original NEIMS setting — the paper's implementation
  targets generalized ESI/LC-MS data, not EI/GC-MS.
- The task only requires running pretrained models on a test candidate set without
  modifying architecture — use the plain train.py evaluation path instead.
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_0091
tools:
- name: PyTorch
  role: Framework for implementing the learnable low-rank mixing matrix Ql = Dl Dl^T,
    the residual mixing weight, and the two-layer LDA-topic multi-task head inside
    the MLP/GNN predictors
  repo: https://github.com/HassounLab/ESP
- name: dgl
  role: Graph library used in the GNN baseline whose spectra peak predictions the
    mixing and multi-task modules enhance
  repo: https://github.com/HassounLab/ESP
- name: PyTorch Geometric
  role: Handles the torchgeometric test-candidate graph data that the enhanced GNN
    model consumes
  repo: https://github.com/HassounLab/ESP
- name: scikit-learn
  role: Used for LDA (Latent Dirichlet Allocation) spectral-topic label generation
    that the multi-task head learns to predict
  repo: https://github.com/HassounLab/ESP
- name: numpy
  role: Loads and manipulates the spectra and topic label tensors used to train the
    mixing and multi-task modules
  repo: https://github.com/HassounLab/ESP
- name: scipy
  role: Reads the preprocessed spectral/topic data files used during training of the
    enhanced predictors
  repo: https://github.com/HassounLab/ESP
- name: rdkit
  role: Provides mol objects via mol_dict.pkl for the training molecules whose spectra
    are used with the mixing modules
  repo: https://github.com/HassounLab/ESP
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/bioinformatics/btae490
    title: An Ensemble Spectral Prediction (ESP) model for metabolite annotation
  source_content_hash: sha256:1faf62ccd5a83d2464565559ec7d191bdce691f0813dea609852abcebe097fdb
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/HassounLab/ESP
  tools:
  - PyTorch
  - PyTorch Geometric
  - dgl
  - numpy
  - rdkit
  - scikit-learn
  - scipy
  edam_operation: http://edamontology.org/operation_3927
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 14
---

# spectral-peak-dependency-mixing-implementation

## Summary

Implementation of ESP's two mechanisms for capturing dependencies among co-occurring spectral peaks: a learnable low-rank correlation mixing matrix Ql = Dl Dl^T applied to the 1000-bin predicted spectra, and a multi-task auxiliary head predicting LDA spectral-topic distributions.

## When to use

Use when extending MLP or GNN spectral predictors so outputs capture dependencies among co-occurring spectral peaks via low-rank label mixing and LDA-topic multi-task heads.

## When NOT to use

- Input is already an ensemble ranking task handled by ens_train_canopus.py — this skill only modifies the base MLP/GNN peak-dependency modules, not the ESP weighting layer.
- Data are EI/GC-MS spectra from the original NEIMS setting — the paper's implementation targets generalized ESI/LC-MS data, not EI/GC-MS.
- The task only requires running pretrained models on a test candidate set without modifying architecture — use the plain train.py evaluation path instead.

## Inputs

- Preprocessed spectral training data (torch_trvate_1000bin.pkl from data_trvate.py)
- 1000-bin spectra label tensors and LDA spectral-topic label tensors (numpy/scipy)
- InChIKey-to-rdkit-mol dictionary (mol_dict.pkl from Zenodo 10929354)
- NPLIB1/NIST-20 training CSV (pos_train.csv from Zenodo 10929401)

## Outputs

- Mixed 1000-bin predicted spectrum tensor incorporating low-rank correlation mixing
- T-topic LDA spectral-topic probability distribution from the multi-task head
- Trained model checkpoint (best_model_' + model_file_suffix + '.pt) with mixing-matrix and topic-head parameters
- Checkpoint module outputs saved to spectra_topic_check.npz for validation

## How to apply

Load the preprocessed spectral training data and spectra/topic label tensors from the NIST-20/NPLIB1 subset files (pos_train.csv, mol_dict.pkl, torch_trvate_1000bin.pkl) using numpy/scipy. In PyTorch, add a learnable low-rank mixing matrix Ql = Dl Dl^T (rank set via --correlation_mat_rank, e.g. 100) applied to the 1000-bin predicted spectra with a residual mixing weight (--correlation_mix_residual_weight 0.7 for GNN, 0.8 for MLP), plus an auxiliary two-layer multi-task head predicting LDA spectral-topic distributions weighted by --mt_lda_weight 0.01. Enable the modules by passing --disable_two_step_pred, --disable_fingerprint, --disable_mt_fingerprint, and --disable_mt_ontology while leaving --disable_mt_lda off, then run training with batch_size=128, lr=0.0005, hidden_dims=1024, num_hidden_layers=3. Confirm the forward/backward pass completes and that the mixed spectrum output is 1000-bin while the topic head emits T-topic probability distributions; save checkpoint outputs (e.g. spectra_topic_check.npz) for validation.

## Related tools

- **PyTorch** (Framework for implementing the learnable low-rank mixing matrix Ql = Dl Dl^T, the residual mixing weight, and the two-layer LDA-topic multi-task head inside the MLP/GNN predictors) — https://github.com/HassounLab/ESP
- **dgl** (Graph library used in the GNN baseline whose spectra peak predictions the mixing and multi-task modules enhance) — https://github.com/HassounLab/ESP
- **PyTorch Geometric** (Handles the torchgeometric test-candidate graph data that the enhanced GNN model consumes) — https://github.com/HassounLab/ESP
- **scikit-learn** (Used for LDA (Latent Dirichlet Allocation) spectral-topic label generation that the multi-task head learns to predict) — https://github.com/HassounLab/ESP
- **numpy** (Loads and manipulates the spectra and topic label tensors used to train the mixing and multi-task modules) — https://github.com/HassounLab/ESP
- **scipy** (Reads the preprocessed spectral/topic data files used during training of the enhanced predictors) — https://github.com/HassounLab/ESP
- **rdkit** (Provides mol objects via mol_dict.pkl for the training molecules whose spectra are used with the mixing modules) — https://github.com/HassounLab/ESP

## Examples

```
python train.py --cuda 1 --model gnn --model_file_suffix gnn_pd --disable_two_step_pred --disable_fingerprint --disable_mt_fingerprint --disable_mt_ontology --correlation_mat_rank 100 --te_cand_dataset_suffix torch_tecand_1000bin_te_cand100
```

## Evaluation signals

- Training forward/backward passes complete without error on the NPLIB1/NIST-20 subset.
- Mixed spectrum output has exactly 1000 bins (1000/hp['resolution'] binning convention).
- Multi-task head emits a probability distribution over T LDA spectral topics for each molecule.
- correlation_mat_rank set to a valid low-rank value (e.g. 100) and mixing weights (0.7 GNN / 0.8 MLP) applied without NaNs.
- Saved checkpoint module outputs (spectra_topic_check.npz) reproduce the expected spectra/topic shapes on reload.

## Limitations

- NIST-20 training data cannot be redistributed under NIST license regulations, so the mixing modules can only be validated on the NPLIB1 subset or user-acquired NIST-20 data.
- In the README demo, the ESP ensemble run used batch_size=128, lr=0.001, epochs=100 while base models used lr=0.0005, epochs=50 — the mixing-module training regime differs from ensemble training.
- The 23.7%/37.2% rank gains are measured on the full NIST candidate set with Rank@1 of 0.230 (MLP) and 0.115 (GNN); the topic-mixing modules are one contributor and not the sole source of improvement.

## Evidence

- [methods] label mixing and LDA topic multi-task modules: [evidence span withheld — anchor status: not_found]
- [readme] correlation_mat_rank and mixing weights: "--correlation_mat_rank 100 --mt_lda_weight=0.01, correlation_mix_residual_weight=0.7"
- [readme] 1000-bin output shape check: "torch_tecand_1000bin_te_cand100"
- [intro] low-rank mixing definition: "We propose a novel ensemble model to take advantage of both MLP and GNN models."
