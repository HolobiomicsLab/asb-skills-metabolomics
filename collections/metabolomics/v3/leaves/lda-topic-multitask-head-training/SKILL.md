---
name: lda-topic-multitask-head-training
description: Use when training or evaluating ESP's MLP/GNN base models where spectral-peak
  dependencies and topic-structured co-occurrence should be exploited — e.g. flags
  like disable_mt_lda=False, mt_lda_weight, correlation_mat_rank appear in the config.
when_to_use_negative:
- When you only want the plain MLP or GNN baselines for comparison — the baselines
  are defined with these enhancements ablated via --disable_* flags
- 'When input spectra are not at the paper''s 1000-bin resolution (resolution parameter
  controls bins: 1000/resolution), since the mixing matrix and topic head shapes depend
  on it'
- When no LDA spectral-topic labels are available for the dataset, as the multi-task
  head requires topic supervision
tools:
- name: PyTorch
  role: Implements the learnable low-rank mixing matrices Ql = Dl Dl^T, the two-layer
    LDA-topic multi-task head, and the training loop
  repo: https://github.com/HassounLab/ESP
- name: dgl
  role: Graph library for the GNN base spectral predictor that carries the multi-task
    head (dgl=0.6.1)
  repo: https://github.com/HassounLab/ESP
- name: numpy
  role: Loads preprocessed spectral and topic label tensors for training
  repo: https://github.com/HassounLab/ESP
- name: scipy
  role: Supports loading/processing of the spectral and topic label data
  repo: https://github.com/HassounLab/ESP
- name: scikit-learn
  role: LDA (Latent Dirichlet Allocation) topic modeling that generates the spectral
    topic labels used for multi-tasking
  repo: https://github.com/HassounLab/ESP
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/bioinformatics/btae490
    title: An Ensemble Spectral Prediction (ESP) model for metabolite annotation
  source_content_hash: sha256:c8f8ae45fce541acf2fd3a3915b86a743e1bb788faaf6cfadd8a9d602bda4e30
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
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6153846153846154
n_claims: 13
---

# lda-topic-multitask-head-training

## Summary

Trains ESP base models (MLP and GNN spectral predictors) with an auxiliary multi-task head that predicts LDA spectral-topic distributions alongside the 1000-bin spectrum, and couples peaks via a rank-limited learnable correlation mixing matrix. This is one of the two enhancement techniques that lift ESP 23.7%/37.2% in average rank over the MLP/GNN baselines.

## When to use

Use when training or evaluating ESP's MLP/GNN base models where spectral-peak dependencies and topic-structured co-occurrence should be exploited — e.g. flags like disable_mt_lda=False, mt_lda_weight, correlation_mat_rank appear in the config.

## When NOT to use

- When you only want the plain MLP or GNN baselines for comparison — the baselines are defined with these enhancements ablated via --disable_* flags
- When input spectra are not at the paper's 1000-bin resolution (resolution parameter controls bins: 1000/resolution), since the mixing matrix and topic head shapes depend on it
- When no LDA spectral-topic labels are available for the dataset, as the multi-task head requires topic supervision

## Inputs

- Preprocessed spectral training/validation/test pickle (torch_trvate_<bins>bin.pkl) from data_trvate.py
- NPLIB1 dataset as pos_train.csv and mol_dict.pkl (InChIKey -> rdkit mol objects) from Zenodo records 10929401 and 10929354
- LDA spectral-topic label tensors for the multi-task supervision
- Model hyperparameters: batch_size, lr, hidden_dims, num_hidden_layers, correlation_mat_rank, mt_lda_weight

## Outputs

- 1000-bin predicted spectrum after low-rank correlation mixing
- T-topic probability distribution from the LDA multi-task head
- Trained checkpoint file best_model_<model_file_suffix>.pt (e.g. best_model_mlp_can.pt, best_model_gnn_can.pt)
- Saved module outputs (e.g. spectra_topic_check.npz) for validation

## How to apply

Load preprocessed spectral tensors from the NIST-20/NPLIB1 subset files (numpy/scipy; NPLIB1 pos_train.csv and mol_dict.pkl from Zenodo records 10929401/10929354). In PyTorch (1.7.1) and DGL (0.6.1), add two modules to the base predictor: (1) a learnable low-rank mixing matrix Ql = Dl Dl^T over the 1000-bin predicted spectrum, controlled by --correlation_mat_rank (default 100) and blended with the residual via correlation_mix_residual_weight (0.7 for GNN, 0.8 for MLP), and (2) an auxiliary two-layer multi-task head that predicts LDA spectral-topic distributions, weighted by mt_lda_weight=0.01. Enable both modules by NOT passing --disable_mt_lda, while setting --disable_two_step_pred, --disable_fingerprint, --disable_mt_fingerprint and --disable_mt_ontology per the train.py/ens_train_canopus.py flag conventions. Train with batch_size=128, lr=0.0005, hidden_dims=1024, num_hidden_layers=3, epochs=50, drop_ratio=0.3. The rationale is that topic labels (LDA, per van der Hooft et al. 2016) provide additional supervision signal so the base models learn dependencies among co-occurring spectral peaks rather than treating bins independently.

## Related tools

- **PyTorch** (Implements the learnable low-rank mixing matrices Ql = Dl Dl^T, the two-layer LDA-topic multi-task head, and the training loop) — https://github.com/HassounLab/ESP
- **dgl** (Graph library for the GNN base spectral predictor that carries the multi-task head (dgl=0.6.1)) — https://github.com/HassounLab/ESP
- **numpy** (Loads preprocessed spectral and topic label tensors for training) — https://github.com/HassounLab/ESP
- **scipy** (Supports loading/processing of the spectral and topic label data) — https://github.com/HassounLab/ESP
- **scikit-learn** (LDA (Latent Dirichlet Allocation) topic modeling that generates the spectral topic labels used for multi-tasking) — https://github.com/HassounLab/ESP

## Examples

```
python train.py --cuda 1 --model gnn --disable_two_step_pred --disable_fingerprint --disable_mt_fingerprint --disable_mt_ontology --correlation_mat_rank 100 --model_file_suffix gnn_XX
```

## Evaluation signals

- Training runs without error on the subset; forward/backward passes complete with the flags set so disable_mt_lda is False and mt_lda_weight=0.01 is active
- Mixed spectrum output is exactly 1000-bin while the topic head emits a valid T-topic probability distribution
- Config Namespace echo shows correlation_mat_rank=100, mt_lda_weight=0.01, disable_mt_fingerprint=True, disable_mt_ontology=True, hidden_dims=1024, num_hidden_layers=3
- Checkpoint file best_model_<suffix>.pt is produced and reloaded for evaluation with the same flag set
- Average rank / Rank@K metrics are reported on the test set (pretrained demo reference: GNN average rank 241.753 ± 939.827, MLP 339.350 ± 1264.715)

## Limitations

- NIST-20 training data cannot be redistributed due to NIST license regulations; only NPLIB1-trained models and data are published
- The reported gains (23.7% over MLP, 37.2% over GNN in average rank) are on the full NIST candidate set; MLP approaches remain strong contenders at top-5 ranks
- Topic-head benefit depends on mt_lda_weight=0.01 and correlation_mat_rank=100 choices; correlation_mix_residual_weight differs between MLP (0.8) and GNN (0.7)

## Evidence

- [readme] the MLP and GNN are enhanced by: 1) multi-tasking on additional data (spectral topic labels obtained using LDA (Latent Dirichlet Allocation): "the MLP and GNN are enhanced by: 1) multi-tasking on additional data (spectral topic labels obtained using LDA (Latent Dirichlet Allocation)"
- [intro] multi-tasking on additional data (spectral topic labels from LDA): [evidence span withheld — anchor status: not_found]
- [readme] disable_mt_lda=False, correlation_mat_rank=100, mt_lda_weight=0.01: "disable_mt_lda=False, correlation_mat_rank=100, mt_lda_weight=0.01"
- [readme] The generated file is torch_trvate_+str(int(1000/hp['resolution']))+bin.pkl: "The generated file is `torch_trvate_`+str(int(1000/hp['resolution']))+`bin.pkl`."
- [abstract] ESP improves average rank by 23.7% and 37.2% over the MLP and GNN baselines, respectively: "ESP improves average rank by 23.7% and 37.2% over the MLP and GNN baselines, respectively"
