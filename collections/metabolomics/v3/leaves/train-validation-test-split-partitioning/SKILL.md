---
name: train-validation-test-split-partitioning
description: Use when training the ROASMI RankNet retention-order model on a reference
  RPLC dataset and you need an 8:1:1 train/validation/test partition to support early
  stopping and held-out pairwise-probability evaluation.
when_to_use_negative:
- The task is HILIC retention modeling — the article states constructing generalized
  HILIC retention models is future work, so RPLC-trained 8:1:1 partitions are not
  transferable
- Molecules fall outside the elemental restrictions of C, H, N, O, P, and S that ROASMI's
  application domain covers
- Prediction is wanted for pairs whose chromatographic peaks are close enough that
  their elution order is not consistent across experiments
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3337
- http://edamontology.org/topic_0154
tools:
- name: ROASMI
  role: Reference retention-order model whose RankNet training consumes the 8:1:1
    partitions
  repo: https://github.com/FangYuan717/ROASMI
- name: chemprop
  role: D-MPNN molecular-embedding module whose extension supplies features for RankNet
    training on the training partition
  repo: https://github.com/chemprop/chemprop
- name: PyTorch
  role: Framework in which the RankNet training loop with Adam optimizer and early
    stopping is implemented
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1186/s13321-025-00968-8
    title: 'ROASMI: accelerating small molecule identification by repurposing retention
      data'
  source_content_hash: sha256:6dfcdcf1e34650a7d237a1b8f222abace2151f91a6fce176cadc1b68e2e27462
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/FangYuan717/ROASMI
  tools:
  - PyTorch
  - ROASMI
  - chemprop
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3337
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# Train-validation-test split partitioning

## Summary

Partitions a reference retention dataset (e.g. dataset_87) into train/validation/test at an 8:1:1 ratio before training the ROASMI RankNet retention-order model, enabling early stopping and honest held-out evaluation of pairwise elution probabilities across five random-seed ensemble members.

## When to use

Use when training the ROASMI RankNet retention-order model on a reference RPLC dataset and you need an 8:1:1 train/validation/test partition to support early stopping and held-out pairwise-probability evaluation.

## When NOT to use

- The task is HILIC retention modeling — the article states constructing generalized HILIC retention models is future work, so RPLC-trained 8:1:1 partitions are not transferable
- Molecules fall outside the elemental restrictions of C, H, N, O, P, and S that ROASMI's application domain covers
- Prediction is wanted for pairs whose chromatographic peaks are close enough that their elution order is not consistent across experiments

## Inputs

- Reference retention dataset of compound pairs with observed elution-order labels (dataset_87)
- Table 1 reported RankNet hyperparameters
- Random seed list for the 5-member ensemble (ROASMI_1–ROASMI_5)

## Outputs

- 8:1:1 train/validation/test partitions of the reference pair dataset
- Trained ROASMI_1–ROASMI_5 RankNet ensemble member checkpoints
- Pairwise elution probabilities P_ij and cross-seed prediction variance for held-out pairs

## How to apply

Load the reference retention dataset (dataset_87 as the RankNet reference set) as compound-pair records with observed elution-order labels and split it into train/validation/test partitions at an 8:1:1 ratio, as prescribed in the ROASMI workflow. Train the RankNet — a Ranking Neural Network built on chemprop D-MPNN molecular embeddings — on the training partition only, minimizing the binary cross-entropy loss between the sigmoid of predicted retention scores and the observed pairwise elution-order label. Use the validation partition to monitor pairwise loss for early stopping and to select the stopping epoch, then repeat training under 5 random seeds (ROASMI_1–ROASMI_5) to form an ensemble whose cross-seed prediction variance quantifies uncertainty (e.g. 81% probability with 9% variance for a distinguishable pair). Reserve the test partition for final evaluation of held-out pairs, checking that distinguishable pairs show low variance and indistinguishable pairs show high variance.

## Related tools

- **ROASMI** (Reference retention-order model whose RankNet training consumes the 8:1:1 partitions) — https://github.com/FangYuan717/ROASMI
- **chemprop** (D-MPNN molecular-embedding module whose extension supplies features for RankNet training on the training partition) — https://github.com/chemprop/chemprop
- **PyTorch** (Framework in which the RankNet training loop with Adam optimizer and early stopping is implemented)

## Examples

```
python code/ROASMI_train.py  # retrain ROASMI on the 8:1:1 partitioned reference set under 5 seeds
```

## Evaluation signals

- Split proportions equal 8:1:1 with no pair leaking across partitions
- Validation pairwise loss decreases and triggers early stopping before overfitting
- Held-out pairs yield P_ij probabilities; distinguishable pairs show low cross-seed variance and indistinguishable pairs high variance
- Five ensemble members (ROASMI_1–ROASMI_5) train independently under distinct seeds and reproduce the reported variance behavior

## Limitations

- RankNet cannot distinguish molecular pairs with very close retention (e.g. 4-hydroxybenzoic acid vs 2,5-dihydroxybenzoic acid), limiting interpretation of test-partition probabilities
- Trained models apply only to RPLC systems at eluent pH around 2.7; retraining with user-defined reference sets is required for other spaces
- RT outliers caused by slight experimental condition changes remain unresolved and can corrupt partition labels

## Evidence

- [methods] 8:1:1 split of dataset_87 for RankNet training: [evidence span withheld — anchor status: not_found]
- [methods] RankNet learns pairwise elution order from consistent retention sequences: [evidence span withheld — anchor status: not_found]
- [readme] Ensemble uncertainty quantified by prediction variance across seeds: "The ensemble approach allowed quantifying model uncertainty using the variance of the retention order predictions across the trained models."
- [discussion] Indistinguishable-pair example with 81% probability and 9% variance: "ROASMI predicts that peak 1 elutes before peak 2 with an 81% probability and a variance of 9%."
