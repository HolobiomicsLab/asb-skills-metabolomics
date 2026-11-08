---
name: genome-scale-metabolic-model-shadow-price-analysis
description: Use when predicting metabolite-protein interactions from constraint-based
  models and you need physiologically grounded metabolite features (shadow prices)
  instead of, or alongside, chemical fingerprints.
when_to_use_negative:
- You already have a rich metabolite feature table (e.g. precomputed fingerprints)
  and do not need constraint-based shadow-price features.
- No active GUROBI license is available — the optimization step (`optimization.py`)
  cannot run without it.
- The task targets a metabolic model other than the curated GEMs and no curation/irreversibility
  step has been performed on it.
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_2259
tools:
- name: SARTRE
  role: Overall framework integrating shadow prices from constraint-based modeling
    with machine learning for metabolite-protein interaction prediction
  repo: https://github.com/fayazsoleymani/SARTRE
- name: COBRAToolbox
  role: Curation of the GEMs (iJO1366, Yeast-GEM) in MATLAB to produce the irreversible
    optimized models
  repo: https://github.com/opencobra/cobratoolbox
- name: GUROBI
  role: Optimizer (with Python interface) that solves the FBA/variability problems
    from which shadow prices are extracted
- name: scikit-learn
  role: Provides the random forest classifier and evaluation metrics used on shadow-price
    features
- name: MATLAB
  role: Environment in which GEM curation with COBRAToolbox is performed
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioadv/vbad098
    title: Prediction of metabolite–protein interactions based on integration of machine
      learning and constraint-based modeling
  source_content_hash: sha256:55463b67e195a329b066232a75b660b90707090e5e3e0893c60db5ed4bcda23c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/fayazsoleymani/SARTRE
  tools:
  - COBRAToolbox
  - GUROBI
  - MATLAB
  - Python
  - SARTRE
  - scikit-learn
  edam_operation: http://edamontology.org/operation_3927
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_2259
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6428571428571429
n_claims: 14
---

# genome-scale-metabolic-model-shadow-price-analysis

## Summary

Computes shadow prices from flux variability analysis of curated genome-scale metabolic models (iJO1366, Yeast-GEM) using the GUROBI optimizer, and uses these dual values as metabolite features for metabolite-protein interaction prediction in the SARTRE framework.

## When to use

Use when predicting metabolite-protein interactions from constraint-based models and you need physiologically grounded metabolite features (shadow prices) instead of, or alongside, chemical fingerprints.

## When NOT to use

- You already have a rich metabolite feature table (e.g. precomputed fingerprints) and do not need constraint-based shadow-price features.
- No active GUROBI license is available — the optimization step (`optimization.py`) cannot run without it.
- The task targets a metabolic model other than the curated GEMs and no curation/irreversibility step has been performed on it.

## Inputs

- Genome-scale metabolic model file (e.g. iJO1366.mat or Yeast-GEM.mat)
- Curated irreversible optimized model (ModelIrrevOpt90.mat / yeast_IrrevOpt90.mat)
- Gold-standard metabolite-protein pair sets (piazza, reznik, stitch_ecoli, stitch_yeast) with STITCH confidence scores (150 low, 400 medium, 700 high, 900 highest)
- Metabolite structures for optional 128-bit fingerprint generation (RDKit)

## Outputs

- Per-reaction shadow-price optimization results saved in an `opt` directory
- Preprocessed shadow-price feature table `met_sps_t80_replaced.csv`
- 128-bit molecular fingerprint table `met_fp128.csv`
- Trained random forest classifier and accuracy/AUC metrics (e.g. average AUC 0.86 for E. coli, 0.85 for S. cerevisiae)

## How to apply

First curate the GEM with COBRAToolbox in MATLAB: `modelPreparing_iJO1366.m` converts `iJO1366.mat` into the irreversible, optimized `ModelIrrevOpt90.mat` (analogously `modelPreparing_yeastGEM.m` produces `yeast_IrrevOpt90.mat`). Then run `optimization.py` (GUROBI via its Python interface) inside each gold-standard directory (piazza, reznik, stitch_ecoli, stitch_yeast) to compute shadow prices, saved per reaction in an `opt` directory. Preprocess the shadow prices with `cleaning.py` and optionally generate 128-bit molecular fingerprints with `generate_fp.py` (RDKit), producing `met_sps_t80_replaced.csv` and `met_fp128.csv`. Shadow prices are then used as features for a scikit-learn random forest classifier trained/evaluated via `evaluate.py sp` (for STITCH datasets add the confidence score, e.g. 400 for medium). Feature-importance ranking showed shadow prices are the key predictors, so verify their dominance when interpreting results.

## Related tools

- **SARTRE** (Overall framework integrating shadow prices from constraint-based modeling with machine learning for metabolite-protein interaction prediction) — https://github.com/fayazsoleymani/SARTRE
- **COBRAToolbox** (Curation of the GEMs (iJO1366, Yeast-GEM) in MATLAB to produce the irreversible optimized models) — https://github.com/opencobra/cobratoolbox
- **GUROBI** (Optimizer (with Python interface) that solves the FBA/variability problems from which shadow prices are extracted)
- **scikit-learn** (Provides the random forest classifier and evaluation metrics used on shadow-price features)
- **MATLAB** (Environment in which GEM curation with COBRAToolbox is performed)

## Examples

```
cd stitch_ecoli; python3 optimization.py; python3 cleaning.py; python3 evaluate.py sp 400
```

## Evaluation signals

- Shadow prices rank highest in feature-importance scores of the trained random forest classifier.
- Classifier reaches average AUC of approximately 0.86 (E. coli) and 0.85 (S. cerevisiae) on held-out pairs.
- Per-reaction optimization outputs are present in the `opt` directory for each gold standard after running `optimization.py`.
- Subsystem-exclusion checks: models trained excluding 'Alternate Carbon Metabolism' and 'Cofactor and Prosthetic Group Biosynthesis' yield accuracies of ~0.615 and ~0.815 respectively on the STITCH-E.coli (400) test set.
- Label/feature permutation tests (`permutation_tests.py`) should show performance dropping to chance levels, confirming real signal in shadow-price features.

## Limitations

- Prediction accuracy drops when restricted to specific subsystems (0.615 for Alternate Carbon Metabolism, 0.815 for Cofactor and Prosthetic Group Biosynthesis on STITCH-E.coli medium confidence) due to a larger number of false positives, indicating shadow prices covering different metabolic systems increase prediction accuracy.
- GUROBI requires an active license before `optimization.py` can be executed.
- Optimization result archives must be unzipped before downstream preprocessing steps can run.

## Evidence

- [abstract] SARTRE employs shadow prices: "a computational framework, termed SARTRE, that employs features corresponding to shadow prices determined in the context of flux variability analysis to predict metabolite-protein interactions"
- [methods] Curation via COBRAToolbox: "Curation of these mentioned GEMs have been performed by the  COBRAToolbox in MATLAB"
- [methods] GUROBI optimization: "The optimization process has been implemented by [GUROBI](https://www.gurobi.com/) optimizer and its python interface"
- [abstract] Shadow prices as key features: "Ranking of features based on their importance for classification demonstrated the key role of shadow prices in predicting metabolite-protein interactions."
- [readme] Shadow-price CSV output: "The results will be saved in `met_sps_t80_replaced.csv` and `met_fp128.csv`, respectively."
- [readme] STITCH confidence argument: "Gold standard from STITCH also gets an additional argument as a confidence score(150: low, 400: medium, 700: high, 900: highest)"
- [readme] Subsystem accuracy drop: "we excluded metabolite-protein pairs of two subsystems of *E. coli* separately from stitch_ecoli and trained two models on remaining pairs"
