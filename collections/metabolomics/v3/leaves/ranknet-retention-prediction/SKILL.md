---
name: ranknet-retention-prediction
description: Use when you need to retrain ROASMI on a user-defined RPLC reference
  set (e.g. dataset_83/dataset_84) and predict pair-wise retention scores to rank
  candidate molecules for small-molecule identification.
when_to_use_negative:
- Retention data were measured under HILIC or other non-RPLC chromatographic systems
  — ROASMI applies only to RPLC at eluent pH ~2.7 and HILIC generalizability is still
  future work
- Compounds contain elements outside C, H, N, O, P, and S — the model is restricted
  to this elemental chemical space
- Two candidate molecules' chromatographic peaks are close enough that retention order
  is not consistent across experiments — these indistinguishable pairs should be excluded
  from identification
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3345
tools:
- name: ROASMI
  role: Framework coupling D-MPNN molecular embedding with RankNet retention prediction;
    provides ROASMI_train.py, ROASMI_predict.py and Identify.py
  repo: https://github.com/FangYuan717/ROASMI
- name: chemprop
  role: D-MPNN molecular property prediction library extended as the molecular embedding
    module of ROASMI
  repo: https://github.com/chemprop/chemprop
- name: aalto-ics-kepaco/msms_rt_score_integration
  role: Source of MS/MS scores that can be integrated with ROASMI retention scores
    via Identify.py
  repo: https://github.com/aalto-ics-kepaco/msms_rt_score_integration
- name: descriptastorus
  role: Dependency installed from GitHub to support descriptor generation within the
    chemprop-based embedding
  repo: https://github.com/bp-kelley/descriptastorus
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1186/s13321-025-00968-8
    title: 'ROASMI: accelerating small molecule identification by repurposing retention
      data'
  source_content_hash: sha256:f6159b9bf851e0a1ad1840232e19ac6cb1ec8c9cc3eb18e7b360610e282fc28d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/FangYuan717/ROASMI
  tools:
  - PyTorch
  - ROASMI
  - aalto-ics-kepaco/msms_rt_score_integration
  - chemprop
  - descriptastorus
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3345
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# ranknet-retention-prediction

## Summary

Use a Ranking Neural Network (RankNet), coupled to a D-MPNN molecular embedding module (chemprop), to learn compound elution orders from RPLC reference sets and predict retention scores for candidate molecules in ROASMI.

## When to use

Use when you need to retrain ROASMI on a user-defined RPLC reference set (e.g. dataset_83/dataset_84) and predict pair-wise retention scores to rank candidate molecules for small-molecule identification.

## When NOT to use

- Retention data were measured under HILIC or other non-RPLC chromatographic systems — ROASMI applies only to RPLC at eluent pH ~2.7 and HILIC generalizability is still future work
- Compounds contain elements outside C, H, N, O, P, and S — the model is restricted to this elemental chemical space
- Two candidate molecules' chromatographic peaks are close enough that retention order is not consistent across experiments — these indistinguishable pairs should be excluded from identification

## Inputs

- Reference-set SMILES with retention order labels (CSV, e.g. data/predict_toy_set.csv style)
- Ready-to-identify candidate compound SMILES (e.g. identify_toy_set.csv in the 'data' folder)
- Initial pre-trained ROASMI model checkpoints (ROASMI_1–ROASMI_5)
- Optional MS/MS scores from external scorers (e.g. aalto-ics-kepaco/msms_rt_score_integration)

## Outputs

- Pair-wise retention scores and elution-order probabilities with variance for each candidate pair
- Rank scores used to annotate detection peaks lacking MS/MS spectra
- Combined MS/MS + retention scores after Identify.py integration

## How to apply

Train the molecular embedding module — a D-MPNN extension of chemprop — on SMILES from the reference set so molecules are mapped directly into a generic chemical space learned for retention, bypassing hand-crafted fingerprints. Train the retention prediction module, a Ranking Neural Network (RankNet), to learn elution order of compound pairs from the reference set rather than absolute RTs. Predict retention scores for ready-to-identify compounds with ROASMI_predict.py, then combine them with MS/MS scores via Identify.py. Fine-tune with user reference sets (e.g. retrain two HILIC models using 'dataset_83' as the reference set and 'dataset_84' as the test set) to improve rank-score only when chemical/chromatographic spaces overlap (ChemSS/ChrSS); ensemble predictions across ROASMI_1–ROASMI_5 models yield a prediction variance used to flag indistinguishable pairs (e.g. probability 81%, variance 9%).

## Related tools

- **ROASMI** (Framework coupling D-MPNN molecular embedding with RankNet retention prediction; provides ROASMI_train.py, ROASMI_predict.py and Identify.py) — https://github.com/FangYuan717/ROASMI
- **chemprop** (D-MPNN molecular property prediction library extended as the molecular embedding module of ROASMI) — https://github.com/chemprop/chemprop
- **aalto-ics-kepaco/msms_rt_score_integration** (Source of MS/MS scores that can be integrated with ROASMI retention scores via Identify.py) — https://github.com/aalto-ics-kepaco/msms_rt_score_integration
- **descriptastorus** (Dependency installed from GitHub to support descriptor generation within the chemprop-based embedding) — https://github.com/bp-kelley/descriptastorus

## Examples

```
python code/ROASMI_train.py && python code/ROASMI_predict.py && python code/Identify.py
```

## Evaluation signals

- Rank-score deltas between initial-train and retrained predictions improve substantially only when ChemSS and ChrSS overlap (e.g. A1/A2 improved by 0.21/0.1 vs negligible A11/A12)
- Retention-order prediction variance across the ROASMI_1–ROASMI_5 ensemble is low for distinguishable pairs (e.g. 9% variance at 81% elution probability)
- Predicted elution order is consistent across experimental replicates for pairs whose chromatographic peaks are sufficiently separated
- Validation should reproduce the reported transferability trend matching Supplementary Fig. 4 across the 12 reference/prediction combinations

## Limitations

- ROASMI applies only to RPLC systems at eluent pH ~2.7; constructing a generalizable HILIC model is future work
- Restricted to conventional compounds with elements C, H, N, O, P, and S; broader chemical space is unexplored
- Cannot distinguish certain molecular pairs with close retention (e.g. 4-hydroxybenzoic acid vs 2,5-dihydroxybenzoic acid)
- RT outliers caused by slight experimental condition changes are unresolved and need mechanistic study
- Machine-learning predictions are imperfect and must be integrated with solid mechanistic understanding

## Evidence

- [readme] RankNet was used to learn elution orders: "A Ranking Neural Network (RankNet) was used to learn elution orders"
- [readme] D-MPNN extension of chemprop for molecular embedding: "A directed message transfer neural network (D-MPNN) is used to learn directly from the structure of compounds, allowing prediction of compounds in new chemical spaces"
- [methods] Retraining command: "To retrain ROASMI, run: python code/ROASMI_train.py"
- [methods] Prediction command: "To predict the retention scores of each compound in the ready-to-identify datasets, run: python code/ROASMI_predict.py"
- [methods] Integration with MS/MS scores: "we integrate predicted retention scores with MS/MS scores by running: python code/Identify.py"
- [methods] HILIC retraining example: "retrain two HILIC models using 'dataset_83' as the reference set and 'dataset_84' as the test set"
- [results] Transferability trend example: "ROASMI predicts that peak 1 elutes before peak 2 with an 81% probability and a variance of 9%."
- [discussion] Indistinguishable pair exclusion: "Such pairs should be excluded from the subsequent small molecule identification process."
