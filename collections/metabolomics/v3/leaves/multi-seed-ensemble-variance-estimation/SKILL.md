---
name: multi-seed-ensemble-variance-estimation
description: Use when training a RankNet-type pairwise retention-order model where
  single-seed predictions are noisy, and you need an interpretable uncertainty estimate
  (prediction variance) to mark distinguishable vs indistinguishable molecular pairs.
when_to_use_negative:
- When only a single deterministic retention-time prediction is needed rather than
  an uncertainty-aware pairwise order (e.g. the RT outliers the paper says require
  in-depth mechanistic study).
- When the chromatographic system is HILIC — the article states ROASMI applies only
  to RPLC and that constructing HILIC models with generalizability is future work.
- When the molecular pair falls outside the elemental restrictions of C, H, N, O,
  P, and S, since ROASMI's learned embeddings have not been validated on a broader
  chemical space.
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_0154
- http://edamontology.org/topic_3524
tools:
- name: ROASMI
  role: Provides the RankNet retention prediction module whose pairwise elution-probability
    outputs are ensembled across five random seeds; retraining is done via python
    code/ROASMI_train.py
  repo: https://github.com/FangYuan717/ROASMI
- name: chemprop
  role: Supplies the directed message passing neural network (D-MPNN) that learns
    molecular embeddings feeding the RankNet head; the ROASMI molecular embedding
    module is an extension of chemprop
  repo: https://github.com/chemprop/chemprop
- name: PyTorch
  role: Framework in which the RankNet training loop, Adam optimizer, and binary cross-entropy
    pairwise loss are implemented; note a GPU-enabled build plus cuda>=8.0 and cuDNN
    are needed to run ROASMI on GPUs
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1186/s13321-025-00968-8
    title: 'ROASMI: accelerating small molecule identification by repurposing retention
      data'
  source_content_hash: sha256:9ff6b8707d44d98c93a93ff60b79bd1b89f7a09b759d4c025f3b20aa48e07bc9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/FangYuan717/ROASMI
  tools:
  - PyTorch
  - ROASMI
  - chemprop
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_0154
  - http://edamontology.org/topic_3524
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# Multi-seed ensemble variance estimation

## Summary

Train the ROASMI RankNet retention model five times from different random seeds to produce ROASMI_1–ROASMI_5, and use the cross-seed variance of predicted pairwise elution probabilities as an uncertainty signal that flags molecular pairs whose retention order is not reliably determined.

## When to use

Use when training a RankNet-type pairwise retention-order model where single-seed predictions are noisy, and you need an interpretable uncertainty estimate (prediction variance) to mark distinguishable vs indistinguishable molecular pairs.

## When NOT to use

- When only a single deterministic retention-time prediction is needed rather than an uncertainty-aware pairwise order (e.g. the RT outliers the paper says require in-depth mechanistic study).
- When the chromatographic system is HILIC — the article states ROASMI applies only to RPLC and that constructing HILIC models with generalizability is future work.
- When the molecular pair falls outside the elemental restrictions of C, H, N, O, P, and S, since ROASMI's learned embeddings have not been validated on a broader chemical space.

## Inputs

- A retention-order reference dataset split into pairwise elution-order comparisons (e.g. dataset_87 for RPLC), with train/validation/test partitions at 8:1:1
- Molecular structures (SMILES) of candidate compounds for which pairwise elution probabilities are to be predicted
- Chemprop D-MPNN molecular embeddings as input features to the RankNet model
- Hyperparameters reported in Table 1 of the ROASMI article plus an early-stopping criterion on validation pairwise loss

## Outputs

- Five independently trained RankNet ensemble members (ROASMI_1 – ROASMI_5)
- For each molecular pair, a mean pairwise elution probability P_ij (probability compound i elutes before compound j) and its cross-seed variance
- A binary distinguishable/indistinguishable call per molecular pair based on the variance and probability distance from 0.5, usable to exclude indistinguishable pairs from downstream identification

## How to apply

Train the ROASMI RankNet retention model (chemprop D-MPNN embeddings fed into a pairwise binary-cross-entropy loss on observed elution order) with five different random seeds, yielding five ensemble members ROASMI_1 through ROASMI_5. For each candidate molecular pair, each member outputs a pairwise elution probability P_ij (probability that compound Ci elutes before Cj); compute the mean and the variance of these five probabilities. The article grounds this in an example where peak 1 eluting before peak 2 has an 81% probability with 9% variance, indicating a distinguishable pair; pairs with low variance and probability far from 0.5 are treated as reliably ordered, while pairs with high variance or probability near 0.5 are indistinguishable and should be excluded from downstream small-molecule identification. Rationale: the ensemble variance quantifies how consistent the learned elution order is across independently initialized models, directly reflecting the uncertainty behavior the paper reports (low variance for clearly separated peaks, high variance for indistinguishable pairs).

## Related tools

- **ROASMI** (Provides the RankNet retention prediction module whose pairwise elution-probability outputs are ensembled across five random seeds; retraining is done via python code/ROASMI_train.py) — https://github.com/FangYuan717/ROASMI
- **chemprop** (Supplies the directed message passing neural network (D-MPNN) that learns molecular embeddings feeding the RankNet head; the ROASMI molecular embedding module is an extension of chemprop) — https://github.com/chemprop/chemprop
- **PyTorch** (Framework in which the RankNet training loop, Adam optimizer, and binary cross-entropy pairwise loss are implemented; note a GPU-enabled build plus cuda>=8.0 and cuDNN are needed to run ROASMI on GPUs)

## Examples

```
python code/ROASMI_train.py  # retrain with a chosen random seed; repeat with 5 seeds to build ROASMI_1–ROASMI_5, then python code/ROASMI_predict.py to obtain P_ij and cross-seed variance per pair.
```

## Evaluation signals

- The ensemble must contain exactly five members (ROASMI_1 – ROASMI_5), each trained under a different random seed on the same reference set.
- For each held-out molecular pair the output must be a probability P_ij in [0,1] for 'compound i elutes before compound j', with a cross-seed variance attached.
- Sanity-check against the paper's reported example: an ordered pair with an 81% probability and 9% variance corresponds to a distinguishable pair.
- Pairs whose predicted probability is close to 0.5 and/or whose variance is high should coincide with pairs the article flags as indistinguishable (e.g. 4-hydroxybenzoic acid vs 2,5-dihydroxybenzoic acid); if not, seed diversity or early stopping may be misconfigured.
- Verify training used the 8:1:1 train/validation/test split with early stopping on validation pairwise loss and Table 1 hyperparameters across all seeds.

## Limitations

- ROASMI currently applies only to RPLC systems, limiting applicability to a specific chromatographic space; HILIC generalizability is future work.
- The model is limited to conventional compounds within the elemental restrictions of C, H, N, O, P, and S.
- Retention order of very close chromatographic peaks may not be consistent across experiments, so the ensemble variance may still be ambiguous for such pairs.
- Addressing RT outliers resulting from slight changes in experimental conditions requires in-depth mechanistic studies beyond what the ensemble variance captures.
- Machine learning approaches are imperfect; the article stresses integration with solid mechanistic understanding rather than relying on the ensemble uncertainty alone.

## Evidence

- [full_text] Pairwise elution probability P_ij trained with binary cross-entropy on observed elution order: [evidence span withheld — anchor status: not_found]
- [readme] Ensemble members ROASMI_1 to ROASMI_5 with variance-based uncertainty: "We provide four initial ROASMI models (ROASMI_1 - ROASMI_5) for predicting the retention behavior of compounds in the reversed-phase liquid chromatography (RPLC) system"
- [readme] Ensemble approach quantifies model uncertainty via variance of retention order predictions: "The ensemble approach allowed quantifying model uncertainty using the variance of the retention order predictions across the trained models."
- [discussion] Reported example: 81% probability with 9% variance indicates a distinguishable pair: "ROASMI predicts that peak 1 elutes before peak 2 with an 81% probability and a variance of 9%."
- [discussion] Indistinguishable pairs should be excluded from subsequent small molecule identification: "Such pairs should be excluded from the subsequent small molecule identification process."
