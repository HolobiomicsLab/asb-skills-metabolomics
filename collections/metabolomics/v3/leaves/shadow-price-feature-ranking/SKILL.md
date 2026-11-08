---
name: shadow-price-feature-ranking
description: Use when trained RandomForest MPI classifiers exist for SARTRE gold standards
  (piazza, reznik, stitch_ecoli, stitch_yeast) and you need to identify which feature
  class — shadow prices or fingerprints — contributes most to classification.
when_to_use_negative:
- Input is already a ranked importance table or you only need classifier performance
  metrics (accuracy/AUC/F1) without feature attribution
- The classifier is not a tree ensemble — feature_importances_ is specific to scikit-learn
  tree-based models, not e.g. the deep neural network used in the Zhao et al. comparison
- The question is predictive performance or significance testing against the fingerprint
  baseline rather than interpreting feature contributions
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_0128
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_2259
tools:
- name: SARTRE
  role: Framework providing the shadow-price features and gold-standard directories
    in which evaluate.py is run to train the classifiers whose importances are ranked
  repo: https://github.com/fayazsoleymani/SARTRE
- name: scikit-learn
  role: Supplies the RandomForestClassifier and its feature_importances_ attribute
    used for impurity-based ranking
- name: GUROBI
  role: Optimizer behind the flux variability analysis whose shadow prices form the
    ranked feature set
- name: RDKit
  role: Generates the 128-bit fingerprint comparator features ranked against shadow
    prices
provenance:
  source_task_ids:
  - task_008
  - task_002
  source_papers:
  - doi: 10.1093/bioadv/vbad098
    title: Prediction of metabolite–protein interactions based on integration of machine
      learning and constraint-based modeling
  source_content_hash: sha256:98f5575580bf5ea891e2954fb3ca2a4e51f11e584b4719581b679d2b4f172a22
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/fayazsoleymani/SARTRE
  tools:
  - GUROBI
  - Python
  - RDKit
  - SARTRE
  - scikit-learn
  edam_operation: http://edamontology.org/operation_3927
  edam_topics:
  - http://edamontology.org/topic_0128
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_2259
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 16
---

# shadow-price-feature-ranking

## Summary

Ranks random-forest feature importances to quantify how strongly shadow-price features (from flux variability analysis over GEMs) versus 128-bit RDKit fingerprint bits drive metabolite-protein interaction prediction in SARTRE, confirming shadow prices as the dominant feature class.

## When to use

Use when trained RandomForest MPI classifiers exist for SARTRE gold standards (piazza, reznik, stitch_ecoli, stitch_yeast) and you need to identify which feature class — shadow prices or fingerprints — contributes most to classification.

## When NOT to use

- Input is already a ranked importance table or you only need classifier performance metrics (accuracy/AUC/F1) without feature attribution
- The classifier is not a tree ensemble — feature_importances_ is specific to scikit-learn tree-based models, not e.g. the deep neural network used in the Zhao et al. comparison
- The question is predictive performance or significance testing against the fingerprint baseline rather than interpreting feature contributions

## Inputs

- met_sps_t80_replaced.csv (preprocessed shadow-price features per gold standard)
- met_fp128.csv (128-bit RDKit fingerprint features)
- Trained or retrainable RandomForestClassifier from evaluate.py ('sp' / 'fp' runs, STITCH confidence scores 150/400/700/900)
- Gold standard directories: piazza, reznik, stitch_ecoli, stitch_yeast

## Outputs

- feature_importance_ranked.csv (per-feature importances sorted descending, labeled by type and metabolite/protein pair)
- Per-feature-type aggregated importance comparing shadow prices against fingerprint bits across the four gold standards

## How to apply

Load the preprocessed shadow-price feature table (met_sps_t80_replaced.csv) and the 128-bit fingerprint table (met_fp128.csv) for each gold standard directory. Train (or load) the scikit-learn RandomForestClassifier via evaluate.py with 'sp' or 'fp' feature arguments, passing a confidence score (150/400/700/900) for STITCH-derived datasets. Extract the impurity-based feature_importances_ attribute from each forest, label each feature by type (shadow price vs. fingerprint bit) and its metabolite/protein pair, then sort in descending importance and aggregate per feature type. Rationale: shadow prices come from GUROBI-based flux variability optimization on curated GEMs (iJO1366, Yeast8), so ranking their impurities directly tests whether constraint-based features — not chemical fingerprints — carry the class-discriminative signal, corroborating the paper's abstract claim that ranking demonstrated the key role of shadow prices.

## Related tools

- **SARTRE** (Framework providing the shadow-price features and gold-standard directories in which evaluate.py is run to train the classifiers whose importances are ranked) — https://github.com/fayazsoleymani/SARTRE
- **scikit-learn** (Supplies the RandomForestClassifier and its feature_importances_ attribute used for impurity-based ranking)
- **GUROBI** (Optimizer behind the flux variability analysis whose shadow prices form the ranked feature set)
- **RDKit** (Generates the 128-bit fingerprint comparator features ranked against shadow prices)

## Examples

```
cd piazza && python3 evaluate.py sp
```

## Evaluation signals

- Shadow-price features occupy the top ranks of the sorted feature_importance_ranked.csv table in each gold standard
- Per-feature-type aggregation shows shadow-price importance exceeds 128-bit fingerprint importance across piazza, reznik, stitch_ecoli, and stitch_yeast
- Classifier quality of the retrained forests is consistent with reported average AUC levels (~0.86 E. coli, ~0.85 S. cerevisiae)
- Every feature row is labeled with its type (shadow price vs. fingerprint bit) and associated metabolite/protein pair

## Limitations

- Importances are impurity-based from scikit-learn random forests, so they reflect training-split variance rather than causal contribution
- GUROBI requires an installed, active license before shadow prices can be recomputed; result archives must be unzipped first
- STITCH datasets require an explicit confidence-score argument (150/400/700/900); omitting it changes the labeled set being ranked

## Evidence

- [abstract] Ranking of features based on their importance for classification demonstrated the key role of shadow prices in predicting metabolite-protein interactions.: "Ranking of features based on their importance for classification demonstrated the key role of shadow prices in predicting metabolite-protein interactions."
- [readme] Preprocessing, constructing datasets, training the random forest classifier and evaluating it can be executed by running evaluate.py: "Preprocessing, constructing datasets, training the random forest classifier and evaluating it can be executed by running `evaluate.py` in each gold standard directory."
- [methods] The classifier and metrics are employed from sklearn python library: "The classifier and metrics are employed from [sklearn](https://scikit-learn.org/stable/) python library"
- [readme] The results will be saved in met_sps_t80_replaced.csv and met_fp128.csv, respectively.: "The results will be saved in `met_sps_t80_replaced.csv` and `met_fp128.csv`, respectively."
