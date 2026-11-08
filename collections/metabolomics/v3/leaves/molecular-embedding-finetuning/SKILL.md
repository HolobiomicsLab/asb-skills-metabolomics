---
name: molecular-embedding-finetuning
description: Use when you have a pre-trained ROASMI checkpoint and a new user-defined
  retention reference set, and need to retrain the model and evaluate how far its
  rank-score gains transfer to new chemical and chromatographic spaces.
when_to_use_negative:
- The candidate molecules fall outside the elemental restrictions of C, H, N, O, P,
  and S or lie in a chemical space far from the retraining reference set — ChemSS
  is low and gains will be negligible.
- The chromatographic system is HILIC or otherwise outside the RPLC/pH ~2.7 space
  the initial models were trained for, with no matching user-defined reference set
  available.
- You only need retention scores from the existing pre-trained ensemble without adapting
  to a new reference set — the base prediction workflow (ROASMI_predict.py alone)
  suffices.
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_0154
- http://edamontology.org/topic_3520
tools:
- name: ROASMI
  role: Core model being fine-tuned; provides ROASMI_train.py and ROASMI_predict.py
    for retraining and prediction
  repo: https://github.com/FangYuan717/ROASMI
- name: chemprop
  role: Molecular embedding module (D-MPNN) whose learned representations are fine-tuned
    to the new reference set
  repo: https://github.com/chemprop/chemprop
- name: PyTorch
  role: Underlying deep-learning framework for training the D-MPNN and RankNet modules;
    GPU-enabled install may be required
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1186/s13321-025-00968-8
    title: 'ROASMI: accelerating small molecule identification by repurposing retention
      data'
  source_content_hash: sha256:175b67f7d0b1421d3919c106bd2dd4634a01460dd4fd8b260ad8e3fb94fe3124
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/FangYuan717/ROASMI
  tools:
  - PyTorch
  - ROASMI
  - chemprop
  edam_operation: http://edamontology.org/operation_3927
  edam_topics:
  - http://edamontology.org/topic_0154
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# molecular-embedding-finetuning

## Summary

Fine-tune the pre-trained ROASMI model's D-MPNN molecular embedding module and RankNet retention prediction module using a user-defined retraining reference set, and quantify rank-score improvement transferability via chemical-space (ChemSS) and chromatographic-space (ChrSS) similarity between reference and prediction sets.

## When to use

Use when you have a pre-trained ROASMI checkpoint and a new user-defined retention reference set, and need to retrain the model and evaluate how far its rank-score gains transfer to new chemical and chromatographic spaces.

## When NOT to use

- The candidate molecules fall outside the elemental restrictions of C, H, N, O, P, and S or lie in a chemical space far from the retraining reference set — ChemSS is low and gains will be negligible.
- The chromatographic system is HILIC or otherwise outside the RPLC/pH ~2.7 space the initial models were trained for, with no matching user-defined reference set available.
- You only need retention scores from the existing pre-trained ensemble without adapting to a new reference set — the base prediction workflow (ROASMI_predict.py alone) suffices.

## Inputs

- Pre-trained ROASMI model checkpoints (ROASMI_1–ROASMI_5 ensemble)
- Retraining reference set (e.g., 'dataset_83' for HILIC or one of 12 RPLC reference sets)
- Paired prediction-set compounds (e.g., 'dataset_84' or the corresponding prediction set)
- Data folder contents including predict_toy_set.csv and identify_toy_set.csv
- ChemSS and ChrSS similarity metrics for each reference/prediction pair

## Outputs

- Retrained ROASMI model weights per reference set
- Predicted retention scores for each prediction-set compound
- Rank-score deltas (retrained − initial) per compound pair
- Transferability summary table grouped by ChemSS/ChrSS
- Dataset-size ablation results

## How to apply

Load the initial ROASMI checkpoints (ROASMI_1–ROASMI_5 ensemble, trained for RPLC at eluent pH ~2.7) and the 12 reference/prediction dataset combinations from the packaged data folder. For each combination, fine-tune the D-MPNN molecular embedding module (an extension of chemprop's directed message-passing network, learned directly from compound structures) together with the RankNet retention prediction module by running the reference set as training data through the ROASMI_train.py script. Then use ROASMI_predict.py to predict retention scores for the paired prediction-set compounds, and compute rank-score deltas between the initial-train and retrained predictions for each compound pair. Group the 12 combinations by chemical-space similarity (ChemSS) and chromatographic-space similarity (ChrSS) — the article shows substantial improvements only when reference and prediction sets share overlapping spaces (e.g., A1 and A2 gained 0.21 and 0.1 in rank-score, whereas A11 and A12 showed negligible gains) — and verify the resulting transferability trend against Supplementary Fig. 4. Optionally run a dataset-size ablation by varying the number of reference compounds per retraining run to record how rank-score deltas depend on training-set size.

## Related tools

- **ROASMI** (Core model being fine-tuned; provides ROASMI_train.py and ROASMI_predict.py for retraining and prediction) — https://github.com/FangYuan717/ROASMI
- **chemprop** (Molecular embedding module (D-MPNN) whose learned representations are fine-tuned to the new reference set) — https://github.com/chemprop/chemprop
- **PyTorch** (Underlying deep-learning framework for training the D-MPNN and RankNet modules; GPU-enabled install may be required)

## Examples

```
python code/ROASMI_train.py --ref data/dataset_83.csv --pred data/dataset_84.csv && python code/ROASMI_predict.py --model_path models/retrained/ --pred_path data/predict_toy_set.csv
```

## Evaluation signals

- Rank-score delta for combinations with high ChemSS/ChrSS (e.g., A1 and A2) should reproduce the reported improvements of 0.21 and 0.1 respectively.
- Rank-score delta for low-similarity combinations (e.g., A11, A12) should remain near zero, matching the negligible gains reported.
- The retrained-vs-initial rank-score patterns across all 12 dataset combinations must match Supplementary Fig. 4.
- Variance of retention-order predictions across the 5-member ROASMI ensemble should remain computable, confirming the ensemble structure survives fine-tuning.
- Dataset-size ablation results should show a monotonic (or documented non-monotonic) dependence of rank-score delta on the number of reference compounds.

## Limitations

- ROASMI applies only to RPLC systems at an eluent pH of around 2.7; constructing a HILIC retention model with similar generalizability is future work.
- The model is limited to conventional compounds within the elemental restrictions of C, H, N, O, P, and S, leaving a vast chemical space unexplored.
- Transferability is not universal: rank-score improvements depend on overlap between the retraining reference set and the prediction set in both chemical and chromatographic space.
- When the chromatographic peaks of two compounds are close enough together, their retention order may not be consistent across experiments, limiting what fine-tuning can resolve.

## Evidence

- [results] Fine-tuning ROASMI yields substantial rank-score improvements only when the retraining reference set shares overlapping chemical and chromatographic spaces with the prediction set (e.g., A1 and A2 improved by 0.21 and 0.1, whereas A11 and A12 showed negligible gains).: [evidence span withheld — anchor status: not_found]
- [methods] Retraining ROASMI models via ROASMI_train.py script: "To retrain ROASMI, run: python code/ROASMI_train.py"
- [methods] Predicting retention scores for ready-to-identify datasets: "To predict the retention scores of each compound in the ready-to-identify datasets, run: python code/ROASMI_predict.py"
- [methods] Molecular embedding module is an extension of chemprop with D-MPNN: "This module is an extension of chemprop described in the paper [Analyzing Learned Molecular Representations for Property Prediction]"
- [readme] Ensemble of four/five initial ROASMI models quantifies uncertainty via variance of predictions: "The ensemble approach allowed quantifying model uncertainty using the variance of the retention order predictions across the trained models."
- [methods] HILIC retraining example: dataset_83 as reference, dataset_84 as test: "retrain two HILIC models using 'dataset_83' as the reference set and 'dataset_84' as the test set"
- [abstract] ROASMI flexible enough to be retrained with user-defined reference sets: "ROASMI is flexible enough to be retrained with user-defined reference sets and is compatible with other MS/MS scorers"
- [discussion] ROASMI only applies to RPLC systems; HILIC model construction is future work: "ROASMI currently applies only to RPLC systems, which limits its application to a specific chromatographic space."
