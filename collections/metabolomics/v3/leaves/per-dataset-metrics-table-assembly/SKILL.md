---
name: per-dataset-metrics-table-assembly
description: Use when evaluating a pairwise retention-order model over many independent
  RPLC datasets and you must compute per-dataset accuracy, Spearman, and rank score
  for model vs. fingerprint baseline, then assemble into a comparison table.
when_to_use_negative:
- When the goal is to retrain or fine-tune the ROASMI models on a new reference set
  — use the article's ROASMI_train.py workflow instead; this skill evaluates generalizability
  of the unretrained ensemble only.
- When evaluating HILIC retention models — the article states ROASMI currently applies
  only to RPLC systems, so HILIC datasets (e.g. dataset_83/dataset_84) require separate
  handling.
- When the target is a single dataset with no fingerprint baseline comparison — the
  per-dataset-vs-baseline table assembly is only meaningful with both models evaluated
  on the same dataset set.
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3339
- http://edamontology.org/topic_0091
tools:
- name: ROASMI
  role: Provides the initial (unretrained) ensemble checkpoints and the RankNet retention-order
    prediction pipeline evaluated across the 71 RPLC datasets
  repo: https://github.com/FangYuan717/ROASMI
- name: chemprop
  role: D-MPNN molecular embedding module used inside ROASMI to produce embeddings
    for each candidate compound before rank scoring
  repo: https://github.com/chemprop/chemprop
- name: PyTorch
  role: Underlying deep-learning framework running the chemprop embedding and RankNet
    scoring during evaluation
- name: descriptastorus
  role: Source of the Morgan_count fingerprint baseline (Morgan3Counts-style descriptors)
    against which ROASMI's rank scores are compared
  repo: https://github.com/bp-kelley/descriptastorus
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1186/s13321-025-00968-8
    title: 'ROASMI: accelerating small molecule identification by repurposing retention
      data'
  source_content_hash: sha256:c818561b919d3b67756dea5a22345a66633158496299295c2a381c0e595f113f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/FangYuan717/ROASMI
  tools:
  - PyTorch
  - ROASMI
  - chemprop
  - descriptastorus
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3339
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 15
---

# per-dataset-metrics-table-assembly

## Summary

Assemble a per-dataset evaluation table (e.g. Table 2 of ROASMI) reporting accuracy, Spearman correlation, and rank score for the ROASMI RankNet ensemble vs. a Morgan_count fingerprint baseline across 71 independent RPLC datasets.

## When to use

Use when evaluating a pairwise retention-order model over many independent RPLC datasets and you must compute per-dataset accuracy, Spearman, and rank score for model vs. fingerprint baseline, then assemble into a comparison table.

## When NOT to use

- When the goal is to retrain or fine-tune the ROASMI models on a new reference set — use the article's ROASMI_train.py workflow instead; this skill evaluates generalizability of the unretrained ensemble only.
- When evaluating HILIC retention models — the article states ROASMI currently applies only to RPLC systems, so HILIC datasets (e.g. dataset_83/dataset_84) require separate handling.
- When the target is a single dataset with no fingerprint baseline comparison — the per-dataset-vs-baseline table assembly is only meaningful with both models evaluated on the same dataset set.

## Inputs

- 71 independent RPLC datasets with compound identities and experimental elution/retention orders
- Initial (unretrained) ROASMI ensemble checkpoints ROASMI_1–ROASMI_5 from the ROASMI package data directory
- Morgan_count fingerprint baseline predictions or a generator for them (e.g. via descriptastorus)
- Per-dataset candidate lists of SMILES for pairwise retention-order prediction

## Outputs

- table2.csv — per-dataset accuracy, Spearman correlation, and rank score for ROASMI vs. Morgan_count
- fig5b.png–fig5e.png — rank-score distributions across the 71 datasets and the average rank score comparison (ROASMI 0.696 vs. initial-set 0.795)
- Aggregated average rank score per model over the 71 datasets

## How to apply

For each of the 71 independent RPLC datasets, run the unretrained ROASMI ensemble (chemprop D-MPNN embeddings feeding the RankNet retention prediction module) to predict pairwise elution orders for all compound pairs, without any retraining. Compute per-dataset accuracy, Spearman correlation, and rank score against the experimental elution order, then repeat the identical evaluation for the Morgan_count fingerprint baseline so both models share the same metric definitions and dataset splits. Aggregate the per-dataset metrics into a single table that reproduces the article's Table 2 (write to table2.csv), and compute the average rank score across the 71 datasets for the headline generalizability figure. Validate that the aggregated average rank score matches the reported 0.696 for the initial ROASMI model (within Monte-Carlo seed tolerance) and that the initial reference set's rank score is 0.795; also produce the Fig. 5b-e panels comparing the rank-score distributions and averages of ROASMI vs. Morgan_count. Because ROASMI's predictions carry a variance across the ensemble (ROASMI_1–ROASMI_5), propagate that uncertainty when reporting which pairs are distinguishable rather than treating a single point estimate as definitive.

## Related tools

- **ROASMI** (Provides the initial (unretrained) ensemble checkpoints and the RankNet retention-order prediction pipeline evaluated across the 71 RPLC datasets) — https://github.com/FangYuan717/ROASMI
- **chemprop** (D-MPNN molecular embedding module used inside ROASMI to produce embeddings for each candidate compound before rank scoring) — https://github.com/chemprop/chemprop
- **PyTorch** (Underlying deep-learning framework running the chemprop embedding and RankNet scoring during evaluation)
- **descriptastorus** (Source of the Morgan_count fingerprint baseline (Morgan3Counts-style descriptors) against which ROASMI's rank scores are compared) — https://github.com/bp-kelley/descriptastorus

## Examples

```
python code/ROASMI_predict.py --pred_path data/predict_toy_set.csv; python code/Identify.py  # then compute per-dataset accuracy/Spearman/rank score vs. Morgan_count baseline and assemble table2.csv
```

## Evaluation signals

- Average rank score across the 71 datasets equals 0.696 for the unretrained ROASMI ensemble (within Monte-Carlo seed tolerance) and the initial reference set value is 0.795
- table2.csv contains one row per independent RPLC dataset with accuracy, Spearman correlation, and rank score for both ROASMI and Morgan_count
- Both ROASMI and Morgan_count are evaluated on the identical 71 dataset splits so per-dataset metrics are directly comparable
- Fig. 5b–e panels are generated from the same per-dataset metrics used in table2.csv, showing consistent rank-score distributions and averages
- No ROASMI retraining is performed during evaluation — the ensemble checkpoints loaded must be the initial, unretrained ones

## Limitations

- ROASMI is limited to conventional compounds within the elemental restrictions of C, H, N, O, P, and S; broader chemical space is unexplored, so per-dataset metrics may degrade on unusual elements.
- ROASMI currently applies only to RPLC systems; per-dataset evaluation on HILIC data is out of scope.
- The model cannot distinguish molecular pairs with very close retention (e.g., 4-hydroxybenzoic acid vs. 2,5-dihydroxybenzoic acid), so accuracy metrics on such pairs may be indistinguishable from chance.
- Rank-score agreement may vary with Monte-Carlo seed; the reported 0.696 average must be checked within seed tolerance rather than exactly.

## Evidence

- [abstract] generalizability of ROASMI is proven by 71 independent RPLC datasets: "The generalizability of ROASMI is proven by 71 independent reversed-phase liquid chromatography (RPLC) datasets."
- [other] average rank score of 0.696 across the 71 datasets vs 0.795 on the initial reference set: [evidence span withheld — anchor status: not_found]
- [readme] D-MPNN embedding module extended from chemprop: "A directed message transfer neural network (D-MPNN) is used to learn directly from the structure of compounds"
- [readme] RankNet used for retention prediction: "A Ranking Neural Network (RankNet) was used to learn elution orders"
- [readme] ensemble approach quantifies model uncertainty: "The ensemble approach allowed quantifying model uncertainty using the variance of the retention order predictions across the trained models."
- [discussion] ROASMI limited to RPLC and conventional elements: "applies to conventional compounds within the elemental restrictions of C, H, N, O, P, and S, leaving a vast chemical space yet to be explored"
- [discussion] indistinguishable pairs with close retention: "when the chromatographic peaks of two compounds are close enough together, their order of retention may not be consistent across experiments"
