---
name: genome-scale-model-curation-and-irreversibility-conversion
description: Use when a constraint-based modeling pipeline needs reaction-level dual
  values (shadow prices) and requires the model to be converted to an irreversible
  form with biomass constrained to 90% of its optimum before optimization.
when_to_use_negative:
- When the downstream analysis needs FBA flux solutions at 100% optimal growth rather
  than dual/shadow-price features from a 90%-constrained model
- When input is not a genome-scale metabolic model (e.g. a pre-computed feature table
  or expression dataset)
edam_operation: http://edamontology.org/operation_3660
edam_topics:
- http://edamontology.org/topic_2259
- http://edamontology.org/topic_0203
tools:
- name: COBRAToolbox
  role: MATLAB toolbox used to curate the GEMs, convert them to irreversible form,
    and fix biomass at 90% of optimum
  repo: https://github.com/fayazsoleymani/SARTRE
- name: MATLAB
  role: environment in which the modelPreparing_*.m curation scripts are executed
- name: yeast-GEM
  role: source S. cerevisiae consensus model (Yeast8) supplied as Yeast-GEM.mat input
    to modelPreparing_yeastGEM.m
  repo: https://github.com/SysBioChalmers/yeast-GEM
- name: SARTRE
  role: downstream framework that consumes the curated irreversible 90%-optimal models
    for shadow-price feature extraction
  repo: https://github.com/fayazsoleymani/SARTRE
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioadv/vbad098
    title: Prediction of metabolite–protein interactions based on integration of machine
      learning and constraint-based modeling
  source_content_hash: sha256:1125680b7912ceb7b546f53909d2f03dea79cffe684069d2159bbdf3fa9e80b6
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
  - yeast-GEM
  edam_operation: http://edamontology.org/operation_3660
  edam_topics:
  - http://edamontology.org/topic_2259
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 14
---

# genome-scale-model-curation-and-irreversibility-conversion

## Summary

Curate genome-scale metabolic models (GEMs) with COBRAToolbox in MATLAB to produce irreversible versions fixed at 90% of optimal biomass, which serve as the substrate for FBA/FVA dual-problem shadow-price feature extraction in SARTRE.

## When to use

Use when a constraint-based modeling pipeline needs reaction-level dual values (shadow prices) and requires the model to be converted to an irreversible form with biomass constrained to 90% of its optimum before optimization.

## When NOT to use

- When the downstream analysis needs FBA flux solutions at 100% optimal growth rather than dual/shadow-price features from a 90%-constrained model
- When input is not a genome-scale metabolic model (e.g. a pre-computed feature table or expression dataset)

## Inputs

- iJO1366.mat (E. coli genome-scale metabolic model)
- Yeast-GEM.mat (S. cerevisiae consensus model Yeast8, from SysBioChalmers/yeast-GEM)

## Outputs

- ModelIrrevOpt90.mat (curated irreversible iJO1366 fixed at 90% optimal biomass)
- yeast_IrrevOpt90.mat (curated irreversible yeast-GEM fixed at 90% optimal biomass)
- protein-reaction association matrix derived from GPR rules of each curated model

## How to apply

Start from the published GEMs — iJO1366.mat for E. coli and Yeast-GEM.mat for S. cerevisiae. In MATLAB with COBRAToolbox, run modelPreparing_iJO1366.m on iJO1366.mat and modelPreparing_yeastGEM.m on Yeast-GEM.mat; each script curates the model, converts it to an irreversible form, and constrains the biomass reaction to 90% of its optimum, writing ModelIrrevOpt90.mat and yeast_IrrevOpt90.mat respectively. The 90%-optimal biomass constraint is deliberate: shadow prices are subsequently obtained by solving the FBA dual problem at this sub-optimal growth level so the LP remains feasible and dual values are informative. Next, parse the GPR rules of each curated model to build the protein-reaction association matrix, which maps reaction-level shadow prices onto proteins for the downstream classifiers. Validation is by confirming the curated outputs are the 90%-optimal irreversible versions produced by the modelPreparing scripts before any optimization.py runs.

## Related tools

- **COBRAToolbox** (MATLAB toolbox used to curate the GEMs, convert them to irreversible form, and fix biomass at 90% of optimum) — https://github.com/fayazsoleymani/SARTRE
- **MATLAB** (environment in which the modelPreparing_*.m curation scripts are executed)
- **yeast-GEM** (source S. cerevisiae consensus model (Yeast8) supplied as Yeast-GEM.mat input to modelPreparing_yeastGEM.m) — https://github.com/SysBioChalmers/yeast-GEM
- **SARTRE** (downstream framework that consumes the curated irreversible 90%-optimal models for shadow-price feature extraction) — https://github.com/fayazsoleymani/SARTRE

## Examples

```
matlab -batch "run('modelPreparing_iJO1366.m')"  % reads iJO1366.mat, writes ModelIrrevOpt90.mat
```

## Evaluation signals

- The curated outputs are exactly ModelIrrevOpt90.mat and yeast_IrrevOpt90.mat — the irreversible, 90%-optimal versions produced by the modelPreparing scripts
- Biomass reaction in each curated model is constrained to 90% of its optimum (FBA growth equals 0.9 × the unconstrained optimum)
- All reactions in the curated models are irreversible (no reversible reaction bounds remain)
- GPR rules parse successfully for every reaction so the protein-reaction association matrix covers all model reactions before shadow-price extraction

## Limitations

- Requires an active GUROBI license downstream and MATLAB with COBRAToolbox for the curation step itself
- The 90%-optimal biomass constraint means the curated models are not suitable for analyses requiring maximal-growth flux distributions
- Archived optimization results ship as .zip archives and must be unzipped before use (README: 'MAKE sure to unzip archives')

## Evidence

- [readme] Curation of these mentioned GEMs have been performed by the  COBRAToolbox in MATLAB. `modelPreparing_iJO1366.m` uses `iJO1366.mat` as the input and creates `ModelIrrevOpt90.mat`: "Curation of these mentioned GEMs have been performed by the  COBRAToolbox in MATLAB. `modelPreparing_iJO1366.m` uses `iJO1366.mat` as the input and creates `ModelIrrevOpt90.mat`"
- [readme] modelPreparing scripts produce yeast_IrrevOpt90.mat from Yeast-GEM.mat: "`modelPreparing_yeastGEM.m` uses `Yeast-GEM.mat` and creates `yeast_IrrevOpt90.mat`."
- [abstract] SARTRE metabolite features are shadow prices from FVA at 90% of optimum biomass: "a computational framework, termed SARTRE, that employs features corresponding to shadow prices determined in the context of flux variability analysis to predict metabolite-protein interactions"
- [methods] Source card workflow on 90% optimum and validity checks: [evidence span withheld — anchor status: not_found]
