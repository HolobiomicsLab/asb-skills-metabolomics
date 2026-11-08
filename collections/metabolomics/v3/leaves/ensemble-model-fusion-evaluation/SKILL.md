---
name: ensemble-model-fusion-evaluation
description: Use when you have two pretrained spectral prediction baselines (MLP-PD
  and GNN-PD), a ranking-task training setup, and a candidate-set evaluation (average
  rank / Rank@K) on spectra-to-candidate annotation.
when_to_use_negative:
- Input data is EI/GC-MS spectra rather than the ESI/LC-MS-style spectra this pipeline
  generalizes; the baselines were built as an implementation of the NEIMS model with
  generalized ESI/LC-MS but not EI/GC-MS data.
- You need only a single predictor (MLP or GNN) rather than a fused ensemble; the
  skill's value is specifically the weighted-average MLP+GNN combination.
- The training dataset or candidate-set composition differs substantially from NPLIB1/NIST-20-style
  annotation tasks where candidate similarity to the target molecule matters; annotation
  performance is dependent on the training dataset, the number of molecules in the
  candidate set, and candidate similarity to the target molecule.
edam_operation: http://edamontology.org/operation_3359
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3474
tools:
- name: Ensemble Spectral Prediction (ESP)
  role: Fuses enhanced MLP and GNN spectral predictions into a single ensemble model
    trained on ranking tasks for metabolite annotation.
  repo: https://github.com/HassounLab/ESP
- name: PyTorch
  role: Underlying deep-learning framework used by train.py and ens_train_canopus.py
    for the MLP, GNN, and ESP models.
  repo: https://pytorch.org/
- name: PyTorch Geometric
  role: Handles torchgeometric representation of test candidate files (torch_tecand_*_torchgeometric.pkl).
  repo: https://pytorch-geometric.readthedocs.io/
- name: dgl
  role: Graph neural network backend for the GNN baseline fused into the ensemble.
  repo: https://www.dgl.ai/
- name: rdkit
  role: Produces mol objects mapped by InChiKeys in mol_dict.pkl used for data preparation.
  repo: https://www.rdkit.org/
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btae490
    title: An Ensemble Spectral Prediction (ESP) model for metabolite annotation
  source_content_hash: sha256:89c7b3981254f0f0b5f74f2964e50babeeeafb23b36d3f5aa67f16e59da3e568
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/HassounLab/ESP
  tools:
  - Ensemble Spectral Prediction (ESP)
  - PyTorch
  - PyTorch Geometric
  - Python
  - dgl
  - numpy
  - rdkit
  - scikit-learn
  - scipy
  edam_operation: http://edamontology.org/operation_3359
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 14
---

# ensemble-model-fusion-evaluation

## Summary

Fuses independently trained MLP and GNN spectral predictors into an Ensembled Spectral Prediction (ESP) model trained on ranking tasks, then evaluates average rank and Rank@K on candidate sets. ESP improves average rank by 23.7% (MLP) and 37.2% (GNN) over the baselines on the full NIST candidate set.

## When to use

Use when you have two pretrained spectral prediction baselines (MLP-PD and GNN-PD), a ranking-task training setup, and a candidate-set evaluation (average rank / Rank@K) on spectra-to-candidate annotation.

## When NOT to use

- Input data is EI/GC-MS spectra rather than the ESI/LC-MS-style spectra this pipeline generalizes; the baselines were built as an implementation of the NEIMS model with generalized ESI/LC-MS but not EI/GC-MS data.
- You need only a single predictor (MLP or GNN) rather than a fused ensemble; the skill's value is specifically the weighted-average MLP+GNN combination.
- The training dataset or candidate-set composition differs substantially from NPLIB1/NIST-20-style annotation tasks where candidate similarity to the target molecule matters; annotation performance is dependent on the training dataset, the number of molecules in the candidate set, and candidate similarity to the target molecule.

## Inputs

- Prepared [M+H]+ NPLIB1 training/validation/test data (torch_trvate_1000bin.pkl generated from pos_train.csv and mol_dict.pkl)
- Test candidate file torch_tecand_1000bin_te_cand100_torchgeometric.pkl from data_tecand.py
- Pretrained MLP and GNN model checkpoints (e.g. best_model_mlp_can.pt, best_model_gnn_can.pt)

## Outputs

- Fused ESP model checkpoint (e.g. ESP_can.pt)
- Per-model ranking metrics: average rank ± standard deviation and Rank@1 through Rank@20 for MLP-PD, GNN-PD, and ESP

## How to apply

Train (or load) the MLP and GNN base models with `train.py` using the ranking objective at batch_size=128, epochs=50, hidden_dims=1024, num_hidden_layers=3, lr=0.0005, and correlation_mat_rank=100; each is enhanced by multi-tasking on LDA spectral topic labels and an attention mechanism over spectra peaks. Then run `ens_train_canopus.py --mode 'canopus'` with the pretrained MLP and GNN suffixes to train the ESP ensemble (ensemble_hidden_dim=256, epochs=100, lr=0.001) that generates the weighted-average MLP+GNN spectral predictions on ranking tasks. Finally evaluate each model (MLP-PD, GNN-PD, ESP) on the test candidate set `torch_tecand_1000bin_te_cand100_torchgeometric.pkl` by computing rank@1 through rank@20 and average rank with standard deviation. Decide the best model per the article's guidance: ESP for lowest average rank, but MLP-PD remains a strong contender for top-five rank performance.

## Related tools

- **Ensemble Spectral Prediction (ESP)** (Fuses enhanced MLP and GNN spectral predictions into a single ensemble model trained on ranking tasks for metabolite annotation.) — https://github.com/HassounLab/ESP
- **PyTorch** (Underlying deep-learning framework used by train.py and ens_train_canopus.py for the MLP, GNN, and ESP models.) — https://pytorch.org/
- **PyTorch Geometric** (Handles torchgeometric representation of test candidate files (torch_tecand_*_torchgeometric.pkl).) — https://pytorch-geometric.readthedocs.io/
- **dgl** (Graph neural network backend for the GNN baseline fused into the ensemble.) — https://www.dgl.ai/
- **rdkit** (Produces mol objects mapped by InChiKeys in mol_dict.pkl used for data preparation.) — https://www.rdkit.org/

## Examples

```
python ens_train_canopus.py --cuda 0 --disable_two_step_pred --disable_fingerprint --disable_mt_fingerprint --disable_mt_ontology --correlation_mat_rank 100 --full_dataset --mode 'canopus'
```

## Evaluation signals

- ESP should yield a lower average rank than the MLP and GNN baselines (reproduced NPLIB1 values: ESP 279.557 ± 1170.300 vs MLP 339.350 ± 1264.715 and GNN 241.753 ± 939.827 — ESP sits between the two baselines and improves over MLP; the paper reports a 23.7% MLP and 37.2% GNN average-rank improvement on the full NIST candidate set).
- Rank@1/Rank@5 ordering should be checked: on NPLIB1 MLP-PD has the highest Rank@1 (~0.230) and Rank@10 (~0.520), so a fused model that does not beat MLP on top ranks is expected and not a failure.
- The output should contain 20 Rank@K values (Rank at 1 through Rank at 20) plus one Average rank line per evaluated model.
- MLP and GNN base models must be trained with the stated hyperparameters (hidden_dims=1024, num_hidden_layers=3, correlation_mat_rank=100) before running the ensemble step.
- The ensemble script requires pretrained MLP and GNN checkpoints and fails or produces a degenerate model if `--mlp_model_file_suffix` / `--gnn_model_file_suffix` are not supplied.

## Limitations

- NIST-20 training data cannot be published alongside the NIST-20-trained models due to NIST license regulations; only NPLIB1 models and data are distributed.
- On NPLIB1, GNN-PD achieves the lowest average rank while MLP-PD consistently outperforms GNN-PD and ESP at ranks 1 through 20 — ensemble gains are metric-dependent.
- Annotation performance is dependent on the training dataset, the number of molecules in the candidate set, and candidate similarity to the target molecule.

## Evidence

- [abstract] ESP improves average rank by 23.7% and 37.2% over the MLP and GNN baselines, respectively: "ESP improves average rank by 23.7% and 37.2% over the MLP and GNN baselines, respectively"
- [abstract] However, MLP approaches remain strong contenders when considering top five ranks.: "However, MLP approaches remain strong contenders when considering top five ranks."
- [methods] This script is used for training and restoring saved ESP models on NPLIB1 dataset.: "This script is used for training and restoring saved ESP models on NPLIB1 dataset."
- [readme] To run the saved GNN model on a test set, set `--te_cand_dataset_suffix` to the desired test set and `--model_file_suffix` to the model which you wish to use.: "To run the saved GNN model on a test set, set `--te_cand_dataset_suffix` to the desired test set and `--model_file_suffix` to the model which you wish to use."
- [readme] The pretrained MLP, GNN, and ESP models on the NPLIB1 dataset are `/pretrained_models/best_model_mlp_can.pt`, `/pretrained_models/best_model_gnn_can.pt`, and `/pretrained_models/ESP_can.pt`, respectfully.: "The pretrained MLP, GNN, and ESP models on the NPLIB1 dataset are `/pretrained_models/best_model_mlp_can.pt`, `/pretrained_models/best_model_gnn_can.pt`, and `/pretrained_models/ESP_can.pt`,"
