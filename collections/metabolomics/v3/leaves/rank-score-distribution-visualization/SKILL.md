---
name: rank-score-distribution-visualization
description: Use when you need to benchmark a retention-order predictor's generalizability
  across many independent RPLC datasets and present the rank-score distribution against
  a fingerprint baseline.
when_to_use_negative:
- The candidate molecules are outside the conventional elemental restrictions of C,
  H, N, O, P, and S, or the chromatographic system is HILIC rather than RPLC at eluent
  pH around 2.7 — ROASMI does not generalize to these spaces.
- The two compounds' chromatographic peaks are too close (indistinguishable pairs,
  e.g. 4-hydroxybenzoic acid vs 2,5-dihydroxybenzoic acid) — their retention order
  is not reproducible and rank-score visualization would overstate distinguishability.
- You already have per-dataset retention-order metrics and only need to compare top-k
  or MS/MS score integration results rather than assess rank-score generalizability.
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: ROASMI
  role: Provides the trained RankNet retention-order model (D-MPNN embeddings) whose
    per-dataset rank scores are aggregated and visualized
  repo: https://github.com/FangYuan717/ROASMI
- name: chemprop
  role: Underlying D-MPNN molecular embedding module used to produce compound representations
    fed into RankNet for retention order prediction
  repo: https://github.com/chemprop/chemprop
- name: PyTorch
  role: Deep learning backend used to run the chemprop-based embedding and RankNet
    inference during the 71-dataset evaluation
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1186/s13321-025-00968-8
    title: 'ROASMI: accelerating small molecule identification by repurposing retention
      data'
  source_content_hash: sha256:8a51563f13ec9624ae2f7693bfb0b03a8fb465d720387c6cef9f766e445e3f8d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/FangYuan717/ROASMI
  tools:
  - PyTorch
  - ROASMI
  - chemprop
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# Rank Score Distribution Visualization

## Summary

Aggregates per-dataset retention-order rank scores from the ROASMI ensemble (D-MPNN embeddings plus RankNet) and the Morgan_count fingerprint baseline across 71 independent RPLC datasets, and visualizes the resulting rank-score distributions and average comparisons (0.696 vs 0.795) as Fig. 5 panels.

## When to use

Use when you need to benchmark a retention-order predictor's generalizability across many independent RPLC datasets and present the rank-score distribution against a fingerprint baseline.

## When NOT to use

- The candidate molecules are outside the conventional elemental restrictions of C, H, N, O, P, and S, or the chromatographic system is HILIC rather than RPLC at eluent pH around 2.7 — ROASMI does not generalize to these spaces.
- The two compounds' chromatographic peaks are too close (indistinguishable pairs, e.g. 4-hydroxybenzoic acid vs 2,5-dihydroxybenzoic acid) — their retention order is not reproducible and rank-score visualization would overstate distinguishability.
- You already have per-dataset retention-order metrics and only need to compare top-k or MS/MS score integration results rather than assess rank-score generalizability.

## Inputs

- 71 independent RPLC datasets (experimental elution orders with compound pairs and measured retention times)
- Initial (unretrained) ROASMI ensemble checkpoint (ROASMI_1–ROASMI_5) from the ROASMI package data directory
- Morgan_count fingerprint baseline predictions for pairwise retention order on the same 71 datasets

## Outputs

- table2.csv — per-dataset metrics (accuracy, Spearman correlation, rank score) for ROASMI and Morgan_count across the 71 RPLC datasets
- fig5b.png – fig5e.png — rank-score distribution panels and average rank score comparison (ROASMI 0.696 vs initial-set 0.795)

## How to apply

Load the 71 independent RPLC datasets and the initial, unretrained ROASMI ensemble checkpoint (ROASMI_1–ROASMI_5) from the ROASMI package data directory. For each dataset, use chemprop-based D-MPNN embeddings with the RankNet retention prediction module to predict pairwise retention orders for all compound pairs without retraining, and run the identical evaluation with the Morgan_count fingerprint baseline. Compute per-dataset accuracy, Spearman correlation, and rank score against experimental elution orders, then assemble the per-dataset metrics into a table (Table 2, written as table2.csv). Generate the Fig. 5b–e panels showing rank-score distributions across the 71 datasets for ROASMI vs Morgan_count and the average rank score comparison (ROASMI 0.696 vs the initial reference set's 0.795). Validate the computed average rank score over the 71 datasets matches the reported 0.696 within Monte-Carlo seed tolerance.

## Related tools

- **ROASMI** (Provides the trained RankNet retention-order model (D-MPNN embeddings) whose per-dataset rank scores are aggregated and visualized) — https://github.com/FangYuan717/ROASMI
- **chemprop** (Underlying D-MPNN molecular embedding module used to produce compound representations fed into RankNet for retention order prediction) — https://github.com/chemprop/chemprop
- **PyTorch** (Deep learning backend used to run the chemprop-based embedding and RankNet inference during the 71-dataset evaluation)

## Examples

```
python code/ROASMI_predict.py  # per-dataset rank scores then assembled into table2.csv and fig5b–e.png
```

## Evaluation signals

- Average rank score over the 71 independent RPLC datasets reproduces the reported value of 0.696 (within Monte-Carlo seed tolerance)
- The initial reference set's rank score is 0.795, serving as the baseline against the cross-dataset average
- table2.csv contains per-dataset rows with accuracy, Spearman correlation, and rank score columns for both ROASMI and Morgan_count
- Fig. 5b–e panels render rank-score distributions and the ROASMI vs Morgan_count average comparison without retraining artifacts
- No retraining is performed: the initial (unretrained) ROASMI checkpoint is used throughout, confirming evaluation of generalizability rather than refit performance

## Limitations

- The evaluation is restricted to RPLC systems at eluent pH around 2.7; HILIC retention models with generalizability remain future work.
- Model uncertainty is quantified only through the variance of retention-order predictions across the ensemble members (ROASMI_1–ROASMI_5), so rank-score distributions reflect ensemble variance rather than epistemic uncertainty of the architecture itself.
- When two compounds' chromatographic peaks are very close, their retention order may not be consistent across experiments, inflating or deflating per-dataset rank scores.

## Evidence

- [abstract] Generalizability benchmark across 71 RPLC datasets: "The generalizability of ROASMI is proven by 71 independent reversed-phase liquid chromatography (RPLC) datasets."
- [results] Per-dataset rank score results from task card: [evidence span withheld — anchor status: not_found]
- [methods] Underlying model architecture used for ranking: "A Ranking Neural Network (RankNet) was used to learn elution orders"
- [readme] Ensemble uncertainty definition: "We provide four initial ROASMI models (ROASMI_1 - ROASMI_5) for predicting the retention behavior of compounds in the reversed-phase liquid chromatography (RPLC) system with an eluent pH of around"
