---
name: spearman-correlation-computation
description: Use when evaluating a retention-order prediction model (e.g. ROASMI,
  Morgan_count fingerprints) against experimental elution orders on multiple RPLC
  datasets, where per-dataset rank agreement must be aggregated into an average rank
  score.
when_to_use_negative:
- Input is already a feature table or embeddings without paired experimental elution-order
  labels — Spearman computation cannot verify rank agreement without ground-truth
  order.
- Evaluating on HILIC data — ROASMI's retention models are RPLC-specific (eluent pH
  ≈ 2.7) and HILIC generalizability is explicitly future work.
- Comparing a single compound's retention value against a threshold (e.g. RT outlier
  detection) rather than pairwise ranking across compounds in a dataset.
edam_operation: http://edamontology.org/operation_0220
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: ROASMI
  role: Source of the unretrained ensemble used to generate predicted retention orders
    for Spearman comparison against experimental elution orders on each of the 71
    datasets
  repo: https://github.com/FangYuan717/ROASMI
- name: chemprop
  role: Provides the D-MPNN molecular embedding module whose output feeds the RankNet
    retention prediction module used to produce the predicted ranks being correlated
  repo: https://github.com/chemprop/chemprop
- name: PyTorch
  role: Backend framework for running the chemprop-based embeddings and RankNet module
    used in the retention-order predictions
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1186/s13321-025-00968-8
    title: 'ROASMI: accelerating small molecule identification by repurposing retention
      data'
  source_content_hash: sha256:c8e93ee53ace221df2974792a293caeab268a60eea0e0ed3d5978ec23117644e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/FangYuan717/ROASMI
  tools:
  - PyTorch
  - ROASMI
  - chemprop
  edam_operation: http://edamontology.org/operation_0220
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# spearman-correlation-computation

## Summary

Computes Spearman rank correlation between predicted retention orders and experimental elution orders across RPLC datasets, serving as the core rank-agreement metric for evaluating ROASMI's retention-order predictions against Morgan fingerprint baselines.

## When to use

Use when evaluating a retention-order prediction model (e.g. ROASMI, Morgan_count fingerprints) against experimental elution orders on multiple RPLC datasets, where per-dataset rank agreement must be aggregated into an average rank score.

## When NOT to use

- Input is already a feature table or embeddings without paired experimental elution-order labels — Spearman computation cannot verify rank agreement without ground-truth order.
- Evaluating on HILIC data — ROASMI's retention models are RPLC-specific (eluent pH ≈ 2.7) and HILIC generalizability is explicitly future work.
- Comparing a single compound's retention value against a threshold (e.g. RT outlier detection) rather than pairwise ranking across compounds in a dataset.

## Inputs

- 71 independent RPLC datasets with experimental retention orders (from the ROASMI package data directory)
- Unretrained ROASMI ensemble checkpoint (ROASMI_1–ROASMI_5) loaded without retraining
- Morgan_count fingerprint predictions for the same compound pairs (baseline)

## Outputs

- table2.csv with per-dataset accuracy, Spearman correlation, and rank score for ROASMI vs Morgan_count
- fig5b.png through fig5e.png showing rank-score distributions and average rank score comparison (0.696 vs 0.795)

## How to apply

For each independent RPLC dataset (the ROASMI generalizability set spans 71 datasets), predict pairwise retention orders using the chemprop D-MPNN embedding plus RankNet module of the unretrained ROASMI ensemble, or compute the Morgan_count fingerprint baseline. Then compute Spearman's rank correlation between predicted retention scores and experimental elution order for every compound in each dataset, alongside accuracy and the average rank score; assemble per-dataset metrics into a table (Table 2) and generate rank-score distribution panels (Fig. 5b–e). Validate by checking that the 71-dataset average rank score matches the reported 0.696 for ROASMI (within Monte-Carlo seed tolerance) against the 0.795 of the initial reference set, demonstrating that the un-retrained model retains generalizability.

## Related tools

- **ROASMI** (Source of the unretrained ensemble used to generate predicted retention orders for Spearman comparison against experimental elution orders on each of the 71 datasets) — https://github.com/FangYuan717/ROASMI
- **chemprop** (Provides the D-MPNN molecular embedding module whose output feeds the RankNet retention prediction module used to produce the predicted ranks being correlated) — https://github.com/chemprop/chemprop
- **PyTorch** (Backend framework for running the chemprop-based embeddings and RankNet module used in the retention-order predictions)

## Examples

```
python code/ROASMI_predict.py  # then compute Spearman vs experimental order per dataset
```

## Evaluation signals

- Average rank score across the 71 independent RPLC datasets equals ~0.696 for ROASMI (within Monte-Carlo seed tolerance) and ~0.795 for the initial reference set
- Per-dataset Spearman correlation values assembled in table2.csv reproduce Table 2 row structure (accuracy, Spearman, rank score per dataset)
- Rank-score distribution panels fig5b–fig5e show ROASMI's distribution centered near 0.696 and clearly above the Morgan_count baseline distribution
- No retraining occurred between loading the ensemble checkpoint and scoring: evaluation is on the initial, unretrained ROASMI models only

## Limitations

- ROASMI applies only to RPLC systems with eluent pH around 2.7; HILIC retention model construction with generalizability remains future work
- Very close chromatographic peaks may have inconsistent retention order across experiments, so Spearman-based rank agreement is inherently uncertain for near-co-eluting pairs
- Model cannot distinguish certain molecular pairs with close retention (e.g. 4-hydroxybenzoic acid vs 2,5-dihydroxybenzoic acid), limiting Spearman resolution for indistinguishable pairs

## Evidence

- [abstract] 71 independent RPLC datasets used for generalizability evaluation: "The generalizability of ROASMI is proven by 71 independent reversed-phase liquid chromatography (RPLC) datasets."
- [readme] The molecular embedding module is an extension of chemprop using D-MPNN: "A directed message transfer neural network (D-MPNN) is used to learn directly from the structure of compounds, allowing prediction of compounds in new chemical spaces."
- [readme] RankNet used for elution-order learning: "A Ranking Neural Network (RankNet) was used"
- [readme] Four initial ROASMI models for RPLC at pH ~2.7: "We provide four initial ROASMI models (ROASMI_1 - ROASMI_5) for predicting the retention behavior of compounds in the reversed-phase liquid chromatography (RPLC) system with an eluent pH of around"
- [readme] Ensemble variance quantifies uncertainty: "The ensemble approach allowed quantifying model uncertainty using the variance of the retention order predictions across the trained models."
- [discussion] Indistinguishable pairs should be excluded from downstream identification: "Such pairs should be excluded from the subsequent small molecule identification process."
