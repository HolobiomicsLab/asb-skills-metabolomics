---
name: retention-order-probability-sourcing-from-roasmi-predictions
description: Use when ready-to-identify datasets carry candidate molecular pairs with
  ROASMI retention scores and you must exclude indistinguishable pairs or infer elution
  order for identification.
when_to_use_negative:
- Chromatographic system is HILIC rather than RPLC — ROASMI currently applies only
  to RPLC systems.
- Candidate compounds fall outside the elemental restrictions of C, H, N, O, P, and
  S.
- A single trained model is available without the full ensemble, since variance-based
  uncertainty quantification requires the multi-model ensemble.
edam_operation: http://edamontology.org/operation_0286
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: ROASMI
  role: Ensemble of ranking neural network (RankNet) retention-order models that produce
    pairwise retention-order probabilities and variances between candidate compounds
  repo: https://github.com/FangYuan717/ROASMI
- name: chemprop
  role: 'Molecular embedding module: D-MPNN maps candidate compounds into generic
    chemical spaces to predict retention in new chemical spaces'
  repo: https://github.com/chemprop/chemprop
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1186/s13321-025-00968-8
    title: 'ROASMI: accelerating small molecule identification by repurposing retention
      data'
  source_content_hash: sha256:0361202a68b34e49c52e646f6470ee2cd35352cec2be1bb175d7ed4bc249ca59
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/FangYuan717/ROASMI
  tools:
  - ROASMI
  - chemprop
  edam_operation: http://edamontology.org/operation_0286
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# retention-order-probability-sourcing-from-ROASMI-predictions

## Summary

Converts ROASMI-predicted retention-order probabilities P_ij and variances between candidate compound pairs into filtered, distinguishable pairwise constraints used to resolve peak identity in untargeted metabolomics.

## When to use

Use when ready-to-identify datasets carry candidate molecular pairs with ROASMI retention scores and you must exclude indistinguishable pairs or infer elution order for identification.

## When NOT to use

- Chromatographic system is HILIC rather than RPLC — ROASMI currently applies only to RPLC systems.
- Candidate compounds fall outside the elemental restrictions of C, H, N, O, P, and S.
- A single trained model is available without the full ensemble, since variance-based uncertainty quantification requires the multi-model ensemble.

## Inputs

- ready-to-identify dataset CSV (e.g. data/predict_toy_set.csv) with candidate compound structures
- trained ROASMI ensemble models (ROASMI_1–ROASMI_5) for RPLC system at eluent pH ~2.7

## Outputs

- pairwise retention-order probabilities P_ij between candidate compounds
- prediction variance per pair quantifying ensemble uncertainty
- set of distinguishable and indistinguishable candidate pairs

## How to apply

Load the ready-to-identify dataset (e.g. 'data/predict_toy_set.csv', default via pred_path in ROASMI_predict.py) containing candidate compounds and run the ROASMI ensemble of five trained models (ROASMI_1–ROASMI_5) to obtain pairwise retention-order probabilities P_ij and prediction variances. For each candidate pair, treat the predicted retention order as reliable when probability is high and variance low; e.g. in the article's example, peak 1 elutes before peak 2 with an 81% probability and 9% variance, making the pair distinguishable. Exclude pairs marked indistinguishable — where ROASMI predicts near-equal retention order with high variance (such as 4-hydroxybenzoic acid vs 2,5-dihydroxybenzoic acid in dataset_2) — from subsequent small-molecule identification. The ensemble design is critical: variance of retention-order predictions across the five trained models quantifies uncertainty, so a single-model output is insufficient for sourcing distinguishability. Retain the per-pair probabilities and variances as inputs for downstream integration with MS/MS scores via msms_rt_score_integration.

## Related tools

- **ROASMI** (Ensemble of ranking neural network (RankNet) retention-order models that produce pairwise retention-order probabilities and variances between candidate compounds) — https://github.com/FangYuan717/ROASMI
- **chemprop** (Molecular embedding module: D-MPNN maps candidate compounds into generic chemical spaces to predict retention in new chemical spaces) — https://github.com/chemprop/chemprop

## Examples

```
python code/ROASMI_predict.py  # default pred_path is 'data/predict_toy_set.csv'
```

## Evaluation signals

- Predicted retention-order probability exceeds ~80% with variance below ~10% for pairs declared distinguishable (article example: 81% probability, 9% variance).
- Indistinguishable pairs (high variance, near-50% probability) are excluded from downstream identification.
- Predictions generalize on independent RPLC datasets; ROASMI was validated on 71 independent RPLC datasets.
- Output pairwise constraints remain compatible with MS/MS score integration in msms_rt_score_integration.
- All candidate pairs in the ready-to-identify dataset receive a probability and variance value from the five-model ensemble.

## Limitations

- ROASMI applies only to RPLC systems with eluent pH around 2.7; HILIC retention model construction with generalizability remains future work.
- Model is restricted to conventional compounds within elemental restrictions of C, H, N, O, P, and S.
- When chromatographic peaks of two compounds are close enough, their retention order may not be consistent across experiments, producing RT outliers.
- Cannot distinguish certain molecular pairs with very close retention, e.g. 4-hydroxybenzoic acid and 2,5-dihydroxybenzoic acid.

## Evidence

- [readme] Ensemble quantifies prediction variance across trained models: "The ensemble approach allowed quantifying model uncertainty using the variance of the retention order predictions across the trained models."
- [discussion] Concrete probability and variance example for distinguishable pair: "ROASMI predicts that peak 1 elutes before peak 2 with an 81% probability and a variance of 9%."
- [discussion] Indistinguishable pairs should be excluded from identification: "Such pairs should be excluded from the subsequent small molecule identification process."
- [methods] Prediction command and default input path: "To predict the retention scores of each compound in the ready-to-identify datasets, run: python code/ROASMI_predict.py"
- [methods] Default input file name for prediction: "the default pred_path is 'data/predict_toy_set. csv'"
- [discussion] ROASMI scope limited to RPLC systems: "ROASMI currently applies only to RPLC systems, which limits its application to a specific chromatographic space."
