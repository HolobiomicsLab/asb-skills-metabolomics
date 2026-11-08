---
name: mass-spectra-annotation-ranking
description: Use when you need to rank molecular candidates for an MS/MS spectrum
  (ESI/LC-MS, 1000-bin resolution, cand_size=100 test sets) and want average rank
  / Rank@K metrics from predicted spectra, or when combining MLP and GNN spectral
  predictions.
when_to_use_negative:
- Data is EI/GC-MS spectra — the article's NEIMS-based MLP was generalized to ESI/LC-MS
  but not EI/GC-MS data
- The NIST-20 dataset — models trained on NIST-20 are not published alongside the
  code, so this skill cannot be retrained on NIST-20
- You have no pretrained MLP/GNN models and want to train ESP — ens_train_canopus.py
  requires pretrained MLP and GNN models first
edam_topics:
- http://edamontology.org/topic_0003
- http://edamontology.org/topic_3520
tools:
- name: Ensemble Spectral Prediction (ESP)
  role: The core ensemble model that combines MLP and GNN spectral predictions via
    a ranking-trained classifier
  repo: https://github.com/HassounLab/ESP
- name: PyTorch
  role: Backend for training the MLP, GNN, and ESP ensemble models and computing ranking
    losses
  repo: https://github.com/HassounLab/ESP
- name: dgl
  role: Graph construction/processing for GNN-based molecular candidate representations
  repo: https://github.com/HassounLab/ESP
- name: PyTorch Geometric
  role: Generates the torchgeometric candidate graph files used at test time
  repo: https://github.com/HassounLab/ESP
- name: rdkit
  role: Provides mol objects for candidates via mol_dict.pkl (fingerprint/formula
    features)
  repo: https://github.com/HassounLab/ESP
provenance:
  source_task_ids:
  - task_006
  - task_007
  source_papers:
  - doi: 10.1093/bioinformatics/btae490
    title: An Ensemble Spectral Prediction (ESP) model for metabolite annotation
  source_content_hash: sha256:49695f2b9b10b353b0d98c5fcb5db6acb7d88303fb594c2b4786455a34424c39
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/HassounLab/ESP
  tools:
  - Ensemble Spectral Prediction (ESP)
  - PyTorch
  - PyTorch Geometric
  - Python
  - conda
  - dgl
  - numpy
  - rdkit
  - scikit-learn
  - scipy
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0003
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6333333333333333
n_claims: 30
---

# mass-spectra-annotation-ranking

## Summary

Ranking-based metabolite annotation that scores candidate molecules against a query MS/MS spectrum by predicting spectra with MLP and GNN models and combining their predictions via a trained ensemble (ESP), evaluated with average rank and Rank@K on PubChem/NPLIB1 candidate sets.

## When to use

Use when you need to rank molecular candidates for an MS/MS spectrum (ESI/LC-MS, 1000-bin resolution, cand_size=100 test sets) and want average rank / Rank@K metrics from predicted spectra, or when combining MLP and GNN spectral predictions.

## When NOT to use

- Data is EI/GC-MS spectra — the article's NEIMS-based MLP was generalized to ESI/LC-MS but not EI/GC-MS data
- The NIST-20 dataset — models trained on NIST-20 are not published alongside the code, so this skill cannot be retrained on NIST-20
- You have no pretrained MLP/GNN models and want to train ESP — ens_train_canopus.py requires pretrained MLP and GNN models first

## Inputs

- pos_train.csv (NPLIB1 training spectra, from zenodo.org/records/10929401)
- mol_dict.pkl (InChIKey → rdkit mol object mapping, from zenodo.org/records/10929354)
- Pretrained model files best_model_mlp_can.pt, best_model_gnn_can.pt, ESP_can.pt
- Test candidate graph dataset torch_tecand_1000bin_te_cand100_torchgeometric.pkl

## Outputs

- Average rank ± standard deviation over the test set
- Rank@1 through Rank@20 fractions (e.g. MLP 0.230@1, GNN 0.115@1, ESP 0.187@1 on NPLIB1)
- Predicted 1000-bin spectra used for candidate ranking

## How to apply

Prepare data with data_trvate.py (torch_trvate_1000bin.pkl) and data_tecand.py (torch_tecand_1000bin_te_cand100_torchgeometric.pkl), which embeds each candidate molecule's fingerprint, molecular formula, and InChIKey with PyTorch Geometric/DGL graphs. Train the two base models with train.py — hidden_dims=1024, num_hidden_layers=3, lr=0.0005, drop_ratio=0.3, batch_size=128, epochs=50, correlation_mat_rank=100 — using a ranking objective where the model scores candidates by spectral similarity between the predicted and the true spectrum. Build the ESP ensemble with ens_train_canopus.py --mode 'canopus': group training spectra by molecular formula to form candidate sets, derive dMLP/dGNN labels from each base model's per-group average rank, weight targets by SMAPE between the two base models' spectral predictions, and train a classifier with ensemble_hidden_dim=256, epochs=100 to output a weighted-average spectrum. Evaluate by loading a saved model (best_model_mlp_can.pt / best_model_gnn_can.pt / ESP_can.pt) against torch_tecand_1000bin_te_cand100 and computing average rank ± std plus Rank@1 through Rank@20; the ranking is computed as the position of the true molecule when all candidates are sorted by predicted-vs-true spectral similarity.

## Related tools

- **Ensemble Spectral Prediction (ESP)** (The core ensemble model that combines MLP and GNN spectral predictions via a ranking-trained classifier) — https://github.com/HassounLab/ESP
- **PyTorch** (Backend for training the MLP, GNN, and ESP ensemble models and computing ranking losses) — https://github.com/HassounLab/ESP
- **dgl** (Graph construction/processing for GNN-based molecular candidate representations) — https://github.com/HassounLab/ESP
- **PyTorch Geometric** (Generates the torchgeometric candidate graph files used at test time) — https://github.com/HassounLab/ESP
- **rdkit** (Provides mol objects for candidates via mol_dict.pkl (fingerprint/formula features)) — https://github.com/HassounLab/ESP

## Examples

```
python ens_train_canopus.py --cuda 0 --disable_two_step_pred --disable_fingerprint --disable_mt_fingerprint --disable_mt_ontology --correlation_mat_rank 100 --full_dataset --mode 'canopus'
```

## Evaluation signals

- Reproduced GNN average rank should be ≈241.753 ± 939.827 and GNN Rank@1 ≈0.115 on NPLIB1 te_cand100
- Reproduced MLP Rank@1 ≈0.230 and Rank@10 ≈0.520 (higher than ESP 0.490 and GNN 0.464 at top ranks)
- Reproduced ESP average rank ≈279.557 ± 1170.300 with Rank@1 0.187 and Rank@20 0.592
- Ranking curve must be monotonic non-decreasing from Rank@1 to Rank@20 for every model
- Molecular formula grouping must cover all training spectra so every formula stratum yields a valid candidate set for ensemble training

## Limitations

- ESP does not always win: on NPLIB1, GNN-PD has the best average rank (~241.8) while MLP-PD leads at Rank@1–20 (0.230@1); ESP is intermediate (0.187@1)
- Annotation performance depends on the training dataset, the number of molecules in the candidate set, and candidate similarity to the target molecule
- NIST-20 models and data cannot be redistributed, so the skill is constrained to the published NPLIB1 configuration

## Evidence

- [abstract] ESP improves average rank over the MLP and GNN baselines when trained on NIST-20 and evaluated on PubChem candidate sets: "ESP improves average rank by 23.7% and 37.2% over the MLP and GNN baselines, respectively, demonstrating performance gain over state-of-the-art neural network approaches"
- [abstract] MLP approaches remain strong contenders when considering top five ranks: "However, MLP approaches remain strong contenders when considering top five ranks."
- [abstract] Annotation performance depends on the training dataset, candidate set size, and candidate similarity: "Importantly, we show that annotation performance is dependent on the training dataset, the number of molecules in the candidate set and candidate similarity to the target molecule."
- [readme] ESP ensemble training uses formula-stratified candidate sets: "The published model and published datasets are for NPLIB1 data."
- [readme] Evaluation of pretrained GNN on NPLIB1 test candidates: "Average rank 241.753 +- 939.827
Rank at 1 0.115"
- [readme] Pretrained ESP evaluation on NPLIB1 test candidates: "Average rank 279.557 +- 1170.300
Rank at 1 0.187"
