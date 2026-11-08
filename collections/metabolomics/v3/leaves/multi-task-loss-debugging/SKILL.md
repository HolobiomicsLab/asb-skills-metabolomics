---
name: multi-task-loss-debugging
description: Use when training or restoring ESP MLP/GNN models with multi-task LDA
  topic prediction and peak-correlation label mixing enabled, and forward/backward
  passes or output shapes must be validated.
when_to_use_negative:
- Only evaluating a pretrained model on a test candidate set (e.g. torch_tecand_1000bin_te_cand100)
  with no multi-task heads active — the loss-debugging path does not apply
- Reproducing EI/GC-MS NEIMS-style spectra — the ESP setup is ESI/LC-MS and the code
  cannot redistribute NIST-20 training data
- Training the ESP ensemble weighting itself (ens_train_canopus.py ranking task) rather
  than debugging the per-base-model multi-task heads
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_0091
tools:
- name: PyTorch
  role: Implements the learnable low-rank mixing matrices Q_l = D_l D_l^T, the auxiliary
    two-layer multi-task LDA-topic head, and the forward/backward passes being debugged
  repo: https://github.com/HassounLab/ESP
- name: dgl
  role: Graph backend for the GNN base predictor whose multi-task heads are being
    validated (dgl=0.6.1)
  repo: https://github.com/HassounLab/ESP
- name: numpy
  role: Loads the preprocessed spectral training data and spectra/topic label tensors
    for the debug run and writes the validation npz dump
  repo: https://github.com/HassounLab/ESP
- name: scikit-learn
  role: Source of the LDA (Latent Dirichlet Allocation) spectral topic labels used
    as multi-task targets
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/bioinformatics/btae490
    title: An Ensemble Spectral Prediction (ESP) model for metabolite annotation
  source_content_hash: sha256:8c7efe5d2092d4a688dfc0da65d9e38638d69da3c3eb2ee6b3c8306bbab03851
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/HassounLab/ESP
  tools:
  - PyTorch
  - dgl
  - numpy
  - rdkit
  - scikit-learn
  - scipy
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5333333333333333
n_claims: 15
---

# multi-task-loss-debugging

## Summary

Debugs the ESP multi-task/peak-dependency extensions by enabling the LDA-topic auxiliary head and the rank-limited label-mixing matrix (Q_l = D_l D_l^T) in the MLP/GNN predictors, then verifying forward/backward passes produce 1000-bin spectra and T-topic distributions.

## When to use

Use when training or restoring ESP MLP/GNN models with multi-task LDA topic prediction and peak-correlation label mixing enabled, and forward/backward passes or output shapes must be validated.

## When NOT to use

- Only evaluating a pretrained model on a test candidate set (e.g. torch_tecand_1000bin_te_cand100) with no multi-task heads active — the loss-debugging path does not apply
- Reproducing EI/GC-MS NEIMS-style spectra — the ESP setup is ESI/LC-MS and the code cannot redistribute NIST-20 training data
- Training the ESP ensemble weighting itself (ens_train_canopus.py ranking task) rather than debugging the per-base-model multi-task heads

## Inputs

- Preprocessed spectral training tensors from data/final_5 (torch_trvate_1000bin.pkl style files)
- mol_dict.pkl mapping InChiKeys to rdkit mol objects
- pos_train.csv (NPLIB1 dataset in csv format)
- PyTorch model hyperparameters: hidden_dims=1024, num_hidden_layers=3, batch_size=128, lr=0.0005

## Outputs

- Mixed 1000-bin predicted spectrum tensor from the label-mixing module
- T-topic LDA spectral-topic probability distribution from the auxiliary multi-task head
- Checkpoint file best_model_' + model_file_suffix + '.pt (e.g. best_model_gnn_pd.pt)
- Validation dump spectra_topic_check.npz

## How to apply

Load the preprocessed NIST-20/NPLIB1 spectral tensors from data/final_5 (pos_train.csv, mol_dict.pkl) and enable the two peak-dependency modules in train.py or ens_train_canopus.py. Keep multi-tasking on LDA spectral-topic labels active by leaving --disable_mt_lda unset (the demo defaults show disable_mt_lda=False) with mt_lda_weight=0.01 so the auxiliary two-layer head emits T-topic probability distributions alongside the 1000-bin spectrum. Enable the learnable low-rank mixing matrix Q_l = D_l D_l^T over the 1000 predicted bins via --correlation_mat_rank 100 and a mixing residual weight of --correlation_mix_residual_weight 0.7 for GNN (0.8 for MLP, per the ESP namespace). Run with the paper's small-subset settings (batch_size=128, lr=0.0005, hidden_dims=1024, num_hidden_layers=3, epochs=50) and confirm backward passes complete and the mixed spectrum output is 1000-bin. Persist the trained-checkpoint module outputs (e.g. to spectra_topic_check.npz) and compare against the demo checkpoint suffixes like best_model_mlp_pd.pt / best_model_gnn_pd.pt.

## Related tools

- **PyTorch** (Implements the learnable low-rank mixing matrices Q_l = D_l D_l^T, the auxiliary two-layer multi-task LDA-topic head, and the forward/backward passes being debugged) — https://github.com/HassounLab/ESP
- **dgl** (Graph backend for the GNN base predictor whose multi-task heads are being validated (dgl=0.6.1)) — https://github.com/HassounLab/ESP
- **numpy** (Loads the preprocessed spectral training data and spectra/topic label tensors for the debug run and writes the validation npz dump) — https://github.com/HassounLab/ESP
- **scikit-learn** (Source of the LDA (Latent Dirichlet Allocation) spectral topic labels used as multi-task targets)

## Examples

```
python train.py --cuda 1 --model gnn --disable_two_step_pred --disable_fingerprint --disable_mt_fingerprint --disable_mt_ontology --correlation_mat_rank 100 --model_file_suffix gnn_XX
```

## Evaluation signals

- Training completes epochs without error on the NPLIB1 subset and the argparse Namespace shows disable_mt_lda=False, correlation_mat_rank=100, mt_lda_weight=0.01, correlation_mix_residual_weight=0.7 (GNN) / 0.8 (MLP)
- Mixed spectrum output tensor is exactly 1000-bin (matching demos like torch_tecand_1000bin_te_cand100)
- Auxiliary multi-task head emits T-topic probability distributions summing to 1 over topics
- Average-rank/Rank@K metrics are in the expected regime seen in the README demo (e.g. MLP average rank ~339.350 ± 1264.715, Rank@1 0.230 on NPLIB1 full candidates)
- Checkpoint file best_model_' + model_file_suffix + '.pt is written and can be restored by a subsequent train.py run

## Limitations

- NIST-20 training data cannot be published alongside the NIST-20 models; published model and datasets cover NPLIB1 only
- The mixing matrix is rank-limited by --correlation_mat_rank, so full peak-correlation structure is approximated, not exact
- Two large files (mol_dict.pkl, pos_train.csv) must be fetched from Zenodo before data_trvate.py/data_tecand.py can run

## Evidence

- [intro] multi-tasking on additional data (spectral topic labels obtained using LDA (Latent Dirichlet Allocation) [3], and 2) attention mechanism to capture dependencies: "the MLP and GNN are enhanced by: 1) multi-tasking on additional data (spectral topic labels obtained using LDA (Latent Dirichlet Allocation) [3], and 2) attention mechanism to capture dependencies"
- [readme] disable_mt_lda=False, correlation_mat_rank=100, mt_lda_weight=0.01, correlation_mix_residual_weight=0.7: "disable_mt_lda=False, correlation_mat_rank=100, mt_lda_weight=0.01, correlation_mix_residual_weight=0.7, disable_two_step_pred=True, disable_reverse=False, disable_fingerprint=True"
- [readme] This script is used for training and restoring saved GNN and MLP models.: "#### `train.py`
This script is used for training and restoring saved GNN and MLP models."
- [methods] Install the required packages under conda (optional, can install only key requirements mentioned above):: "Install the required packages under conda (optional, can install only key requirements mentioned above):"
