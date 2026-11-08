---
name: rplc-dataset-benchmark-evaluation
description: Use when you need to quantify how well an unretrained retention-prediction
  model generalizes to new reversed-phase liquid chromatography (RPLC) datasets versus
  a fingerprint baseline, e.g. computing average rank scores, accuracy, and Spearman
  correlation.
when_to_use_negative:
- Input chromatography is HILIC rather than RPLC — ROASMI only applies to RPLC systems
  with an eluent pH around 2.7; HILIC generalizability is explicitly left as future
  work.
- The model has already been retrained on a user-defined reference set — this skill
  specifically benchmarks the initial, unretrained ensemble for out-of-the-box generalizability.
- Target compounds fall outside the elemental restrictions of C, H, N, O, P, and S
  — the model's applicability domain does not cover the broader chemical space.
tools:
- name: ROASMI
  role: Supplies the unretrained ensemble checkpoint (D-MPNN embedding + RankNet)
    used to predict pairwise retention orders across the 71 datasets, plus the accompanying
    data for reproducing the benchmark.
  repo: https://github.com/FangYuan717/ROASMI
- name: chemprop
  role: Underlying D-MPNN molecular embedding module used to map candidate compounds
    directly into generic chemical spaces before retention-score prediction.
  repo: https://github.com/chemprop/chemprop
- name: PyTorch
  role: Deep-learning framework running the D-MPNN and RankNet modules during batch
    inference on the 71 datasets.
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1186/s13321-025-00968-8
    title: 'ROASMI: accelerating small molecule identification by repurposing retention
      data'
  source_content_hash: sha256:fe6bbb038a889294c26518b90b6d1afb3519832051e02c4424795b8a24e229b3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/FangYuan717/ROASMI
  tools:
  - PyTorch
  - ROASMI
  - chemprop
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# rplc-dataset-benchmark-evaluation

## Summary

Evaluates the out-of-the-box generalizability of a retention-order model (ROASMI) against an Morgan_count fingerprint baseline across 71 independent RPLC datasets, reporting rank scores (e.g. 0.696 vs the initial-set 0.795) without any retraining.

## When to use

Use when you need to quantify how well an unretrained retention-prediction model generalizes to new reversed-phase liquid chromatography (RPLC) datasets versus a fingerprint baseline, e.g. computing average rank scores, accuracy, and Spearman correlation.

## When NOT to use

- Input chromatography is HILIC rather than RPLC — ROASMI only applies to RPLC systems with an eluent pH around 2.7; HILIC generalizability is explicitly left as future work.
- The model has already been retrained on a user-defined reference set — this skill specifically benchmarks the initial, unretrained ensemble for out-of-the-box generalizability.
- Target compounds fall outside the elemental restrictions of C, H, N, O, P, and S — the model's applicability domain does not cover the broader chemical space.

## Inputs

- 71 independent RPLC datasets (experimental elution orders / retention data)
- Initial (unretrained) ROASMI ensemble model checkpoint from the ROASMI package data directory
- Candidate compound structures (SMILES) for each dataset
- Morgan_count fingerprint baseline model

## Outputs

- Per-dataset accuracy, Spearman correlation, and rank score for ROASMI and Morgan_count
- table2.csv (reproducing the article's Table 2 per-dataset metrics)
- Fig. 5b-e panels: rank-score distributions across the 71 datasets and average rank-score comparison (ROASMI 0.696 vs initial-set 0.795)

## How to apply

Load the 71 independent RPLC datasets plus the initial (unretrained) ROASMI ensemble checkpoint from the ROASMI package data directory. For each dataset, use the chemprop-based D-MPNN molecular embedding module and the RankNet retention prediction module to predict pairwise retention orders for all compound pairs without retraining, and run the identical evaluation on the Morgan_count fingerprint baseline. For each dataset compute accuracy, Spearman correlation, and rank score against the experimentally observed elution order; the initial ROASMI model achieves an average rank score of 0.696 on the 71 independent sets versus 0.795 on the initial reference set. Assemble the per-dataset metrics into a table (reproducing the article's Table 2, e.g. written to table2.csv) and generate the Fig. 5b-e panels showing rank-score distributions and the average rank-score comparison between ROASMI and Morgan_count. Treat Monte-Carlo seed tolerance as acceptable when verifying that the computed average rank score matches the reported value.

## Related tools

- **ROASMI** (Supplies the unretrained ensemble checkpoint (D-MPNN embedding + RankNet) used to predict pairwise retention orders across the 71 datasets, plus the accompanying data for reproducing the benchmark.) — https://github.com/FangYuan717/ROASMI
- **chemprop** (Underlying D-MPNN molecular embedding module used to map candidate compounds directly into generic chemical spaces before retention-score prediction.) — https://github.com/chemprop/chemprop
- **PyTorch** (Deep-learning framework running the D-MPNN and RankNet modules during batch inference on the 71 datasets.)

## Examples

```
python code/ROASMI_predict.py  # predicts retention scores on the ready-to-identify datasets (default pred_path is 'data/predict_toy_set.csv')
```

## Evaluation signals

- Average rank score over the 71 independent RPLC datasets reproduces the reported 0.696 within Monte-Carlo seed tolerance.
- The initial-set reference rank score reproduces the reported 0.795.
- No retraining is performed — predictions come solely from the pre-packaged ensemble checkpoint.
- Per-dataset accuracy and Spearman correlation are computed for both ROASMI and the Morgan_count fingerprint baseline, allowing a side-by-side comparison.
- Output artifacts (table2.csv, fig5b.png through fig5e.png) reproduce the article's Table 2 and Fig. 5b-e.

## Limitations

- ROASMI currently applies only to RPLC systems, limiting application to a specific chromatographic space; HILIC generalizability is future work.
- The model is unable to distinguish the relative order of certain molecular pairs with close retention (e.g. 4-hydroxybenzoic acid and 2,5-dihydroxybenzoic acid); such indistinguishable pairs should be excluded from subsequent identification.
- Applies only to conventional compounds within elemental restrictions of C, H, N, O, P, and S; broader chemical space remains unexplored.
- When two compounds' chromatographic peaks are close enough, their retention order may be inconsistent across experiments, and RT outliers from slight experimental condition changes are unresolved.

## Evidence

- [abstract] 71-dataset generalizability benchmark: "The generalizability of ROASMI is proven by 71 independent reversed-phase liquid chromatography (RPLC) datasets."
- [methods] RankNet retention prediction module: "The retention prediction module. A Ranking Neural Network (RankNet) was used to learn elution orders"
- [readme] unretrained ROASMI checkpoint used for out-of-box inference: "We provide four initial ROASMI models (ROASMI_1 - ROASMI_5) for predicting the retention behavior of compounds in the reversed-phase liquid chromatography (RPLC) system"
- [readme] chemprop D-MPNN embedding module: "The molecular embedding module. This module is an extension of chemprop"
