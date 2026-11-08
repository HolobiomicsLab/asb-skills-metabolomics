---
name: cross-fold-hyperparameter-representation-comparison
description: Use when several molecular featurizations are candidates for a property/retention
  prediction model and you need a fair, fold-level performance comparison plus an
  independent-dataset generalizability check before choosing one.
when_to_use_negative:
- The chromatographic system is HILIC rather than RPLC — the ROASMI retention-order
  model currently applies only to RPLC systems, and HILIC models with generalizability
  are future work
- Compounds fall outside the elemental restrictions of C, H, N, O, P, and S — the
  model's chemical space is limited to conventional compounds
- A single trained model is already chosen and validated; this skill is for comparing
  representations/hyperparameters, not for scoring new compounds
edam_topics:
- http://edamontology.org/topic_0154
tools:
- name: chemprop
  role: Underlying molecular embedding module (D-MPNN) extended by ROASMI to learn
    representations for the retention-order model
  repo: https://github.com/chemprop/chemprop
- name: PyTorch
  role: Deep-learning framework for training the RankNet retention prediction module
    (GPU-enabled version may need manual install)
- name: descriptastorus
  role: Descriptor generator (e.g., RDKit2D normalized, Morgan counts) supporting
    the non-learned candidate representations such as Rdkit_norm and Morgan_count
  repo: https://github.com/bp-kelley/descriptastorus
- name: ROASMI
  role: End-to-end pipeline (molecular embedding + RankNet retention prediction) whose
    representations are compared under 5-fold CV
  repo: https://github.com/FangYuan717/ROASMI
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1186/s13321-025-00968-8
    title: 'ROASMI: accelerating small molecule identification by repurposing retention
      data'
  source_content_hash: sha256:db3db31dfbaae2496c3513edf3696a3d2e7f4af30cab83467bc302768cb599ea
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/chemprop/chemprop
  tools:
  - PyTorch
  - ROASMI
  - chemprop
  - descriptastorus
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# cross-fold-hyperparameter-representation-comparison

## Summary

Compares candidate molecular representations (Rdkit_norm, Morgan_count, D-MPNN, Hybrid_1, Hybrid_2) for retention-order modeling by training the chemprop/RankNet-based ROASMI pipeline under 5-fold cross-validation on a reference set (dataset_87), then stress-tests the top representations on independent RPLC datasets to pick the most generalizing representation.

## When to use

Use when several molecular featurizations are candidates for a property/retention prediction model and you need a fair, fold-level performance comparison plus an independent-dataset generalizability check before choosing one.

## When NOT to use

- The chromatographic system is HILIC rather than RPLC — the ROASMI retention-order model currently applies only to RPLC systems, and HILIC models with generalizability are future work
- Compounds fall outside the elemental restrictions of C, H, N, O, P, and S — the model's chemical space is limited to conventional compounds
- A single trained model is already chosen and validated; this skill is for comparing representations/hyperparameters, not for scoring new compounds

## Inputs

- dataset_87 reference set of compounds with retention-order pairs for RPLC (eluent pH ~2.7)
- Molecular structures as SMILES strings for each candidate representation
- Configuration for the chemprop-based molecular embedding module for each of the five representations (Rdkit_norm, Morgan_count, D-MPNN, Hybrid_1, Hybrid_2)
- 71 independent reversed-phase LC datasets for the generalizability evaluation

## Outputs

- CSV file aggregating per-fold test metrics from the 5-fold cross-validation of all five representations (Fig. 5a comparison)
- CSV file aggregating independent-dataset evaluation metrics for the two selected representations

## How to apply

Load the dataset_87 reference set and configure the chemprop-based molecular embedding module (a D-MPNN extension of chemprop) with each of the five representations. Train the ROASMI Ranking Neural Network (RankNet) retention prediction module on dataset_87 with 5-fold cross-validation, holding out one fold as the test set per fold, and record per-fold test metrics as in Fig. 5a. Select the two best-performing representations from the five-fold comparison and evaluate them on 71 independent RPLC datasets to assess generalizability, computing the same metrics used in the cross-validation. Aggregate the per-fold test metrics and independent-dataset results into two CSV output files — one for the five-representation comparison and one for the two-representation generalizability evaluation. The rationale is that learned D-MPNN representations map molecules directly into chemical spaces tuned to the property, so they may generalize beyond artificially designed fingerprints even when cross-validated performance ties (e.g., D-MPNN/ROASMI matched Morgan_count on dataset_87 but beat it on most independent datasets).

## Related tools

- **chemprop** (Underlying molecular embedding module (D-MPNN) extended by ROASMI to learn representations for the retention-order model) — https://github.com/chemprop/chemprop
- **PyTorch** (Deep-learning framework for training the RankNet retention prediction module (GPU-enabled version may need manual install))
- **descriptastorus** (Descriptor generator (e.g., RDKit2D normalized, Morgan counts) supporting the non-learned candidate representations such as Rdkit_norm and Morgan_count) — https://github.com/bp-kelley/descriptastorus
- **ROASMI** (End-to-end pipeline (molecular embedding + RankNet retention prediction) whose representations are compared under 5-fold CV) — https://github.com/FangYuan717/ROASMI

## Examples

```
conda env create -f environment.yml && python code/ROASMI_train.py
```

## Evaluation signals

- Per-fold test metrics are recorded for all five folds for each of the five representations, mirroring the Fig. 5a comparison
- Exactly two best-performing representations are selected from the five-fold comparison before the independent-dataset evaluation
- Independent-dataset evaluation covers the 71 independent RPLC datasets using the same metrics as the cross-validation
- Two CSV outputs are produced: one for the five-representation CV comparison, one for the two-representation generalizability evaluation
- If D-MPNN (ROASMI) ties with Morgan_count on dataset_87 CV, the tie-break should favor the representation that wins on most independent datasets

## Limitations

- Cross-validated performance on the reference set can tie while generalizability differs substantially (D-MPNN matched Morgan_count on dataset_87 but outperformed it on most of the 71 independent datasets), so selection must not rely on CV metrics alone
- Results are specific to RPLC systems; no HILIC retention-order comparison is supported
- Retention order of very close chromatographic peaks may not be consistent across experiments, and RT outliers from slight experimental condition changes are unresolved

## Evidence

- [results] In the initial training stage on dataset_87, the learned D-MPNN representation (ROASMI) performed comparably to Morgan_count, but ROASMI outperformed Morgan_count on most of the 71 independent datasets, making it the better generalizing representation.: [evidence span withheld — anchor status: not_found]
- [methods] The molecular embedding module is an extension of chemprop.: "This module is an extension of chemprop described in the paper [Analyzing Learned Molecular Representations for Property Prediction]"
- [abstract] The generalizability of ROASMI is proven by 71 independent reversed-phase liquid chromatography (RPLC) datasets.: "The generalizability of ROASMI is proven by 71 independent reversed-phase liquid chromatography (RPLC) datasets."
- [readme] A directed message transfer neural network (D-MPNN) is used to learn directly from the structure of compounds, allowing prediction of compounds in new chemical spaces.: "A directed message transfer neural network (D-MPNN) is used to learn directly from the structure of compounds, allowing prediction of compounds in new chemical spaces"
- [readme] ROASMI trains four initial models (ROASMI_1 - ROASMI_5) with an ensemble approach that quantifies model uncertainty via variance of retention order predictions.: "We provide four initial ROASMI models (ROASMI_1 - ROASMI_5) for predicting the retention behavior of compounds in the reversed-phase liquid chromatography (RPLC) system with an eluent pH of around"
