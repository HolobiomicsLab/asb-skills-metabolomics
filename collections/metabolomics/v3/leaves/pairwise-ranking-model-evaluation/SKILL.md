---
name: pairwise-ranking-model-evaluation
description: Use when you have an unretrained pairwise ranking model (e.g., ROASMI
  ensemble) and one or more independent datasets with experimental elution orders,
  and need to quantify cross-dataset generalizability via rank score, accuracy, and
  Spearman correlation.
when_to_use_negative:
- The evaluation requires retraining the model — this skill evaluates only the initial,
  unretrained ensemble for generalizability; use a separate retraining workflow instead.
- The datasets are HILIC rather than RPLC — the article states ROASMI currently applies
  only to RPLC systems.
- The compounds fall outside the elemental restrictions of C, H, N, O, P, and S or
  are outside the application domain of the ROASMI model.
- The comparison is intended as a mechanistic study of retention outliers rather than
  a cross-dataset generalizability benchmark.
edam_operation: http://edamontology.org/operation_3803
edam_topics:
- http://edamontology.org/topic_3399
tools:
- name: ROASMI
  role: Provides the initial unretrained ensemble checkpoint and the RankNet-based
    pairwise retention-order predictor being evaluated
  repo: https://github.com/FangYuan717/ROASMI
- name: chemprop
  role: Provides the D-MPNN molecular embedding module used to generate representations
    for compounds before ranking
  repo: https://github.com/chemprop/chemprop
- name: PyTorch
  role: Deep-learning backend used by the ROASMI ensemble for inference on compound
    pairs
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1186/s13321-025-00968-8
    title: 'ROASMI: accelerating small molecule identification by repurposing retention
      data'
  source_content_hash: sha256:da860bd530f7c6685e10da85a1d742e3c8edfb2622e4fd7945552417db210b2e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/FangYuan717/ROASMI
  tools:
  - PyTorch
  - ROASMI
  - chemprop
  edam_operation: http://edamontology.org/operation_3803
  edam_topics:
  - http://edamontology.org/topic_3399
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# Pairwise Ranking Model Evaluation

## Summary

Evaluates a pairwise ranking model (e.g., a RankNet retention-order predictor) by computing per-dataset accuracy, Spearman correlation, and rank score against experimental elution orders, without retraining, to quantify generalizability.

## When to use

Use when you have an unretrained pairwise ranking model (e.g., ROASMI ensemble) and one or more independent datasets with experimental elution orders, and need to quantify cross-dataset generalizability via rank score, accuracy, and Spearman correlation.

## When NOT to use

- The evaluation requires retraining the model — this skill evaluates only the initial, unretrained ensemble for generalizability; use a separate retraining workflow instead.
- The datasets are HILIC rather than RPLC — the article states ROASMI currently applies only to RPLC systems.
- The compounds fall outside the elemental restrictions of C, H, N, O, P, and S or are outside the application domain of the ROASMI model.
- The comparison is intended as a mechanistic study of retention outliers rather than a cross-dataset generalizability benchmark.

## Inputs

- 71 independent RPLC datasets with experimental elution orders (compound pairs and observed order)
- Initial, unretrained ROASMI ensemble checkpoint (ROASMI_1–ROASMI_5) from the ROASMI package data directory
- Morgan_count fingerprint baseline predictions for the same compound pairs
- Initial reference set used to establish the baseline rank score (0.795)

## Outputs

- Per-dataset accuracy, Spearman correlation, and rank score for ROASMI and the Morgan_count baseline
- table2.csv — assembled per-dataset metrics reproducing Table 2
- fig5b.png through fig5e.png — rank-score distributions across 71 datasets for ROASMI vs Morgan_count and average rank-score comparison

## How to apply

Load the 71 independent RPLC datasets and the initial (unretrained) ROASMI ensemble checkpoint from the ROASMI package data directory. For each dataset, use the chemprop-based D-MPNN molecular embeddings plus the RankNet retention prediction module to predict pairwise retention orders for all compound pairs, performing no retraining. Compute per-dataset accuracy, Spearman correlation, and rank score against experimental elution orders, and repeat the identical evaluation for the Morgan_count fingerprint baseline. Assemble per-dataset metrics into a table (reproducing Table 2) and write to table2.csv; generate the Fig. 5b-e panels showing rank-score distributions across the 71 datasets for ROASMI vs Morgan_count and the average rank-score comparison (0.696 across 71 datasets vs 0.795 on the initial reference set). The ensemble of four initial ROASMI models (ROASMI_1–ROASMI_5) allows quantifying uncertainty via the variance of retention-order predictions; the ensemble average rank score over 71 independent datasets is 0.696 versus 0.795 for the initial reference set.

## Related tools

- **ROASMI** (Provides the initial unretrained ensemble checkpoint and the RankNet-based pairwise retention-order predictor being evaluated) — https://github.com/FangYuan717/ROASMI
- **chemprop** (Provides the D-MPNN molecular embedding module used to generate representations for compounds before ranking) — https://github.com/chemprop/chemprop
- **PyTorch** (Deep-learning backend used by the ROASMI ensemble for inference on compound pairs)

## Examples

```
python code/ROASMI_predict.py  # evaluate unretrained ROASMI ensemble against experimental elution orders; compare with Morgan_count baseline
```

## Evaluation signals

- The computed average rank score over the 71 independent RPLC datasets matches the reported 0.696 (within Monte-Carlo seed tolerance).
- The reference rank score on the initial set is 0.795.
- The produced table2.csv contains the same per-dataset accuracy, Spearman correlation, and rank-score columns as Table 2 for both ROASMI and the Morgan_count baseline.
- The generated fig5b.png–fig5e.png show rank-score distributions and the average-rank-score comparison matching the article's reported values.

## Limitations

- ROASMI applies only to RPLC systems (eluent pH ~2.7); HILIC retention model generalizability is future work.
- ROASMI covers conventional compounds within elemental restrictions of C, H, N, O, P, and S, leaving broader chemical space unexplored.
- Molecular pairs with very close retention (e.g., 4-hydroxybenzoic acid vs 2,5-dihydroxybenzoic acid) may be indistinguishable; the model can flag them via ensemble variance (81% probability, 9% variance in the example) but the order may not be consistent across experiments.
- Results depend on Monte-Carlo seeds, so exact numeric values may fluctuate slightly between runs.

## Evidence

- [abstract] The generalizability of ROASMI is proven by 71 independent reversed-phase liquid chromatography (RPLC) datasets.: "The generalizability of ROASMI is proven by 71 independent reversed-phase liquid chromatography (RPLC) datasets."
- [methods] This module is an extension of chemprop described in the paper: "This module is an extension of chemprop described in the paper"
- [methods] A Ranking Neural Network (RankNet) was used to learn elution orders: "A Ranking Neural Network (RankNet) was used to learn elution orders"
- [results] the initial ROASMI model displayed an average rank score of 0.696 across the 71 independent RPLC datasets, whereas the initial reference set's rank score was 0.795: [evidence span withheld — anchor status: not_found]
- [readme] we provide four initial ROASMI models (ROASMI_1 - ROASMI_5): "We provide four initial ROASMI models (ROASMI_1 - ROASMI_5)"
- [readme] The ensemble approach allowed quantifying model uncertainty using the variance of the retention order predictions across the trained models.: "The ensemble approach allowed quantifying model uncertainty using the variance of the retention order predictions across the trained models."
- [discussion] ROASMI currently applies only to RPLC systems, which limits its application to a specific chromatographic space.: "ROASMI currently applies only to RPLC systems, which limits its application to a specific chromatographic space."
