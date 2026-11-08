---
name: low-rank-matrix-factorization-pytorch
description: Use when enhancing an MLP or GNN spectral predictor so co-occurring spectral-peak
  dependencies are learned, by adding a rank-limited (e.g. correlation_mat_rank=100)
  mixing matrix and an auxiliary LDA topic-distribution head.
when_to_use_negative:
- You only need the baseline MLP or GNN spectrum prediction without peak-dependency
  mixing or LDA topic multi-tasking — set the disable flags and skip this module.
- Your spectra are not discretized to the 1000-bin resolution the ESP pipeline expects
  (e.g. EI/GC-MS NEIMS-style data), since Ql operates on the 1000-bin predicted spectra.
- You cannot train ESP end-to-end because pretrained MLP and GNN checkpoints are missing
  — ESP training requires best_model_mlp_*.pt and best_model_gnn_*.pt first.
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_3482
tools:
- name: PyTorch
  role: Implements the learnable low-rank factors Dl of Ql = Dl Dl^T, the mixing residual
    weight, and the auxiliary two-layer LDA-topic multi-task head (pytorch=1.7.1)
  repo: https://github.com/HassounLab/ESP
- name: numpy
  role: Loads the preprocessed spectral training data and spectra/topic label tensors
    used to train the mixing and topic modules
  repo: https://github.com/HassounLab/ESP
- name: scipy
  role: Reads the preprocessed spectral tensors from the NIST-20 subset files alongside
    numpy
  repo: https://github.com/HassounLab/ESP
- name: dgl
  role: Supports the GNN base model whose 1000-bin predicted spectra are mixed by
    the low-rank matrix (dgl=0.6.1)
  repo: https://github.com/HassounLab/ESP
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/bioinformatics/btae490
    title: An Ensemble Spectral Prediction (ESP) model for metabolite annotation
  source_content_hash: sha256:ce73ce843dc109a1c0afde330c80ef34f1610cddce1687010257b542328f9a67
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_3482
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 16
---

# low-rank-matrix-factorization-pytorch

## Summary

Implementation of ESP's learnable low-rank peak-dependency mixing — matrices Ql = Dl Dl^T applied to 1000-bin predicted spectra via a mixing residual weight — plus the multi-task LDA spectral-topic head, in PyTorch, to capture dependencies among co-occurring spectral peaks.

## When to use

Use when enhancing an MLP or GNN spectral predictor so co-occurring spectral-peak dependencies are learned, by adding a rank-limited (e.g. correlation_mat_rank=100) mixing matrix and an auxiliary LDA topic-distribution head.

## When NOT to use

- You only need the baseline MLP or GNN spectrum prediction without peak-dependency mixing or LDA topic multi-tasking — set the disable flags and skip this module.
- Your spectra are not discretized to the 1000-bin resolution the ESP pipeline expects (e.g. EI/GC-MS NEIMS-style data), since Ql operates on the 1000-bin predicted spectra.
- You cannot train ESP end-to-end because pretrained MLP and GNN checkpoints are missing — ESP training requires best_model_mlp_*.pt and best_model_gnn_*.pt first.

## Inputs

- Preprocessed spectral training data from the NIST-20 subset / NPLIB1 dataset (pos_train.csv, mol_dict.pkl)
- 1000-bin spectra and LDA spectral-topic label tensors (numpy/scipy loaders)
- PyTorch model definition and train.py/ens_train_canopus.py hyperparameter namespace (hidden_dims=1024, num_hidden_layers=3, batch_size=128, lr=0.0005)

## Outputs

- 1000-bin mixed spectrum predictions with low-rank mixing applied (rank-100 Ql = Dl Dl^T)
- T-topic LDA spectral-topic probability distributions from the auxiliary multi-task head
- Trained checkpoint .pt files and module outputs saved to spectra_topic_check.npz

## How to apply

Load the preprocessed spectral training tensors (e.g. from data/final_5 NPLIB1 files such as pos_train.csv and mol_dict.pkl) with numpy/scipy. In PyTorch, add learnable rank-limited factors Dl so that Ql = Dl Dl^T mixes the 1000-bin predicted spectrum, with a residual mixing weight (correlation_mix_residual_weight=0.7 for GNN, 0.8 for MLP in ESP) and correlation_mat_rank=100. Add a two-layer auxiliary multi-task head predicting LDA spectral-topic distributions weighted by mt_lda_weight=0.01. Enable both modules via the flag convention (leave mt_lda enabled; set --disable_two_step_pred, --disable_fingerprint, --disable_mt_fingerprint, --disable_mt_ontology) and train with the paper's hyperparameters: batch_size=128, lr=0.0005, hidden_dims=1024, num_hidden_layers=3. Validate that the mixed spectrum output is 1000-bin and the topic head emits T-topic probability distributions, then save checkpoint module outputs (e.g. spectra_topic_check.npz).

## Related tools

- **PyTorch** (Implements the learnable low-rank factors Dl of Ql = Dl Dl^T, the mixing residual weight, and the auxiliary two-layer LDA-topic multi-task head (pytorch=1.7.1)) — https://github.com/HassounLab/ESP
- **numpy** (Loads the preprocessed spectral training data and spectra/topic label tensors used to train the mixing and topic modules) — https://github.com/HassounLab/ESP
- **scipy** (Reads the preprocessed spectral tensors from the NIST-20 subset files alongside numpy) — https://github.com/HassounLab/ESP
- **dgl** (Supports the GNN base model whose 1000-bin predicted spectra are mixed by the low-rank matrix (dgl=0.6.1)) — https://github.com/HassounLab/ESP

## Examples

```
python train.py --cuda 1 --model gnn --disable_two_step_pred --disable_fingerprint --disable_mt_fingerprint --disable_mt_ontology --correlation_mat_rank 100 --model_file_suffix gnn_XX
```

## Evaluation signals

- Forward/backward passes complete without error on the small subset for a few training epochs.
- The mixed spectrum output is exactly 1000-bin, matching the paper's own demo runs (e.g. 'torch_tecand_1000bin_te_cand100').
- The auxiliary topic head emits valid T-topic probability distributions (sums to 1 per spectrum).
- The Namespace printed by train.py shows correlation_mat_rank=100, mt_lda_weight=0.01, and the expected correlation_mix_residual_weight (0.7 GNN / 0.8 MLP) with disable_mt_lda=False.
- Module outputs saved to spectra_topic_check.npz round-trip with the expected shapes.

## Limitations

- NIST-20 training data cannot be redistributed under NIST license regulations; only NPLIB1-trained models and data are published, so reproduction on NIST-20 requires licensed data.
- The low-rank mixing operates at the pipeline's fixed 1000-bin resolution (1000/resolution bins naming convention), so other resolutions change tensor shapes.
- Multi-task LDA labels require precomputed spectral topics via Latent Dirichlet Allocation (van der Hooft et al. topic modeling), an additional preprocessing dependency.

## Evidence

- [intro] Objective: enhance the MLP and GNN base models via multi-tasking on additional data (spectral topic labels from LDA) and an attention mechanism over spectra peaks: "the MLP and GNN are enhanced by: 1) multi-tasking on additional data (spectral topic labels obtained using LDA (Latent Dirichlet Allocation) [3], and 2) attention mechanism to capture dependencies"
- [methods] Workflow adds learnable low-rank mixing matrices applied to the 1000-bin predicted spectra: [evidence span withheld — anchor status: not_found]
- [readme] README demo Namespace shows the concrete parameters for the mixing and topic modules: "disable_mt_lda=False, correlation_mat_rank=100, mt_lda_weight=0.01, correlation_mix_residual_weight=0.7, disable_two_step_pred=True"
- [readme] README documents the flag convention used to enable/train the modules: "python train.py --cuda 1 --model gnn  --disable_two_step_pred --disable_fingerprint --disable_mt_fingerprint --disable_mt_ontology --correlation_mat_rank 100 --model_file_suffix gnn_XX"
