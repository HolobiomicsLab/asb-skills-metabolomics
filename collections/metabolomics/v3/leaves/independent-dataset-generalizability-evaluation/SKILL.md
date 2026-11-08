---
name: independent-dataset-generalizability-evaluation
description: Use when a retention-order or molecular-property model has been trained
  on one reference set (e.g., dataset_87) and you must quantify whether its performance
  transfers to unseen chromatographic runs before trusting it for annotation.
when_to_use_negative:
- The 'independent' datasets overlap compounds or analytical runs with the training
  reference set — transfer results would be optimistic and the comparison invalid.
- The task is within-reference-set model selection or hyperparameter tuning — use
  5-fold cross-validation on the training set instead of external evaluation.
- The chromatographic system is outside the model's domain (e.g., HILIC, since ROASMI
  currently applies only to RPLC systems).
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3336
tools:
- name: ROASMI
  role: Retention-order model whose trained D-MPNN/RankNet pipeline is evaluated unchanged
    on independent RPLC datasets
  repo: https://github.com/FangYuan717/ROASMI
- name: chemprop
  role: Extended molecular embedding module (D-MPNN) whose representations are compared
    for cross-dataset generalizability
  repo: https://github.com/chemprop/chemprop
- name: PyTorch
  role: Deep-learning backend used to run and evaluate the trained models on independent
    datasets
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1186/s13321-025-00968-8
    title: 'ROASMI: accelerating small molecule identification by repurposing retention
      data'
  source_content_hash: sha256:beb4d57474172ed0b5ec2eaff69fe842237dd0d1f8bde2f258fd519c13d10f71
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/FangYuan717/ROASMI
  tools:
  - PyTorch
  - ROASMI
  - chemprop
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3336
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# Independent dataset generalizability evaluation

## Summary

Protocol for testing whether a retention-order model (e.g., ROASMI trained with a chosen molecular representation) generalizes beyond its training reference set by evaluating on truly independent chromatography datasets, as ROASMI did with 71 independent RPLC datasets.

## When to use

Use when a retention-order or molecular-property model has been trained on one reference set (e.g., dataset_87) and you must quantify whether its performance transfers to unseen chromatographic runs before trusting it for annotation.

## When NOT to use

- The 'independent' datasets overlap compounds or analytical runs with the training reference set — transfer results would be optimistic and the comparison invalid.
- The task is within-reference-set model selection or hyperparameter tuning — use 5-fold cross-validation on the training set instead of external evaluation.
- The chromatographic system is outside the model's domain (e.g., HILIC, since ROASMI currently applies only to RPLC systems).

## Inputs

- Trained retention-order model (e.g., ROASMI RankNet + D-MPNN embedding trained on dataset_87)
- Independent test datasets (71 independent RPLC datasets in the ROASMI study)
- Per-fold cross-validation metrics from the 5-fold comparison for baseline consistency

## Outputs

- CSV of generalizability evaluation metrics per independent dataset for the selected representations
- CSV of five-fold comparison metrics across the five candidate representations
- Ranking of representations by independent-dataset performance to support final selection

## How to apply

After selecting candidate representations via within-reference-set 5-fold cross-validation (as with Rdkit_norm, Morgan_count, D-MPNN, Hybrid_1, and Hybrid_2 in ROASMI), evaluate the top performers on datasets that share no compounds or analytical runs with the training reference set. Apply the identical prediction pipeline used in cross-validation — here the chemprop-based D-MPNN molecular embedding module feeding the RankNet retention prediction module — to each independent dataset without retraining, and compute the same per-fold metrics used during CV so results are directly comparable. In ROASMI, the learned D-MPNN representation (ROASMI) matched Morgan_count on the dataset_87 reference set but won on most of the 71 independent RPLC datasets, demonstrating that cross-validation alone can mask generalization differences. Aggregate per-dataset metrics into a CSV to support the generalizability comparison (Fig. 5a-style analysis). Judge success by whether the representation's ranking on independent datasets is consistent with, or better than, its cross-validated ranking.

## Related tools

- **ROASMI** (Retention-order model whose trained D-MPNN/RankNet pipeline is evaluated unchanged on independent RPLC datasets) — https://github.com/FangYuan717/ROASMI
- **chemprop** (Extended molecular embedding module (D-MPNN) whose representations are compared for cross-dataset generalizability) — https://github.com/chemprop/chemprop
- **PyTorch** (Deep-learning backend used to run and evaluate the trained models on independent datasets)

## Examples

```
python code/ROASMI_predict.py  # then score predictions on each independent dataset and aggregate per-dataset metrics into the generalizability CSV
```

## Evaluation signals

- The same metric used in the 5-fold comparison is computed on every independent dataset, enabling apples-to-apples ranking against the cross-validated baseline.
- The selected representation (D-MPNN/ROASMI) outperforms the runner-up (e.g., Morgan_count) on most of the independent datasets, reproducing the article's finding.
- Metrics are aggregated into two CSVs: one for the five-representation cross-validated comparison and one for the two-representation independent-dataset evaluation.
- No retraining or refitting occurred on the independent datasets; predictions come from the models fixed after the dataset_87 training stage.

## Limitations

- Good independent-dataset performance was demonstrated for RPLC only; HILIC retention models with generalizability remain future work.
- The model cannot distinguish molecular pairs whose retention is very close (e.g., 4-hydroxybenzoic acid vs 2,5-dihydroxybenzoic acid), and very close peaks may order inconsistently across experiments, adding noise to evaluation.
- ROASMI covers only conventional compounds within elemental restrictions of C, H, N, O, P, and S, so generalizability beyond that chemical space is unexplored.

## Evidence

- [methods] Generalization evaluation protocol: [evidence span withheld — anchor status: not_found]
- [results] Key comparative finding: [evidence span withheld — anchor status: not_found]
- [abstract] Scope of validated generalizability: "The generalizability of ROASMI is proven by 71 independent reversed-phase liquid chromatography (RPLC) datasets."
- [readme] Model modules being evaluated: "A directed message transfer neural network (D-MPNN) is used to learn directly from the structure of compounds, allowing prediction of compounds in new chemical spaces"
