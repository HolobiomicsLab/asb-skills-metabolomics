---
name: molecular-representation-training-cross-validation
description: Use when comparing candidate molecular representations (RDKit-normalized
  descriptors, Morgan counts, D-MPNN, hybrid embeddings) for retention-order prediction,
  via 5-fold cross-validation on a reference training set followed by independent-dataset
  generalizability testing.
when_to_use_negative:
- Retention data from HILIC chromatographic systems — ROASMI currently applies only
  to RPLC systems, and HILIC model construction with generalizability is stated as
  future work
- Compounds outside the elemental restrictions of C, H, N, O, P, and S — the broader
  chemical space is unexplored
- The goal is already-decided retention scoring with a trained model rather than representation
  comparison — use ROASMI_predict.py instead of retraining
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: ROASMI
  role: Framework whose RankNet retention prediction module is trained with 5-fold
    cross-validation for each molecular representation; provides ROASMI_train.py for
    retraining
  repo: https://github.com/FangYuan717/ROASMI
- name: chemprop
  role: D-MPNN molecular embedding module (extension of chemprop) that learns molecular
    representations directly from structure
  repo: https://github.com/chemprop/chemprop
- name: PyTorch
  role: Deep-learning backend for the chemprop-based molecular embedding module and
    RankNet training
- name: descriptastorus
  role: Provides RDKit2D-normalized and Morgan-count descriptor generators used for
    the descriptor-based representations (Rdkit_norm, Morgan_count)
  repo: https://github.com/bp-kelley/descriptastorus
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1186/s13321-025-00968-8
    title: 'ROASMI: accelerating small molecule identification by repurposing retention
      data'
  source_content_hash: sha256:6439e2aef7d550e4443e1563ad796043ed00774f9932b64fc11fc4b5af65a066
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/FangYuan717/ROASMI
  tools:
  - PyTorch
  - ROASMI
  - chemprop
  - descriptastorus
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# molecular-representation-training-cross-validation

## Summary

Trains the ROASMI RankNet retention-order model on the dataset_87 reference set under 5-fold cross-validation using five candidate molecular representations (Rdkit_norm, Morgan_count, D-MPNN, Hybrid_1, Hybrid_2), then evaluates the top two on independent test datasets to select the best-generalizing representation.

## When to use

Use when comparing candidate molecular representations (RDKit-normalized descriptors, Morgan counts, D-MPNN, hybrid embeddings) for retention-order prediction, via 5-fold cross-validation on a reference training set followed by independent-dataset generalizability testing.

## When NOT to use

- Retention data from HILIC chromatographic systems — ROASMI currently applies only to RPLC systems, and HILIC model construction with generalizability is stated as future work
- Compounds outside the elemental restrictions of C, H, N, O, P, and S — the broader chemical space is unexplored
- The goal is already-decided retention scoring with a trained model rather than representation comparison — use ROASMI_predict.py instead of retraining

## Inputs

- dataset_87 reference set (retention-order training data)
- Compound structures (SMILES) for the five molecular representations: Rdkit_norm, Morgan_count, D-MPNN, Hybrid_1, Hybrid_2
- Independent RPLC test datasets (71 datasets used for generalizability evaluation)

## Outputs

- CSV file with per-fold test metrics for the five-representation 5-fold cross-validation comparison (Fig. 5a reproduction)
- CSV file with generalizability metrics for the two selected representations on the independent test datasets
- Selected best-generalizing representation (D-MPNN/ROASMI in this study)

## How to apply

Load the dataset_87 reference set and configure the chemprop-based molecular embedding module for each of the five representations: Rdkit_norm, Morgan_count, D-MPNN, Hybrid_1, and Hybrid_2. Train the ROASMI Ranking Neural Network (RankNet) retention prediction module on dataset_87 using 5-fold cross-validation, holding out one fold as the test set per fold, and record held-out test performance per representation to reproduce the comparison in Fig. 5a. Select the two best-performing representations from the five-fold comparison and evaluate them on the independent test datasets (the 71 independent RPLC datasets) using the same metrics. Aggregate per-fold test metrics and independent-dataset results into two CSV files: one for the five-representation comparison and one for the two-representation generalizability evaluation. The rationale is that a learned D-MPNN representation maps molecules directly into generic chemical spaces attuned to retention without hand-designed fingerprints, and in this study D-MPNN (ROASMI) matched Morgan_count on dataset_87 but outperformed it on most of the 71 independent datasets.

## Related tools

- **ROASMI** (Framework whose RankNet retention prediction module is trained with 5-fold cross-validation for each molecular representation; provides ROASMI_train.py for retraining) — https://github.com/FangYuan717/ROASMI
- **chemprop** (D-MPNN molecular embedding module (extension of chemprop) that learns molecular representations directly from structure) — https://github.com/chemprop/chemprop
- **PyTorch** (Deep-learning backend for the chemprop-based molecular embedding module and RankNet training)
- **descriptastorus** (Provides RDKit2D-normalized and Morgan-count descriptor generators used for the descriptor-based representations (Rdkit_norm, Morgan_count)) — https://github.com/bp-kelley/descriptastorus

## Examples

```
python code/ROASMI_train.py
```

## Evaluation signals

- Held-out test metrics are recorded for all five representations across all five folds, reproducing the Fig. 5a comparison
- Two representations are selected from the five-fold comparison before any independent-dataset evaluation is run
- The two selected representations are evaluated on independent datasets with the same metrics used in the five-fold comparison
- Two CSV outputs exist: one for the five-representation comparison, one for the two-representation generalizability evaluation
- Sanity check against the article's finding: the learned D-MPNN representation (ROASMI) should perform comparably to Morgan_count on dataset_87 but outperform it on most of the 71 independent datasets

## Limitations

- ROASMI applies only to RPLC systems (eluent pH around 2.7 in the provided models); HILIC retention models with generalizability are future work
- The model cannot distinguish certain molecular pairs with very close retention (e.g., 4-hydroxybenzoic acid vs 2,5-dihydroxybenzoic acid); such indistinguishable pairs should be excluded from subsequent identification
- Retention order of very close chromatographic peaks may not be consistent across experiments, and RT outliers from slight experimental condition changes are unresolved
- Applies to conventional compounds within elemental restrictions of C, H, N, O, P, and S

## Evidence

- [results] D-MPNN outperforms Morgan_count on independent datasets: [evidence span withheld — anchor status: not_found]
- [abstract] Generalizability evaluation set size: "The generalizability of ROASMI is proven by 71 independent reversed-phase liquid chromatography (RPLC) datasets."
- [readme] Molecular embedding module is a chemprop D-MPNN extension: "This module is an extension of chemprop described in the paper [Analyzing Learned Molecular Representations for Property Prediction]"
- [methods] RankNet retention prediction module learns elution orders: "The retention prediction module. A Ranking Neural Network (RankNet) was used to learn elution orders"
- [methods] Retraining entry point: "To retrain ROASMI, run: python code/ROASMI_train.py"
