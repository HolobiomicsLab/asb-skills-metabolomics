---
name: ranknet-pairwise-loss-implementation
description: Use when training a retention-order model from pairwise elution comparisons
  of molecular pairs rather than absolute RT regression, e.g. when chromatographic
  RTs are non-replicable across methods but relative elution orders are consistent.
when_to_use_negative:
- The chromatographic system is HILIC rather than RPLC — ROASMI currently applies
  only to RPLC systems and HILIC generalizable models are future work.
- Compounds lie outside the elemental restrictions of C, H, N, O, P, and S — model
  applicability is limited to conventional compounds within these elements.
- Absolute retention times are the target; RankNet learns pairwise elution order,
  not RT values.
edam_operation: http://edamontology.org/operation_3936
edam_topics:
- http://edamontology.org/topic_3336
- http://edamontology.org/topic_0154
tools:
- name: ROASMI
  role: Host framework whose retention prediction module is the RankNet trained with
    the pairwise binary cross-entropy loss
  repo: https://github.com/FangYuan717/ROASMI
- name: chemprop
  role: Source of the D-MPNN molecular embeddings that feed the RankNet pairwise score
    comparisons
  repo: https://github.com/chemprop/chemprop
- name: PyTorch
  role: Framework in which the RankNet training loop, binary cross-entropy loss, and
    Adam optimizer are implemented
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1186/s13321-025-00968-8
    title: 'ROASMI: accelerating small molecule identification by repurposing retention
      data'
  source_content_hash: sha256:535344cbbcd8a03f503e060d79fd76736e79ded54b998a92fa612477179eaca6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/FangYuan717/ROASMI
  tools:
  - PyTorch
  - ROASMI
  - chemprop
  edam_operation: http://edamontology.org/operation_3936
  edam_topics:
  - http://edamontology.org/topic_3336
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# ranknet-pairwise-loss-implementation

## Summary

Implementation of the RankNet (Ranking Neural Network) pairwise loss used in ROASMI to learn elution order: the loss is binary cross-entropy between the sigmoid of predicted retention scores — the pairwise probability P_ij that compound Ci elutes before Cj — and the observed pairwise elution-order label.

## When to use

Use when training a retention-order model from pairwise elution comparisons of molecular pairs rather than absolute RT regression, e.g. when chromatographic RTs are non-replicable across methods but relative elution orders are consistent.

## When NOT to use

- The chromatographic system is HILIC rather than RPLC — ROASMI currently applies only to RPLC systems and HILIC generalizable models are future work.
- Compounds lie outside the elemental restrictions of C, H, N, O, P, and S — model applicability is limited to conventional compounds within these elements.
- Absolute retention times are the target; RankNet learns pairwise elution order, not RT values.

## Inputs

- Reference retention dataset with pairwise elution-order labels (e.g. dataset_87 split 8:1:1)
- SMILES strings for each compound, embedded via the chemprop D-MPNN module
- RankNet hyperparameters from Table 1 of the ROASMI paper

## Outputs

- Trained RankNet models ROASMI_1–ROASMI_5 (5-seed ensemble)
- For each molecular pair, P_ij — the probability compound Ci elutes before Cj
- Cross-seed mean and variance of predicted pairwise probabilities

## How to apply

Represent each compound with a chemprop D-MPNN molecular embedding (the molecular embedding module that learns directly from structure rather than fingerprints). For each training pair (Ci, Cj), form the input as the embedding difference/score difference s_ij and compute P_ij = sigmoid(s_ij); train with a binary cross-entropy loss comparing P_ij against the observed label for which compound elutes first. Split the reference dataset (dataset_87) into train/validation/test at 8:1:1, train in PyTorch with the Adam optimizer using the hyperparameters of Table 1, apply early stopping on validation pairwise loss, and repeat under 5 random seeds to build the ensemble ROASMI_1–ROASMI_5. Ensemble variance across seeds quantifies pairwise uncertainty — low variance for distinguishable pairs, high variance for indistinguishable pairs (e.g. pairs with ~50% P_ij).

## Related tools

- **ROASMI** (Host framework whose retention prediction module is the RankNet trained with the pairwise binary cross-entropy loss) — https://github.com/FangYuan717/ROASMI
- **chemprop** (Source of the D-MPNN molecular embeddings that feed the RankNet pairwise score comparisons) — https://github.com/chemprop/chemprop
- **PyTorch** (Framework in which the RankNet training loop, binary cross-entropy loss, and Adam optimizer are implemented)

## Examples

```
python code/ROASMI_train.py
```

## Evaluation signals

- Held-out pairwise predictions are probabilities P_ij in [0,1] that rank elution order for molecular pairs
- Distinguishable pairs show low cross-seed prediction variance while indistinguishable pairs (e.g. 4-hydroxybenzoic acid vs 2,5-dihydroxybenzoic acid) show high variance
- Validation pairwise loss decreases and early stopping halts training before overfitting on the 8:1:1 split of dataset_87
- Five ensemble members (ROASMI_1–ROASMI_5) produce predictions whose variance is consistent with reported pairwise uncertainty behavior

## Limitations

- ROASMI is limited to RPLC systems with eluent pH around 2.7; HILIC model construction with generalizability is future work.
- Compounds must fall within the C, H, N, O, P, S elemental space; broader chemical space is unexplored.
- Pairs with close enough chromatographic peaks may have inconsistent retention order across experiments, making them indistinguishable.
- RT outliers from slight experimental condition changes remain unresolved and require mechanistic study.

## Evidence

- [other] RankNet BCE loss on sigmoid of pairwise score: [evidence span withheld — anchor status: not_found]
- [readme] RankNet learns elution orders: "The retention prediction module. A Ranking Neural Network (RankNet) was used to learn elution orders"
- [readme] ensemble variance quantifies pairwise uncertainty: "The ensemble approach allowed quantifying model uncertainty using the variance of the retention order predictions across the trained models."
- [discussion] indistinguishable pairs cannot be ordered: "the model is unable to distinguish the relative order of the two peaks (i.e., indistinguishable)"
