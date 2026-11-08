---
name: mass-spectrometry-annotation-rank-metric-computation
description: Use when evaluating spectral annotation models on a candidate set and
  you need average rank / Rank@K metrics, or relative improvement percentages (e.g.
  23.7% and 37.2%) against baselines.
when_to_use_negative:
- When the models being compared were evaluated on different candidate sets or different
  training datasets (e.g. NPLIB1-published models vs unpublished NIST-20-trained models)
  — ranks are not comparable across these conditions.
- 'When only top-1 or top-5 hit rate is needed for a definitive conclusion: the paper
  notes ''MLP approaches remain strong contenders when considering top five ranks'',
  so Rank@K alone can contradict average-rank conclusions.'
- 'When trying to reproduce NIST-20-trained metric tables end-to-end: NIST-20 data
  is not published alongside the NIST-20-trained models due to NIST license regulations.'
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: numpy
  role: Compute the relative improvement percentages (rank_MLP - rank_ESP)/rank_MLP*100
    and (rank_GNN - rank_ESP)/rank_GNN*100 from the reported average-rank values
- name: scikit-learn
  role: Supporting dependency of the ESP environment (env.yml) used in the scripts
    that print the ranking metrics
- name: PyTorch
  role: Runtime for train.py and ens_train_canopus.py, which emit the 'Average rank
    X +- Y' and 'Rank at K' output lines
- name: Ensemble Spectral Prediction (ESP)
  role: Model under evaluation whose average rank (e.g. 279.557 on NPLIB1) is compared
    to the MLP and GNN baselines
  repo: https://github.com/HassounLab/ESP
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btae490
    title: An Ensemble Spectral Prediction (ESP) model for metabolite annotation
  source_content_hash: sha256:7929832a1bb011db7363b74e33a03506110ffb6a62a48d35ff5b80f6de0f15b5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/HassounLab/ESP
  tools:
  - Ensemble Spectral Prediction (ESP)
  - PyTorch
  - Python
  - numpy
  - scikit-learn
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# mass-spectrometry-annotation-rank-metric-computation

## Summary

Computes average rank and Rank@K (k=1..20) metrics for mass spectrometry metabolite annotation models (MLP, GNN, ESP) on candidate sets, and derives relative improvement percentages such as the 23.7%/37.2% average-rank gains ESP shows over MLP/GNN baselines.

## When to use

Use when evaluating spectral annotation models on a candidate set and you need average rank / Rank@K metrics, or relative improvement percentages (e.g. 23.7% and 37.2%) against baselines.

## When NOT to use

- When the models being compared were evaluated on different candidate sets or different training datasets (e.g. NPLIB1-published models vs unpublished NIST-20-trained models) — ranks are not comparable across these conditions.
- When only top-1 or top-5 hit rate is needed for a definitive conclusion: the paper notes 'MLP approaches remain strong contenders when considering top five ranks', so Rank@K alone can contradict average-rank conclusions.
- When trying to reproduce NIST-20-trained metric tables end-to-end: NIST-20 data is not published alongside the NIST-20-trained models due to NIST license regulations.

## Inputs

- Test candidate file torch_tecand_<resolution>bin_te_cand<cand_size>_torchgeometric.pkl (e.g. torch_tecand_1000bin_te_cand100)
- Pretrained model checkpoints: best_model_mlp_can.pt, best_model_gnn_can.pt, ESP_can.pt (or NPLIB1 variants *_pd / *_fp)
- Reported average-rank values for MLP, GNN, and ESP under the same NIST-20-trained / PubChem candidate-set evaluation
- Command-line hyperparameters: --te_cand_dataset_suffix, --model_file_suffix, --correlation_mat_rank 100, --full_dataset

## Outputs

- Average rank with standard deviation (e.g. 'Average rank 279.557 +- 1170.300')
- Rank at K for K = 1..20 (e.g. Rank at 1 0.187, Rank at 5 0.398, Rank at 20 0.592)
- Relative improvement percentages computed with numpy, validated against the abstract's 23.7% (MLP) and 37.2% (GNN)

## How to apply

Run the saved MLP or GNN model on a test candidate set via train.py (setting --te_cand_dataset_suffix, --model_file_suffix, and disabling two_step_pred/fingerprint/mt_fingerprint/mt_ontology with --correlation_mat_rank 100), or the ESP ensemble via ens_train_canopus.py --mode 'canopus'. The scripts print 'Average rank X +- Y' plus Rank at 1..20 for the test spectra, e.g. ESP gives Average rank 279.557 +- 1170.300 on the NPLIB1 full candidate set. To derive relative improvement, extract average rank values for ESP, MLP, and GNN under the same training data (NIST-20) and candidate-set evaluation, then compute (rank_MLP - rank_ESP) / rank_MLP * 100 and (rank_GNN - rank_ESP) / rank_GNN * 100 with numpy. Crucially, compare models only on the identical candidate set (e.g. torch_tecand_1000bin_te_cand100 or --full_dataset), since annotation performance depends on the training dataset, the number of molecules in the candidate set, and candidate similarity to the target molecule.

## Related tools

- **numpy** (Compute the relative improvement percentages (rank_MLP - rank_ESP)/rank_MLP*100 and (rank_GNN - rank_ESP)/rank_GNN*100 from the reported average-rank values)
- **scikit-learn** (Supporting dependency of the ESP environment (env.yml) used in the scripts that print the ranking metrics)
- **PyTorch** (Runtime for train.py and ens_train_canopus.py, which emit the 'Average rank X +- Y' and 'Rank at K' output lines)
- **Ensemble Spectral Prediction (ESP)** (Model under evaluation whose average rank (e.g. 279.557 on NPLIB1) is compared to the MLP and GNN baselines) — https://github.com/HassounLab/ESP

## Examples

```
python ens_train_canopus.py --cuda 0 --disable_two_step_pred --disable_fingerprint --disable_mt_fingerprint --disable_mt_ontology --correlation_mat_rank 100 --full_dataset --mode 'canopus'
```

## Evaluation signals

- Computed relative improvements must equal 23.7% (vs MLP) and 37.2% (vs GNN) within rounding tolerance, matching the abstract's claim.
- Demo reproduction check: running the pretrained ESP on the NPLIB1 test data with all downloaded candidates must yield Average rank 279.557 +- 1170.300.
- Rank@K values must be monotonically non-decreasing in K (e.g. ESP: Rank at 1 0.187 <= Rank at 20 0.592).
- Average rank must be reported with its standard deviation in the 'Average rank X +- Y' format shown in the demo outputs.
- All compared models must be evaluated on the same --te_cand_dataset_suffix candidate set (or the same --full_dataset setting); mixing candidate sets invalidates the comparison.

## Limitations

- NIST-20-trained models cannot be fully re-evaluated externally because the NIST-20 data is not published alongside the models due to NIST license regulations.
- The published model and datasets cover only NPLIB1 data, so the 23.7%/37.2% NIST-20/PubChem numbers cannot be directly reproduced from released artifacts.
- Average-rank superiority does not guarantee top-K superiority: MLP baselines remain strong contenders when considering top five ranks.
- Annotation performance is dependent on the training dataset, the number of molecules in the candidate set, and candidate similarity to the target molecule.

## Evidence

- [abstract] ESP improves average rank by 23.7% and 37.2% over the MLP and GNN baselines, respectively, demonstrating performance gain: "ESP improves average rank by 23.7% and 37.2% over the MLP and GNN baselines, respectively, demonstrating performance gain"
- [methods] To run the saved GNN model on a test set, set `--te_cand_dataset_suffix` to the desired test set and `--model_file_suffix`: "To run the saved GNN model on a test set, set `--te_cand_dataset_suffix` to the desired test set and `--model_file_suffix`"
- [readme] Average rank 279.557 +- 1170.300: "Average rank 279.557 +- 1170.300"
- [readme] Our results, measured in average rank and Rank@K for the test spectra, show remarkable performance gain: "Our results, measured in average rank and Rank@K for the test spectra, show remarkable performance gain"
