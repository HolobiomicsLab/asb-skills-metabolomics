---
name: retention-order-ranking-model-training
description: Use when you have paired compound-retention data (SMILES + retention
  times/retention order) from an RPLC reference set and need to train, cross-validate,
  or retrain a retention-order prediction model or compare molecular representations.
when_to_use_negative:
- Compounds were separated under HILIC or another non-RPLC chromatographic system
  — ROASMI applies only to RPLC (eluent pH ~2.7); HILIC generalizable models remain
  future work
- The task is only to score already-trained models on new compounds (use ROASMI_predict.py)
  or to integrate scores with MS/MS scorers (use Identify.py) rather than retraining
- Compounds fall outside the elemental restrictions of C, H, N, O, P, and S — model
  applicability to broader chemical space is unexplored
edam_operation: http://edamontology.org/operation_2423
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: ROASMI
  role: 'End-to-end retention-order model: RankNet retention prediction module plus
    chemprop-extended molecular embedding; trained via ROASMI_train.py'
  repo: https://github.com/FangYuan717/ROASMI
- name: chemprop
  role: D-MPNN message-passing neural network underlying the molecular embedding module,
    learning molecular representations directly from structure
  repo: https://github.com/chemprop/chemprop
- name: PyTorch
  role: Deep learning framework required to train the D-MPNN and RankNet modules (GPU-enabled
    install supported)
- name: descriptastorus
  role: Provides RDKit2D normalized descriptors (Rdkit_norm) and Morgan count descriptors
    used as candidate feature representations in the comparison
  repo: https://github.com/bp-kelley/descriptastorus
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1186/s13321-025-00968-8
    title: 'ROASMI: accelerating small molecule identification by repurposing retention
      data'
  source_content_hash: sha256:0f180599a3413021807ce05af6d88d80ec3db27483417c789380f3df5b4997cf
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/FangYuan717/ROASMI
  tools:
  - PyTorch
  - ROASMI
  - chemprop
  - descriptastorus
  edam_operation: http://edamontology.org/operation_2423
  edam_topics:
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# retention-order-ranking-model-training

## Summary

Trains ROASMI's RankNet-based retention-order model on a reference retention dataset (e.g. dataset_87) using a chemprop-extended D-MPNN molecular embedding module, with 5-fold cross-validation and comparison across candidate molecular representations to pick the best-generalizing representation.

## When to use

Use when you have paired compound-retention data (SMILES + retention times/retention order) from an RPLC reference set and need to train, cross-validate, or retrain a retention-order prediction model or compare molecular representations.

## When NOT to use

- Compounds were separated under HILIC or another non-RPLC chromatographic system — ROASMI applies only to RPLC (eluent pH ~2.7); HILIC generalizable models remain future work
- The task is only to score already-trained models on new compounds (use ROASMI_predict.py) or to integrate scores with MS/MS scorers (use Identify.py) rather than retraining
- Compounds fall outside the elemental restrictions of C, H, N, O, P, and S — model applicability to broader chemical space is unexplored

## Inputs

- Reference retention dataset (e.g. dataset_87) with compound SMILES and retention times/retention order pairs
- Independent RPLC test datasets for generalizability evaluation (paper used 71)
- Candidate molecular representation specification (Rdkit_norm, Morgan_count, D-MPNN, Hybrid_1, Hybrid_2)
- ROASMI environment (conda environment.yml with PyTorch, CUDA >= 8.0, cuDNN, chemprop, descriptastorus)

## Outputs

- Trained ROASMI retention-order RankNet model(s) (ROASMI_1–ROASMI_5 ensemble)
- Per-fold cross-validation test metrics for each molecular representation (reproducing Fig. 5a)
- Independent-dataset evaluation metrics CSV (generalizability assessment)
- Five-representation comparison CSV

## How to apply

Load the reference set (e.g. dataset_87) and configure the chemprop-based molecular embedding module for each candidate representation (Rdkit_norm, Morgan_count, D-MPNN, Hybrid_1, Hybrid_2). Train the ROASMI Ranking Neural Network (RankNet) retention prediction module with 5-fold cross-validation, holding out one fold as the test set per fold, and record test performance per fold (Fig. 5a comparison). Select the best-performing representations and evaluate them on independent RPLC datasets (the paper used 71) to assess generalizability, aggregating per-fold and independent-dataset metrics into two CSV output files. Rationale: the learned D-MPNN embedding maps molecules directly into chemical spaces tuned to retention without hand-designed fingerprints, which generalizes better than Morgan_count even when 5-fold performance is comparable. Use the ensemble of trained models (ROASMI_1–ROASMI_5) to quantify prediction uncertainty via variance of retention-order predictions.

## Related tools

- **ROASMI** (End-to-end retention-order model: RankNet retention prediction module plus chemprop-extended molecular embedding; trained via ROASMI_train.py) — https://github.com/FangYuan717/ROASMI
- **chemprop** (D-MPNN message-passing neural network underlying the molecular embedding module, learning molecular representations directly from structure) — https://github.com/chemprop/chemprop
- **PyTorch** (Deep learning framework required to train the D-MPNN and RankNet modules (GPU-enabled install supported))
- **descriptastorus** (Provides RDKit2D normalized descriptors (Rdkit_norm) and Morgan count descriptors used as candidate feature representations in the comparison) — https://github.com/bp-kelley/descriptastorus

## Examples

```
git clone https://github.com/FangYuan717/ROASMI.git && cd ROASMI && conda env create -f environment.yml && python code/ROASMI_train.py
```

## Evaluation signals

- 5-fold cross-validation completes with one fold held out as test per fold and per-fold test metrics are recorded for all five representations (Fig. 5a reproduced)
- The selected representation (D-MPNN/ROASMI) matches or beats Morgan_count within folds but outperforms it on most of the 71 independent RPLC datasets — check generalizability CSV
- Ensemble variance across ROASMI_1–ROASMI_5 is computable for retention-order predictions (e.g. 9% variance reported for a distinguishable pair at 81% probability)
- Output consists of exactly two CSV files: one for the five-representation comparison and one for the two-representation generalizability evaluation
- Predicted retention-order probabilities near 0.5 with high variance flag indistinguishable pairs, which should be excluded from downstream identification

## Limitations

- ROASMI currently applies only to RPLC systems (eluent pH ~2.7); no HILIC support — constructing generalizable HILIC retention models is future work
- Limited to conventional compounds within elemental restrictions of C, H, N, O, P, and S; broader chemical space unexplored
- The model cannot distinguish certain molecular pairs with close retention (e.g. 4-hydroxybenzoic acid and 2,5-dihydroxybenzoic acid); such indistinguishable pairs should be excluded from downstream identification
- When chromatographic peaks of two compounds are close enough, their retention order may not be consistent across experiments, and RT outliers from slight experimental condition changes remain unresolved

## Evidence

- [methods] The retention prediction module: "The retention prediction module. A Ranking Neural Network (RankNet) was used to learn elution orders"
- [methods] chemprop-based molecular embedding: "This module is an extension of chemprop described in the paper [Analyzing Learned Molecular Representations for Property Prediction]"
- [abstract] 71 independent RPLC datasets: "The generalizability of ROASMI is proven by 71 independent reversed-phase liquid chromatography (RPLC) datasets."
- [readme] retraining command: [evidence span withheld — anchor status: not_found]
- [readme] ensemble uncertainty: "The ensemble approach allowed quantifying model uncertainty using the variance of the retention order predictions across the trained models."
