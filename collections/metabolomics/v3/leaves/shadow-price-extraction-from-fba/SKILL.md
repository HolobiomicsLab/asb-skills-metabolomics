---
name: shadow-price-extraction-from-fba
description: Use when metabolite features for machine learning must be derived from
  constraint-based modeling of a GEM (e.g., iJO1366 or Yeast-GEM), specifically when
  shadow prices under near-optimal growth (90% of maximum biomass) are needed per
  reaction.
when_to_use_negative:
- The metabolite-side features are already available (e.g., precomputed shadow prices
  or feature tables), so no constraint-based solving is needed
- A 128-bit molecular fingerprint representation of metabolites is required instead
  — use generate_fp.py with RDKit/Bio.Entrez rather than shadow-price extraction
- The GEM cannot be constrained to 90% of optimal biomass or GUROBI licensing is unavailable,
  since the dual formulation depends on both
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_2259
tools:
- name: SARTRE
  role: Framework whose metabolite features are the shadow prices extracted here,
    combined with protein-reaction association vectors from GPR rules
  repo: https://github.com/fayazsoleymani/SARTRE
- name: COBRAToolbox
  role: Used in MATLAB to curate iJO1366.mat and Yeast-GEM.mat into the irreversible
    90%-optimal models (ModelIrrevOpt90.mat, yeast_IrrevOpt90.mat) that serve as inputs
- name: GUROBI
  role: Optimizer, accessed via its Python interface, that solves the FBA dual formulation
    to compute shadow prices per reaction
  repo: https://www.gurobi.com/
- name: Python
  role: Language in which optimization.py, cleaning.py and the validity-check pipeline
    are implemented
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioadv/vbad098
    title: Prediction of metabolite–protein interactions based on integration of machine
      learning and constraint-based modeling
  source_content_hash: sha256:114226f01b92ad1b74dbe737a98a5b38d73de68059b43ab47df211e5dd03fb43
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
  edam_operation: http://edamontology.org/operation_3927
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_2259
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7857142857142857
n_claims: 14
---

# shadow-price-extraction-from-fba

## Summary

Extracts metabolite-side features for supervised metabolite-protein interaction prediction (SARTRE) by solving the FBA dual problem with GUROBI over curated genome-scale models constrained to 90% of optimal biomass, retaining only shadow prices that pass allowable increase/decrease and incremental/decremental (epsilon=1e-5) validity checks.

## When to use

Use when metabolite features for machine learning must be derived from constraint-based modeling of a GEM (e.g., iJO1366 or Yeast-GEM), specifically when shadow prices under near-optimal growth (90% of maximum biomass) are needed per reaction.

## When NOT to use

- The metabolite-side features are already available (e.g., precomputed shadow prices or feature tables), so no constraint-based solving is needed
- A 128-bit molecular fingerprint representation of metabolites is required instead — use generate_fp.py with RDKit/Bio.Entrez rather than shadow-price extraction
- The GEM cannot be constrained to 90% of optimal biomass or GUROBI licensing is unavailable, since the dual formulation depends on both

## Inputs

- iJO1366.mat (E. coli genome-scale metabolic model)
- Yeast-GEM.mat (S. cerevisiae consensus model Yeast8)
- Gold-standard metabolite-protein interaction datasets: piazza, reznik, stitch_ecoli, stitch_yeast
- GPR rules from the curated irreversible models (for the protein-reaction association matrix)

## Outputs

- ModelIrrevOpt90.mat and yeast_IrrevOpt90.mat (90%-optimal irreversible curated models)
- Shadow-price optimization results in the opt directory, one result per reaction indexed by reaction number (also provided as .zip archives)
- Validated shadow-price feature vectors for downstream classifiers

## How to apply

First curate the GEMs with COBRAToolbox in MATLAB: run modelPreparing_iJO1366.m on iJO1366.mat to produce ModelIrrevOpt90.mat and modelPreparing_yeastGEM.m on Yeast-GEM.mat to produce yeast_IrrevOpt90.mat, yielding irreversible models fixed at 90% optimal biomass. Parse the GPR rules of each curated model to build the protein-reaction association matrix that maps reaction-level shadow prices onto proteins. Then, in each gold-standard directory (piazza, reznik, stitch_ecoli, stitch_yeast), run optimization.py to solve the FBA/FVA dual formulation via the GUROBI Python interface (an active GUROBI license is required); results are saved in the opt directory, one result per reaction indexed by its reaction number. Finally apply the validity conditions — allowable increase/decrease ranges plus incremental/decremental shadow-price agreement with epsilon=1e-5 — so only valid shadow prices are kept for downstream classifiers (cleaning.py handles preprocessing).

## Related tools

- **SARTRE** (Framework whose metabolite features are the shadow prices extracted here, combined with protein-reaction association vectors from GPR rules) — https://github.com/fayazsoleymani/SARTRE
- **COBRAToolbox** (Used in MATLAB to curate iJO1366.mat and Yeast-GEM.mat into the irreversible 90%-optimal models (ModelIrrevOpt90.mat, yeast_IrrevOpt90.mat) that serve as inputs)
- **GUROBI** (Optimizer, accessed via its Python interface, that solves the FBA dual formulation to compute shadow prices per reaction) — https://www.gurobi.com/
- **Python** (Language in which optimization.py, cleaning.py and the validity-check pipeline are implemented)

## Examples

```
cd piazza && python3 optimization.py
```

## Evaluation signals

- Optimization results exist in the opt directory for every reaction, saved per reaction number (and matching the provided .zip archives)
- Curated models are exactly the 90%-optimal irreversible versions produced by the modelPreparing scripts (ModelIrrevOpt90.mat, yeast_IrrevOpt90.mat)
- Each retained shadow price passes both the allowable increase/decrease check and the incremental/decremental agreement check with epsilon=1e-5
- Extracted shadow prices are combined with protein-reaction association vectors derived from the parsed GPR rules without unmatched proteins
- Downstream random forest classifiers trained on these features reach expected discrimination (SARTRE reports average AUC 0.86 for E. coli and 0.85 for S. cerevisiae)

## Limitations

- Requires an active GUROBI license before running optimization.py
- Archives with precomputed optimization results must be unzipped before use
- Shadow prices are only valid under the 90%-optimal irreversible model curation; results depend on media composition, as shown by comparisons across acetate, fructose, glycerol, mannose, and succinate carbon sources vs. glucose

## Evidence

- [methods] 90%-optimal curation: "`modelPreparing_iJO1366.m` uses `iJO1366.mat` as the input and creates `ModelIrrevOpt90.mat`"
- [methods] dual optimization via GUROBI: "The optimization process has been implemented by [GUROBI](https://www.gurobi.com/) optimizer and its python interface"
- [readme] outputs per reaction: "The results of optimization will be saved on the `opt` directory for each reaction by its number."
- [full_text] validity checks and feature combination: [evidence span withheld — anchor status: not_found]
