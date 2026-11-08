---
name: genome-scale-metabolic-model-data-handling
description: Use when a pipeline needs curated GEM files (e.g., .mat models) converted
  to irreversible, optimized versions as the basis for computing reaction-level shadow
  prices used as MPI-prediction features.
when_to_use_negative:
- You already have a preprocessed shadow-price feature table (e.g., met_sps_t80_replaced.csv)
  — proceed directly to cleaning.py/evaluate.py instead
- Your target organisms/models are not among iJO1366 or Yeast-GEM and their curation
  scripts do not apply
edam_operation: http://edamontology.org/operation_3660
edam_topics:
- http://edamontology.org/topic_2259
- http://edamontology.org/topic_0605
tools:
- name: COBRAToolbox
  role: MATLAB toolbox used to curate iJO1366.mat and Yeast-GEM.mat into irreversible
    optimized model files
- name: MATLAB
  role: Environment in which modelPreparing_iJO1366.m and modelPreparing_yeastGEM.m
    are executed
- name: GUROBI
  role: Optimizer used via its Python interface to compute shadow prices from the
    curated models
- name: Python
  role: Runs optimization.py to perform the shadow-price optimization against the
    curated GEMs
  repo: https://github.com/fayazsoleymani/SARTRE
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioadv/vbad098
    title: Prediction of metabolite–protein interactions based on integration of machine
      learning and constraint-based modeling
  source_content_hash: sha256:f711e559a82d507354078e7f65be8a1ec49d00f1d343f0a4c7752206bf4f00db
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
  edam_operation: http://edamontology.org/operation_3660
  edam_topics:
  - http://edamontology.org/topic_2259
  - http://edamontology.org/topic_0605
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4666666666666667
n_claims: 15
---

# genome-scale-metabolic-model-data-handling

## Summary

Curate and prepare genome-scale metabolic models (GEMs) such as iJO1366 (E. coli) and Yeast-GEM/Yeast8 (S. cerevisiae) for downstream constraint-based analyses, converting them into irreversible, optimized model files from which shadow prices can be computed as machine-learning features.

## When to use

Use when a pipeline needs curated GEM files (e.g., .mat models) converted to irreversible, optimized versions as the basis for computing reaction-level shadow prices used as MPI-prediction features.

## When NOT to use

- You already have a preprocessed shadow-price feature table (e.g., met_sps_t80_replaced.csv) — proceed directly to cleaning.py/evaluate.py instead
- Your target organisms/models are not among iJO1366 or Yeast-GEM and their curation scripts do not apply

## Inputs

- iJO1366.mat (E. coli genome-scale metabolic model)
- Yeast-GEM.mat (S. cerevisiae consensus metabolic model Yeast8)
- Gold standard metabolite-protein pair lists (e.g., stitch_ecoli, stitch_yeast, piazza, reznik)

## Outputs

- ModelIrrevOpt90.mat (curated irreversible, optimized E. coli model)
- yeast_IrrevOpt90.mat (curated irreversible, optimized yeast model)
- Shadow-price results per reaction saved in the opt directory for each gold standard

## How to apply

Obtain the organism-specific GEM files (iJO1366.mat for E. coli, Yeast-GEM.mat for S. cerevisiae) and run the MATLAB curation scripts via COBRAToolbox: modelPreparing_iJO1366.m produces ModelIrrevOpt90.mat and modelPreparing_yeastGEM.m produces yeast_IrrevOpt90.mat. These curated, irreversible models are then used by optimization.py with the GUROBI optimizer and its Python interface to compute shadow prices (saved per reaction in the opt directory) for gold-standard datasets such as stitch_ecoli and stitch_yeast. The rationale is that shadow prices from flux variability analysis context encode metabolite availability information that serves as the key features for random-forest classification of metabolite-protein interactions. Note that GUROBI requires an active license before running the optimization step.

## Related tools

- **COBRAToolbox** (MATLAB toolbox used to curate iJO1366.mat and Yeast-GEM.mat into irreversible optimized model files)
- **MATLAB** (Environment in which modelPreparing_iJO1366.m and modelPreparing_yeastGEM.m are executed)
- **GUROBI** (Optimizer used via its Python interface to compute shadow prices from the curated models)
- **Python** (Runs optimization.py to perform the shadow-price optimization against the curated GEMs) — https://github.com/fayazsoleymani/SARTRE

## Examples

```
cd stitch_ecoli && python3 optimization.py
```

## Evaluation signals

- modelPreparing_iJO1366.m run on iJO1366.mat yields ModelIrrevOpt90.mat; modelPreparing_yeastGEM.m run on Yeast-GEM.mat yields yeast_IrrevOpt90.mat
- Shadow-price outputs exist per reaction in the opt directory of each gold standard folder after optimization.py completes
- Downstream classifier trained on these shadow-price features reaches average AUC of ~0.86 (E. coli) and ~0.85 (S. cerevisiae)
- GUROBI license is active and no optimization failures are reported during optimization.py execution

## Limitations

- GUROBI requires installation and an active license before shadow-price optimization can run
- Curation scripts are organism-specific (iJO1366 and Yeast-GEM only); adapting to other GEMs requires new curation scripts
- Archived optimization results are provided as .zip archives and must be unzipped before use

## Evidence

- [readme] GEM curation step: "Curation of these mentioned GEMs have been performed by the  COBRAToolbox in MATLAB. `modelPreparing_iJO1366.m` uses `iJO1366.mat` as the input and creates `ModelIrrevOpt90.mat`"
- [readme] GUROBI requirement: "The optimization process has been implemented by [GUROBI](https://www.gurobi.com/) optimizer and its python interface. Make sure to be installed and have an active license for it beforehand."
- [abstract] shadow price features finding: "Ranking of features based on their importance for classification demonstrated the key role of shadow prices in predicting metabolite-protein interactions."
