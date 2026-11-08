---
name: smape-spectral-error-weighting
description: 'Use when training the ESP ensemble: NPLIB1 training spectra (torch_trvate_10bin.pkl)
  are grouped by molecular formula, and SMAPE between the two base models'' per-group
  spectral predictions supplies importance-weight targets for fENS instead of external
  candidate retrieval.'
when_to_use_negative:
- You are training the MLP or GNN base models themselves (use train.py; SMAPE weighting
  only concerns the ensemble stage).
- Your spectra are EI/GC-MS rather than the ESI/LC-MS-style data used here — the MLP
  baseline is a NEIMS implementation generalized to ESI/LC-MS, not EI/GC-MS.
- You lack pretrained MLP and GNN models; the README states a new ESP model requires
  pretrained MLP and GNN models first.
tools:
- name: PyTorch
  role: Trains the ensemble classifier fENS with binary cross-entropy loss on SMAPE-derived
    importance-weight targets and dMLP/dGNN labels (pytorch=1.7.1).
  repo: https://github.com/HassounLab/ESP
- name: rdkit
  role: Provides mol objects used to derive molecular formulae that stratify the training
    spectra into candidate groups (mol_dict.pkl maps InChiKeys to rdkit mol objects).
- name: numpy
  role: Numerical computation of SMAPE between base-model spectral prediction vectors
    within each formula group.
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bioinformatics/btae490
    title: An Ensemble Spectral Prediction (ESP) model for metabolite annotation
  source_content_hash: sha256:84454d8ad503ae676642cca2b029e460df073671eeca87f6ef5342babdbda8b2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/HassounLab/ESP
  tools:
  - PyTorch
  - Python
  - conda
  - numpy
  - rdkit
  - scikit-learn
  - scipy
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7368421052631579
n_claims: 19
---

# smape-spectral-error-weighting

## Summary

Within each molecular-formula candidate stratum of the NPLIB1 training spectra, compute SMAPE between the pretrained MLP and GNN spectral predictions and convert these per-group errors into importance-weight targets for training the ESP ensemble classifier fENS (binary cross-entropy), yielding ESP_can.pt.

## When to use

Use when training the ESP ensemble: NPLIB1 training spectra (torch_trvate_10bin.pkl) are grouped by molecular formula, and SMAPE between the two base models' per-group spectral predictions supplies importance-weight targets for fENS instead of external candidate retrieval.

## When NOT to use

- You are training the MLP or GNN base models themselves (use train.py; SMAPE weighting only concerns the ensemble stage).
- Your spectra are EI/GC-MS rather than the ESI/LC-MS-style data used here — the MLP baseline is a NEIMS implementation generalized to ESI/LC-MS, not EI/GC-MS.
- You lack pretrained MLP and GNN models; the README states a new ESP model requires pretrained MLP and GNN models first.

## Inputs

- torch_trvate_10bin.pkl (preprocessed NPLIB1 training/validation/test spectra)
- NPLIB1 training spectra grouped by molecular formula (formula-stratified candidate sets)
- pretrained MLP base model best_model_mlp_can.pt
- pretrained GNN base model best_model_gnn_can.pt
- per-group spectral predictions from the MLP and GNN base models

## Outputs

- SMAPE-derived importance weights per formula-group candidate set
- dMLP/dGNN binary labels derived from per-group average ranks
- trained ensemble classifier fENS with binary cross-entropy loss
- ESP_can.pt (saved ensemble model parameters)

## How to apply

Load the preprocessed NPLIB1 training spectra from torch_trvate_10bin.pkl and group them by molecular formula so each group acts as a molecular candidate set drawn from the training data itself. Restore the pretrained base models best_model_mlp_can.pt (MLP) and best_model_gnn_can.pt (GNN) and compute each model's average rank within every formula group; derive dMLP/dGNN binary labels by comparing the two per-group ranks. Compute SMAPE between the two base models' spectral predictions within each group and convert those errors to importance weights used as regression-style targets. Train fENS with PyTorch under binary cross-entropy loss (lr=0.001, batch_size=128, 100 epochs, ensemble_hidden_dim=256 per the canopus-mode namespace), taking the SMAPE-derived importance weights as targets and the derived dMLP/dGNN labels as outputs. Save the resulting parameters as ESP_can.pt; the rationale is to up-weight spectral bins where the better base model predicts more accurately, so the weighted-average combination ranks the true candidate higher.

## Related tools

- **PyTorch** (Trains the ensemble classifier fENS with binary cross-entropy loss on SMAPE-derived importance-weight targets and dMLP/dGNN labels (pytorch=1.7.1).) — https://github.com/HassounLab/ESP
- **rdkit** (Provides mol objects used to derive molecular formulae that stratify the training spectra into candidate groups (mol_dict.pkl maps InChiKeys to rdkit mol objects).)
- **numpy** (Numerical computation of SMAPE between base-model spectral prediction vectors within each formula group.)

## Examples

```
python ens_train_canopus.py --cuda 0 --disable_two_step_pred --disable_fingerprint --disable_mt_fingerprint --disable_mt_ontology --correlation_mat_rank 100 --full_dataset --mode 'canopus'
```

## Evaluation signals

- Running the saved ESP model on the NPLIB1 test set must reproduce Average rank 279.557 ± 1170.300 with Rank at 1 = 0.187 and Rank at 20 = 0.592.
- The trained ensemble parameters are written to ESP_can.pt (ens_model_file_suffix must start with 'ESP').
- dMLP/dGNN labels must agree with the sign of the MLP-vs-GNN per-group average-rank comparison.
- Importance-weight targets must be bounded SMAPE-derived values (SMAPE ∈ [0, 2] or [0, 1] depending on convention), one set per formula-stratified candidate group.
- Ensemble hyperparameters must match the canopus-mode namespace: lr=0.001, batch_size=128, epochs=100, ensemble_hidden_dim=256, bins=1000.

## Limitations

- NIST-20 training data cannot be published alongside the models trained on it, so retraining weights from the full NIST-20 corpus requires user-supplied data; published models/datasets cover only NPLIB1.
- Annotation performance depends on the training dataset, the number of molecules in the candidate set, and candidate similarity to the target molecule.
- Candidate sets come from the training data's own formula groups; the NIST-2020 abstract-level evaluation used relevant candidate sets from PubChem, which is a separate evaluation path.

## Evidence

- [methods] workflow card (task_007): [evidence span withheld — anchor status: not_found]
- [readme] README: "The pretrained MLP, GNN, and ESP models on the NPLIB1 dataset are `/pretrained_models/best_model_mlp_can.pt`, `/pretrained_models/best_model_gnn_can.pt`, and `/pretrained_models/ESP_can.pt`"
- [intro] article: "we create an Ensembled Spectral Prediction (ESP) model that is trained on ranking tasks to generate the average weighted MLP and GNN spectral predictions."
