---
name: gpr-rule-parsing-protein-reaction-mapping
description: Use when reaction-level shadow prices from FBA/FVA dual solutions on
  iJO1366 or Yeast8 must be converted into protein-level features, requiring GPR rule
  parsing of the 90%-optimal irreversible models.
when_to_use_negative:
- The input is already a protein-level feature table (e.g. precomputed met_sps_t80_replaced.csv),
  so no GPR mapping is needed
- The metabolic model lacks GPR annotations for its reactions, so proteins cannot
  be associated with reactions
- The task operates purely at the reaction level (e.g. raw FVA/shadow-price solving
  in optimization.py) before any protein mapping is required
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_0602
tools:
- name: COBRAToolbox
  role: Used in MATLAB to curate the GEMs (modelPreparing_iJO1366.m, modelPreparing_yeastGEM.m)
    whose GPR rules are parsed to construct the protein-reaction association matrix
  repo: https://github.com/opencobra/cobratoolbox
- name: MATLAB
  role: Environment in which COBRAToolbox model preparation scripts produce the IrrevOpt90
    .mat models used for GPR parsing
- name: GUROBI
  role: Optimizer solving the FBA/FVA dual formulation via its Python interface to
    yield per-reaction shadow prices that the association matrix maps onto proteins
- name: Python
  role: Runs optimization.py and the downstream preprocessing/classifier scripts that
    consume the protein-mapped shadow-price features
  repo: https://github.com/fayazsoleymani/SARTRE
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioadv/vbad098
    title: Prediction of metabolite–protein interactions based on integration of machine
      learning and constraint-based modeling
  source_content_hash: sha256:fa8aff9c97feadda6dcd30a6226a4346757800d6e1d2ae952cba34db15a1e0b8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/opencobra/cobratoolbox
  tools:
  - COBRAToolbox
  - GUROBI
  - MATLAB
  - Python
  - SARTRE
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# gpr-rule-parsing-protein-reaction-mapping

## Summary

Parse the gene-protein-reaction (GPR) rules of curated genome-scale metabolic models (iJO1366's ModelIrrevOpt90.mat and Yeast-GEM's yeast_IrrevOpt90.mat) to build a protein-reaction association matrix that maps reaction-level shadow prices onto proteins as SARTRE features for metabolite-protein interaction prediction.

## When to use

Use when reaction-level shadow prices from FBA/FVA dual solutions on iJO1366 or Yeast8 must be converted into protein-level features, requiring GPR rule parsing of the 90%-optimal irreversible models.

## When NOT to use

- The input is already a protein-level feature table (e.g. precomputed met_sps_t80_replaced.csv), so no GPR mapping is needed
- The metabolic model lacks GPR annotations for its reactions, so proteins cannot be associated with reactions
- The task operates purely at the reaction level (e.g. raw FVA/shadow-price solving in optimization.py) before any protein mapping is required

## Inputs

- Curated irreversible 90%-optimal metabolic models: ModelIrrevOpt90.mat (iJO1366) and yeast_IrrevOpt90.mat (Yeast-GEM)
- GPR (gene-protein-reaction) logical rules embedded in the curated .mat models
- Per-reaction shadow-price results saved per reaction number in the opt directory after running optimization.py

## Outputs

- Protein-reaction association matrix (protein x reaction binary/association vectors) for each model
- Protein-level feature vectors aggregating valid reaction-level shadow prices for downstream random forest classifiers

## How to apply

First curate the genome-scale models with COBRAToolbox in MATLAB (modelPreparing_iJO1366.m on iJO1366.mat, modelPreparing_yeastGEM.m on Yeast-GEM.mat) to obtain the irreversible 90%-optimal-biomass versions ModelIrrevOpt90.mat and yeast_IrrevOpt90.mat. Then parse the GPR logical rules of each curated model to construct a protein-reaction association matrix, where each protein is linked to the reactions it catalyzes (AND/OR logic in GPRs determines whether one or multiple reactions map to a protein). Use this matrix to aggregate the per-reaction shadow prices computed by GUROBI via optimization.py (saved one result per reaction number in the opt directory) onto protein-level feature vectors, valid only for shadow prices passing the allowable increase/decrease and incremental/decremental (epsilon=1e-5) checks. These protein-reaction association vectors are combined with metabolite-side shadow-price features (and 128-bit RDKit fingerprints) as inputs to the sklearn random forest classifiers in evaluate.py.

## Related tools

- **COBRAToolbox** (Used in MATLAB to curate the GEMs (modelPreparing_iJO1366.m, modelPreparing_yeastGEM.m) whose GPR rules are parsed to construct the protein-reaction association matrix) — https://github.com/opencobra/cobratoolbox
- **MATLAB** (Environment in which COBRAToolbox model preparation scripts produce the IrrevOpt90 .mat models used for GPR parsing)
- **GUROBI** (Optimizer solving the FBA/FVA dual formulation via its Python interface to yield per-reaction shadow prices that the association matrix maps onto proteins)
- **Python** (Runs optimization.py and the downstream preprocessing/classifier scripts that consume the protein-mapped shadow-price features) — https://github.com/fayazsoleymani/SARTRE

## Evaluation signals

- The curated models are the 90%-optimal irreversible versions (ModelIrrevOpt90.mat / yeast_IrrevOpt90.mat) produced by the modelPreparing scripts, not the raw iJO1366.mat or Yeast-GEM.mat
- Optimization results exist for every reaction (saved per reaction number in the opt directory) so the association matrix covers the full reaction set
- Only shadow prices passing the allowable increase/decrease and incremental/decremental (epsilon=1e-5) validity checks are mapped onto proteins
- The mapped protein features feed random forest classifiers achieving reported average AUC of 0.86 (E. coli) and 0.85 (S. cerevisiae), indicating feature construction is consistent with the published pipeline

## Limitations

- GUROBI requires an active license before optimization.py can generate the shadow prices that the GPR mapping consumes
- Archives with precomputed optimization results must be unzipped before use ('MAKE sure to unzip archives')
- The curated model fixes biomass at 90% of optimum, so protein-mapped features reflect that growth constraint rather than unrestricted growth

## Evidence

- [other] GPR parsing constructs association matrix: [evidence span withheld — anchor status: not_found]
- [other] Combined with association vectors: [evidence span withheld — anchor status: not_found]
- [readme] Model curation scripts: "`modelPreparing_iJO1366.m` uses `iJO1366.mat` as the input and creates `ModelIrrevOpt90.mat`"
- [abstract] Shadow prices as key features: "Ranking of features based on their importance for classification demonstrated the key role of shadow prices in predicting metabolite-protein interactions."
