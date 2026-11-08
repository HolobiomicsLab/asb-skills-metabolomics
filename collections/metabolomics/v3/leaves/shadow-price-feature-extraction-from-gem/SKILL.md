---
name: shadow-price-feature-extraction-from-gem
description: Use when you need numeric features for metabolites from a constraint-based
  GEM — specifically FVA-derived shadow prices — to feed a downstream classifier such
  as SARTRE's random forest.
when_to_use_negative:
- The metabolite features are already available (e.g. met_sps_t80_replaced.csv or
  met_fp128.csv exists) — skip directly to evaluate.py
- You only need molecular-structure features; generate 128-bit Morgan fingerprints
  with generate_fp.py/RDKit instead of running the shadow-price optimization
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_2259
- http://edamontology.org/topic_0203
tools:
- name: COBRAToolbox
  role: Curates the GEMs in MATLAB, converting iJO1366.mat / Yeast-GEM.mat into irreversible
    optimized models (ModelIrrevOpt90.mat, yeast_IrrevOpt90.mat)
  repo: https://github.com/fayazsoleymani/SARTRE
- name: GUROBI
  role: Optimizer used through its Python interface to compute shadow prices for every
    reaction in optimization.py
- name: MATLAB
  role: Environment in which the GEM curation scripts (modelPreparing_iJO1366.m, modelPreparing_yeastGEM.m)
    run
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioadv/vbad098
    title: Prediction of metabolite–protein interactions based on integration of machine
      learning and constraint-based modeling
  source_content_hash: sha256:445081485232e97a2e617f511c67ff66264327022e078d2d3749ab9d9cb17695
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/fayazsoleymani/SARTRE
  tools:
  - Bio.Entrez
  - COBRAToolbox
  - GUROBI
  - MATLAB
  - Python
  - RDKit
  - SARTRE
  - scikit-learn
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_2259
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 16
---

# shadow-price-feature-extraction-from-gem

## Summary

Extracts per-reaction shadow prices from curated genome-scale metabolic models (iJO1366, Yeast8) via FVA-based optimization with GUROBI, producing metabolite features that SARTRE uses to predict metabolite-protein interactions.

## When to use

Use when you need numeric features for metabolites from a constraint-based GEM — specifically FVA-derived shadow prices — to feed a downstream classifier such as SARTRE's random forest.

## When NOT to use

- The metabolite features are already available (e.g. met_sps_t80_replaced.csv or met_fp128.csv exists) — skip directly to evaluate.py
- You only need molecular-structure features; generate 128-bit Morgan fingerprints with generate_fp.py/RDKit instead of running the shadow-price optimization

## Inputs

- Curated irreversible GEM file, e.g. ModelIrrevOpt90.mat produced from iJO1366.mat by modelPreparing_iJO1366.m
- Gold-standard MPI dataset directory (piazza, reznik, stitch_ecoli, stitch_yeast) containing optimization.py and cleaning.py
- Active GUROBI license and Python environment with the GUROBI interface

## Outputs

- Per-reaction optimization results saved as files in the opt directory (also provided as .zip archives)
- Cleaned shadow-price feature table met_sps_t80_replaced.csv

## How to apply

First curate the GEM with COBRAToolbox in MATLAB: run modelPreparing_iJO1366.m on iJO1366.mat (or modelPreparing_yeastGEM.m on Yeast-GEM.mat) to produce the irreversible optimized model ModelIrrevOpt90.mat (yeast_IrrevOpt90.mat). Then, with an active GUROBI license, cd into the gold-standard directory (piazza, reznik, stitch_ecoli, or stitch_yeast) and run `python3 optimization.py`, which computes shadow prices for every reaction via the GUROBI Python interface and saves per-reaction results in the opt directory. Preprocess the shadow prices by running `python3 cleaning.py` to produce met_sps_t80_replaced.csv for the downstream classifier. The shadow prices, rather than structural fingerprints, carry the predictive signal — feature-importance ranking identified them as key for MPI prediction.

## Related tools

- **COBRAToolbox** (Curates the GEMs in MATLAB, converting iJO1366.mat / Yeast-GEM.mat into irreversible optimized models (ModelIrrevOpt90.mat, yeast_IrrevOpt90.mat)) — https://github.com/fayazsoleymani/SARTRE
- **GUROBI** (Optimizer used through its Python interface to compute shadow prices for every reaction in optimization.py)
- **MATLAB** (Environment in which the GEM curation scripts (modelPreparing_iJO1366.m, modelPreparing_yeastGEM.m) run)

## Examples

```
cd reznik && python3 optimization.py && python3 cleaning.py
```

## Evaluation signals

- The opt directory contains per-reaction result files (saved by reaction number) after optimization.py completes; archives should be unzipped before use
- cleaning.py produces the expected output file met_sps_t80_replaced.csv in the gold-standard directory
- Downstream evaluate.py sp runs yield average AUC near the reported 0.86 for E. coli (0.85 for S. cerevisiae), confirming the shadow prices are informative
- GUROBI license is active — optimization fails without one, so a successful full run over all reactions is itself a check

## Limitations

- A paid/active GUROBI license is required beforehand; the optimization cannot run without it
- Result archives (.zip) must be manually unzipped before further processing
- Shadow prices are model- and condition-dependent: performance was assessed across different media compositions (acetate, fructose, glycerol, mannose, succinate vs glucose), so metrics change with the carbon source

## Evidence

- [methods] Curating the GEMs: "`modelPreparing_iJO1366.m` uses `iJO1366.mat` as the input and creates `ModelIrrevOpt90.mat`"
- [methods] Calculating shadow prices: "The optimization process has been implemented by [GUROBI](https://www.gurobi.com/) optimizer and its python interface"
- [methods] Data pre-processing: "The framework is followed by preprocessing calculated shadow prices for downstream classifiers changing the directory to specific gold standard and executing `cleaning.py`"
- [readme] README: "The results of optimization will be saved on the `opt` directory for each reaction by its number."
- [abstract] Findings: "Ranking of features based on their importance for classification demonstrated the key role of shadow prices in predicting metabolite-protein interactions."
