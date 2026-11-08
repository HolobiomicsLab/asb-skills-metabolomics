---
name: shadow-price-feature-based-classification
description: 'Use when predicting metabolite-protein interactions from constraint-based
  models: shadow-price feature tables and gold standard labels (e.g. stitch_ecoli,
  stitch_yeast) are available and a random forest classifier is required.'
when_to_use_negative:
- Metabolite 128-bit molecular fingerprints (from generate_fp.py with RDKit) are used
  as features instead of shadow prices — run evaluate.py fp rather than the shadow-price
  workflow
- The input pairs are not covered by any of the supported gold standards (piazza,
  reznik, stitch_ecoli, stitch_yeast), since the pipeline is tied to those integrated
  datasets
- GUROBI is not installed or licensed, since shadow-price computation via optimization.py
  requires an active GUROBI license
edam_operation: http://edamontology.org/operation_3936
edam_topics:
- http://edamontology.org/topic_0128
- http://edamontology.org/topic_2269
tools:
- name: SARTRE
  role: Framework orchestrating shadow-price feature extraction from GEMs and random
    forest prediction of metabolite-protein interactions
  repo: https://github.com/fayazsoleymani/SARTRE
- name: COBRAToolbox
  role: MATLAB toolbox used to curate the GEMs (iJO1366 and Yeast-GEM) before shadow-price
    calculation
- name: GUROBI
  role: Optimizer implementing the constraint-based optimization to compute shadow
    prices per reaction
  repo: https://www.gurobi.com/
- name: scikit-learn
  role: Provides the random forest classifier and evaluation metrics used by evaluate.py
    and shared.py
  repo: https://scikit-learn.org/stable/
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioadv/vbad098
    title: Prediction of metabolite–protein interactions based on integration of machine
      learning and constraint-based modeling
  source_content_hash: sha256:29e35ef2af6736b3761e1a792d3303b1489e099b0b2c74e573a79e11af54a74c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/fayazsoleymani/SARTRE
  tools:
  - COBRAToolbox
  - GUROBI
  - Python
  - SARTRE
  - scikit-learn
  edam_operation: http://edamontology.org/operation_3936
  edam_topics:
  - http://edamontology.org/topic_0128
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7222222222222222
n_claims: 18
---

# shadow-price-feature-based-classification

## Summary

Predict metabolite-protein interactions (MPIs) using shadow prices from flux variability analysis of genome-scale metabolic models (GEMs such as iJO1366 and Yeast-GEM) as features for random forest classifiers, as implemented in the SARTRE framework.

## When to use

Use when predicting metabolite-protein interactions from constraint-based models: shadow-price feature tables and gold standard labels (e.g. stitch_ecoli, stitch_yeast) are available and a random forest classifier is required.

## When NOT to use

- Metabolite 128-bit molecular fingerprints (from generate_fp.py with RDKit) are used as features instead of shadow prices — run evaluate.py fp rather than the shadow-price workflow
- The input pairs are not covered by any of the supported gold standards (piazza, reznik, stitch_ecoli, stitch_yeast), since the pipeline is tied to those integrated datasets
- GUROBI is not installed or licensed, since shadow-price computation via optimization.py requires an active GUROBI license

## Inputs

- Curated genome-scale metabolic models (iJO1366.mat, Yeast-GEM.mat -> ModelIrrevOpt90.mat, yeast_IrrevOpt90.mat)
- Gold standard MPI datasets (piazza, reznik, stitch_ecoli, stitch_yeast)
- Shadow-price feature tables from optimization.py (saved in the opt directory, provided as .zip archives)
- STITCH confidence score (150, 400, 700, or 900) for STITCH-derived gold standards

## Outputs

- Trained random forest classifiers with evaluation metrics (average AUC ~0.86 for E. coli and ~0.85 for S. cerevisiae)
- Accuracy and cosine similarity of predictions between organism-specific classifiers on shared MPIs (e.g. shared_results.csv; expected ~0.754/0.755 and ~0.980)
- Feature importance rankings demonstrating the key role of shadow prices

## How to apply

Curate the GEMs (iJO1366, Yeast-GEM) with COBRAToolbox in MATLAB (e.g. modelPreparing_iJO1366.m produces ModelIrrevOpt90.mat), then compute shadow prices per reaction by running optimization.py with the GUROBI optimizer and its Python interface. Preprocess the shadow prices for downstream classifiers by running cleaning.py in each gold standard directory (optionally also generate_fp.py for 128-bit molecular fingerprints with RDKit). Train a random forest classifier from scikit-learn by running evaluate.py in each gold standard directory (STITCH-derived datasets additionally take a confidence score argument: 150 low, 400 medium, 700 high, 900 highest). For cross-organism agreement, exclude metabolite-protein pairs shared between the two GEMs, train two separate models on the remaining pairs, apply both to the shared pairs, and report accuracy and cosine similarity between prediction vectors (expect ~0.754/0.755 accuracy and ~0.980 cosine similarity). Rationale: shadow prices are the key features for MPI prediction, so classification quality depends on correct GEM curation and optimization before training.

## Related tools

- **SARTRE** (Framework orchestrating shadow-price feature extraction from GEMs and random forest prediction of metabolite-protein interactions) — https://github.com/fayazsoleymani/SARTRE
- **COBRAToolbox** (MATLAB toolbox used to curate the GEMs (iJO1366 and Yeast-GEM) before shadow-price calculation)
- **GUROBI** (Optimizer implementing the constraint-based optimization to compute shadow prices per reaction) — https://www.gurobi.com/
- **scikit-learn** (Provides the random forest classifier and evaluation metrics used by evaluate.py and shared.py) — https://scikit-learn.org/stable/

## Examples

```
cd stitch_ecoli && python3 evaluate.py sp 400
```

## Evaluation signals

- Reported accuracies on shared metabolite-protein pairs reproduce the paper's values of approximately 0.754 (E. coli model) and 0.755 (S. cerevisiae model)
- Cosine similarity between the two organism-specific prediction vectors is approximately 0.980
- Average AUC of the random forest classifiers is approximately 0.86 (E. coli) and 0.85 (S. cerevisiae) as in the abstract
- Feature importance ranking places shadow prices as the key features for classification
- Permutation tests (label_permutation and feature_permutation) show performance drops consistent with non-random signal

## Limitations

- GUROBI requires installation and an active license before optimization.py can compute shadow prices
- Archives with optimization results must be unzipped before running the pipeline
- Results depend on the chosen STITCH confidence score (150/400/700/900); a different score changes the gold standard and metrics
- Model curation is organism-specific: modelPreparing_iJO1366.m and modelPreparing_yeastGEM.m must be run separately for iJO1366 and Yeast-GEM

## Evidence

- [abstract] SARTRE random forest on shadow prices: "a computational framework, termed SARTRE, that employs features corresponding to shadow prices determined in the context of flux variability analysis to predict metabolite-protein interactions"
- [abstract] Ranking of features: "Ranking of features based on their importance for classification demonstrated the key role of shadow prices in predicting metabolite-protein interactions."
- [methods] GEM curation: "Curation of these mentioned GEMs have been performed by the  COBRAToolbox in MATLAB"
- [methods] optimization.py: "Calculation of shadow prices can be performed in their directory by changing the working directory followed by executing `optimization.py`"
- [readme] STITCH confidence argument: "Gold standard from STITCH also gets an additional argument as a confidence score(150: low, 400: medium, 700: high, 900: highest) to perform this process."
- [readme] GUROBI license requirement: "Make sure to be installed and have an active license for it beforehand."
