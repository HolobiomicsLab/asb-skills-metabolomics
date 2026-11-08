---
name: retention-order-model-training
description: Use when you need to train or fine-tune a retention-order prediction
  model from RT data of a reference compound set (e.g. dataset_87), producing pairwise
  P_ij elution probabilities for candidate small molecules in an RPLC system with
  eluent pH ~2.7.
when_to_use_negative:
- The compounds fall outside conventional elemental restrictions (C, H, N, O, P, S)
  — ROASMI's scope is limited to these elements
- 'You are working in a HILIC chromatographic system: ROASMI''s initial models are
  specific to RPLC at eluent pH ~2.7; HILIC retraining is possible (dataset_83/dataset_84)
  but constructing generalizable HILIC models is explicitly described as future work'
- You need absolute RT predictions rather than retention-order probabilities — the
  model outputs pairwise order probabilities, not RT values
edam_operation: http://edamontology.org/operation_3664
edam_topics:
- http://edamontology.org/topic_0154
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_3474
tools:
- name: ROASMI
  role: 'The trained retention-order model itself: D-MPNN embeddings + RankNet head,
    producing P_ij pairwise elution probabilities for candidate compounds; also trainable
    with user-defined reference sets'
  repo: https://github.com/FangYuan717/ROASMI
- name: chemprop
  role: The molecular embedding module (D-MPNN) that learns compound representations
    directly from SMILES, forming the input features to the RankNet head
  repo: https://github.com/chemprop/chemprop
- name: PyTorch
  role: Deep-learning framework used to implement and train the D-MPNN + RankNet architecture
- name: conda
  role: Environment setup for ROASMI dependencies (conda env create -f environment.yml)
- name: git
  role: Clone the ROASMI repository to obtain code, trained models, and reproducibility
    data
- name: descriptastorus
  role: Optional dependency installed for descriptor generation support in the environment
    (not the primary representation path, which uses D-MPNN embeddings)
  repo: https://github.com/bp-kelley/descriptastorus
provenance:
  source_task_ids:
  - task_003
  - task_006
  source_papers:
  - doi: 10.1186/s13321-025-00968-8
    title: 'ROASMI: accelerating small molecule identification by repurposing retention
      data'
  source_content_hash: sha256:9f347b19c87973a55223084761cb816fd75054f144b8f86d4eb145fbb1b8d61d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/FangYuan717/ROASMI
  tools:
  - PyTorch
  - ROASMI
  - chemprop
  - conda
  - descriptastorus
  - git
  edam_operation: http://edamontology.org/operation_3664
  edam_topics:
  - http://edamontology.org/topic_0154
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# retention-order-model-training

## Summary

Trains a RankNet-based retention-order model (ROASMI) that couples chemprop D-MPNN molecular embeddings with pairwise elution-order probabilities, enabling retention-score prediction across new chemical and chromatographic spaces. The core insight is predicting retention order rather than absolute RT, which sidesteps the poor replicability of RTs across chromatographic methods.

## When to use

Use when you need to train or fine-tune a retention-order prediction model from RT data of a reference compound set (e.g. dataset_87), producing pairwise P_ij elution probabilities for candidate small molecules in an RPLC system with eluent pH ~2.7.

## When NOT to use

- The compounds fall outside conventional elemental restrictions (C, H, N, O, P, S) — ROASMI's scope is limited to these elements
- You are working in a HILIC chromatographic system: ROASMI's initial models are specific to RPLC at eluent pH ~2.7; HILIC retraining is possible (dataset_83/dataset_84) but constructing generalizable HILIC models is explicitly described as future work
- You need absolute RT predictions rather than retention-order probabilities — the model outputs pairwise order probabilities, not RT values

## Inputs

- SMILES file of reference compounds with experimentally observed RTs (e.g. dataset_87, or a user-defined reference set like dataset_83 for HILIC retraining)
- Pairwise compound-comparison labels derived from retention sequences (which compound elutes first) for training
- Candidate-compound SMILES file for scoring (e.g. data/predict_toy_set.csv, or identify_toy_set.csv for integration with MS/MS tools)

## Outputs

- A set of 5 trained ROASMI models (ROASMI_1-ROASMI_5) whose ensemble provides retention-score predictions plus cross-seed variance (e.g. 81% P_ij with 9% variance for a distinguishable pair)
- For each candidate pair: P_ij (probability that compound i elutes before compound j) and variance across ensemble members, used to label pairs as distinguishable vs. indistinguishable
- Retention scores for each compound in a ready-to-identify dataset, suitable for downstream integration with MS/MS scores via `Identify.py`

## How to apply

1) Build the training set from a reference dataset of experimentally observed RTs (e.g. dataset_87) by generating pairwise comparisons of compounds with known elution order from consistent retention sequences. 2) Use chemprop's D-MPNN as the molecular embedding module to map each compound SMILES into a learned representation directly from structure, avoiding hand-crafted fingerprints/descriptors. 3) Feed the two compound embeddings into a RankNet (Ranking Neural Network) head that outputs a pairwise probability P_ij; train with binary cross-entropy comparing the sigmoid of the score difference against the observed label for whether compound i elutes before compound j. 4) Train the ensemble of 5 models (ROASMI_1-ROASMI_5) under different random seeds to enable uncertainty quantification via prediction variance across members. 5) Run training via `python code/ROASMI_train.py` (CLI configurable via args as described in the article methods); predict scores via `python code/ROASMI_predict.py`. 6) Rationale: because RTs are poorly replicable across chromatographic methods (e.g. differences between configurations, polarities, or runs as shown in ST001683, dataset_56, dataset_61), predicting the *order* within a defined application domain is more robust than predicting absolute RT.

## Related tools

- **ROASMI** (The trained retention-order model itself: D-MPNN embeddings + RankNet head, producing P_ij pairwise elution probabilities for candidate compounds; also trainable with user-defined reference sets) — https://github.com/FangYuan717/ROASMI
- **chemprop** (The molecular embedding module (D-MPNN) that learns compound representations directly from SMILES, forming the input features to the RankNet head) — https://github.com/chemprop/chemprop
- **PyTorch** (Deep-learning framework used to implement and train the D-MPNN + RankNet architecture)
- **conda** (Environment setup for ROASMI dependencies (conda env create -f environment.yml))
- **git** (Clone the ROASMI repository to obtain code, trained models, and reproducibility data)
- **descriptastorus** (Optional dependency installed for descriptor generation support in the environment (not the primary representation path, which uses D-MPNN embeddings)) — https://github.com/bp-kelley/descriptastorus

## Examples

```
git clone https://github.com/FangYuan717/ROASMI.git; conda env create -f environment.yml; python code/ROASMI_train.py; python code/ROASMI_predict.py
```

## Evaluation signals

- Held-out pairwise predictions should yield P_ij in [0,1] and cross-seed variance consistent with reported behavior — distinguishable pairs show low variance (e.g. 81% P_ij with 9% variance) while close-eluting pairs show high variance
- The model must distinguish retention order for pairs whose RTs differ enough to be consistent across experiments; pairs whose chromatographic peaks are close enough that their order is not consistent across experiments are expected to be indistinguishable
- Ensemble size must be 5 (ROASMI_1–ROASMI_5); verify that each member is trained under a different random seed
- For retraining/fine-tuning transferability: rank-score improvements should correlate with chemical-space similarity (ChemSS) and chromatographic-space similarity (ChrSS) between reference and prediction sets (e.g. A1 improved by 0.21, A2 by 0.1; A11 and A12 showed negligible gains) — verify your retrained model reproduces this trend on the 12 combination table
- Generalizability check: the initial model was validated on 71 independent RPLC datasets; a retrained model should also be evaluated on independent RPLC test sets rather than only the training distribution

## Limitations

- ROASMI currently applies only to RPLC systems at eluent pH ~2.7; generalizable HILIC models are future work
- Applies to conventional compounds within the elemental restrictions of C, H, N, O, P, and S; broader chemical space is unexplored
- The model is unable to distinguish the relative order of peaks for pairs like 4-hydroxybenzoic acid vs 2,5-dihydroxybenzoic acid where RTs are close
- RT outliers from slight experimental condition changes remain unresolved and require mechanistic study
- Machine learning approaches are imperfect and require integration with mechanistic understanding
- Requires a GPU-enabled PyTorch install and CUDA >= 8.0 / cuDNN for GPU training

## Evidence

- [methods] RankNet binary cross-entropy training on pairwise elution order: "A Ranking Neural Network (RankNet) was used to learn elution orders"
- [methods] chemprop D-MPNN is the molecular embedding module: "**The molecular embedding module.** This module is an extension of chemprop"
- [readme] Ensemble of 5 models enables uncertainty quantification: "We provide four initial ROASMI models (ROASMI_1 - ROASMI_5)"
- [readme] Model specific to RPLC at eluent pH ~2.7: "for predicting the retention behavior of compounds in the reversed-phase liquid chromatography (RPLC) system with an eluent pH of around 2.7"
- [readme] Ensemble variance quantifies uncertainty: "The ensemble approach allowed quantifying model uncertainty using the variance of the retention order predictions across the trained models"
- [readme] D-MPNN learns directly from structure: "A directed message transfer neural network (D-MPNN) is used to learn directly from the structure of compounds"
- [intro] Motivation: RTs are poorly replicable across chromatographic methods: "The underutilization of retention data in automated annotation tools is due primarily to the poor replicability of RTs across different chromatographic methods"
- [discussion] Scope: RPLC only, HILIC future work: "ROASMI currently applies only to RPLC systems, which limits its application to a specific chromatographic space."
- [discussion] Scope: elemental restrictions C, H, N, O, P, S: "applies to conventional compounds within the elemental restrictions of C, H, N, O, P, and S, leaving a vast chemical space yet to be explored."
- [discussion] Concrete example of distinguishable pair prediction: "ROASMI predicts that peak 1 elutes before peak 2 with an 81% probability and a variance of 9%."
- [methods] Concrete CLI entry points for training and prediction: "To retrain ROASMI, run: python code/ROASMI_train.py"
