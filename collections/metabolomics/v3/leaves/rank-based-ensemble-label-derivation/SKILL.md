---
name: rank-based-ensemble-label-derivation
description: Use when training a ranking-based ensemble over spectral predictions
  and you need per-formula candidate sets, dMLP/dGNN labels, and SMAPE-derived importance
  weights from training data alone.
when_to_use_negative:
- You need test-time candidate sets from external databases (e.g. PubChem); this skill
  derives candidates only from training-data formula groups — use data_tecand.py for
  test candidate generation instead.
- You have not yet pretrained the MLP and GNN base models; the README states 'Before
  you train a new ESP model, you must have pretrained MLP and GNN models'.
- You are working with EI/GC-MS data; the published models and this procedure target
  ESI/LC-MS (NPLIB1/NIST-20) spectra.
edam_operation: http://edamontology.org/operation_3936
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3474
tools:
- name: PyTorch
  role: Trains the ensemble classifier fENS with binary cross-entropy loss on SMAPE-derived
    importance-weight targets
  repo: https://github.com/HassounLab/ESP
- name: rdkit
  role: Provides mol objects (via mol_dict.pkl) used for molecular formula grouping
    into candidate strata
  repo: https://github.com/HassounLab/ESP
- name: Ensemble Spectral Prediction (ESP)
  role: The ensemble framework whose rank-based labels and weighted-average predictions
    this skill produces (ens_train_canopus.py)
  repo: https://github.com/HassounLab/ESP
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bioinformatics/btae490
    title: An Ensemble Spectral Prediction (ESP) model for metabolite annotation
  source_content_hash: sha256:7f6f2e5910494238b547b44b866feeeaea6ef432bc27968d3ff7814fd165bf1b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/HassounLab/ESP
  tools:
  - Ensemble Spectral Prediction (ESP)
  - PyTorch
  - Python
  - conda
  - numpy
  - rdkit
  - scikit-learn
  - scipy
  edam_operation: http://edamontology.org/operation_3936
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 16
---

# rank-based-ensemble-label-derivation

## Summary

Derives training labels and importance weights for an ensemble spectral-prediction classifier by computing each base model's (MLP, GNN) average rank within formula-stratified candidate groups and converting SMAPE between their spectral predictions into importance-weight targets. This lets ESP learn to weight MLP vs GNN predictions without external candidate retrieval.

## When to use

Use when training a ranking-based ensemble over spectral predictions and you need per-formula candidate sets, dMLP/dGNN labels, and SMAPE-derived importance weights from training data alone.

## When NOT to use

- You need test-time candidate sets from external databases (e.g. PubChem); this skill derives candidates only from training-data formula groups — use data_tecand.py for test candidate generation instead.
- You have not yet pretrained the MLP and GNN base models; the README states 'Before you train a new ESP model, you must have pretrained MLP and GNN models'.
- You are working with EI/GC-MS data; the published models and this procedure target ESI/LC-MS (NPLIB1/NIST-20) spectra.

## Inputs

- Preprocessed NPLIB1 training/validation/test spectra pickle (torch_trvate_10bin.pkl from data_trvate.py)
- Pretrained MLP base model checkpoint (best_model_mlp_can.pt)
- Pretrained GNN base model checkpoint (best_model_gnn_can.pt)
- mol_dict.pkl mapping InChiKeys to rdkit mol objects (for formula grouping)

## Outputs

- Trained ensemble classifier parameters saved as ESP_can.pt
- Per-formula-group dMLP/dGNN binary labels
- SMAPE-derived importance weights used as ensemble training targets

## How to apply

Load the preprocessed NPLIB1 spectra (e.g. torch_trvate_10bin.pkl) and group training spectra by molecular formula, so each formula group is a molecular candidate set drawn from the training data itself. Restore the pretrained MLP and GNN base models (best_model_mlp_can.pt, best_model_gnn_can.pt) and compute each base model's average rank over every formula group's candidate set; derive binary dMLP/dGNN labels by comparing the two per-group ranks. Within each group compute SMAPE between the two base models' spectral predictions and convert these to importance-weight targets. Train the ensemble classifier fENS in PyTorch (1.7.1) with binary cross-entropy loss on the SMAPE-derived weights as targets and the derived dMLP/dGNN labels as outputs, then save the parameters as ESP_can.pt. The rationale is that candidate sets and supervision come from the training spectra, avoiding retrieval of candidates and spectra from external databases.

## Related tools

- **PyTorch** (Trains the ensemble classifier fENS with binary cross-entropy loss on SMAPE-derived importance-weight targets) — https://github.com/HassounLab/ESP
- **rdkit** (Provides mol objects (via mol_dict.pkl) used for molecular formula grouping into candidate strata) — https://github.com/HassounLab/ESP
- **Ensemble Spectral Prediction (ESP)** (The ensemble framework whose rank-based labels and weighted-average predictions this skill produces (ens_train_canopus.py)) — https://github.com/HassounLab/ESP

## Examples

```
python ens_train_canopus.py --cuda 0 --disable_two_step_pred --disable_fingerprint --disable_mt_fingerprint --disable_mt_ontology --correlation_mat_rank 100 --full_dataset --mode 'canopus'
```

## Evaluation signals

- The ensemble's weighted-average combination of MLP and GNN predictions on the NPLIB1 test set should reproduce Rank at 1 of 0.187 and Rank at 20 of 0.592 (Average rank 279.557 ± 1170.300).
- Each training spectrum must be assigned to exactly one formula group, and every group must contain at least the MLP and GNN predictions for its candidates.
- dMLP/dGNN labels must be binary and consistent with the sign of the per-group rank comparison between the two base models.
- SMAPE-derived importance weights must lie in a valid normalized range and correlate with agreement/disagreement between the two base models' spectral predictions within each group.
- Output checkpoint should be written with the ens_model_file_suffix (starting with 'ESP') and loadable by ens_train_canopus.py in 'canopus' mode.

## Limitations

- NIST-20 training data cannot be redistributed (NIST license regulations), so retraining on NIST-20 requires obtaining the data separately; only NPLIB1-pretrained models and datasets are published.
- Candidate sets are limited to molecules sharing a molecular formula within the training data, so group size and candidate similarity constrain annotation performance.
- Reproducibility details for the discussion section are unavailable in the extracted context.

## Evidence

- [readme] Candidates from training data not external retrieval: [evidence span withheld — anchor status: not_found]
- [readme] SMAPE-to-importance-weights training: [evidence span withheld — anchor status: not_found]
- [readme] Pretrained model files: "The pretrained MLP, GNN, and ESP models on the NPLIB1 dataset are `/pretrained_models/best_model_mlp_can.pt`, `/pretrained_models/best_model_gnn_can.pt`, and `/pretrained_models/ESP_can.pt`"
- [intro] Ranking-task ensemble training objective: "we create an Ensembled Spectral Prediction (ESP) model that is trained on ranking tasks to generate the average weighted MLP and GNN spectral predictions"
- [readme] Prerequisite pretrained base models: "Before you train a new ESP model, you must have pretrained MLP and GNN models (see above instructions)."
- [methods] ESP ensemble training script: "This script is used for training and restoring saved ESP models on NPLIB1 dataset."
