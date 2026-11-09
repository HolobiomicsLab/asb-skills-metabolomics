---
name: matlab-cobra-model-manipulation
description: Use when draft genome-scale metabolic reconstructions must be loaded,
  identifier-normalized, solver-configured, and LP-solved in MATLAB with COBRA toolbox
  for gap filling or FBA.
when_to_use_negative:
- Input models are already in SBML and must be processed with Python-based tools (e.g.,
  CarveMe or SMETANA) rather than MATLAB COBRA
- The task requires whole-community optimization with community objective functions
  — use SteadyCom or MICOM instead of per-member COBRA manipulation
- No LP solver (CPLEX/Gurobi) is licensed or available; glpk is explicitly not recommended
  for this workflow
edam_operation: http://edamontology.org/operation_3660
edam_topics:
- http://edamontology.org/topic_2259
- http://edamontology.org/topic_0196
tools:
- name: COBRA toolbox
  role: MATLAB framework for loading, manipulating, and LP-solving metabolic models
    (FastGapFilling, FBA)
  repo: https://github.com/opencobra/cobratoolbox
- name: IBM CPLEX
  role: Preferred LP solver for COBRA toolbox gap-filling and FBA in this workflow
  repo: https://www.ibm.com/analytics/cplex-optimizer
- name: Gurobi
  role: Alternative LP solver tested for COBRA toolbox gap-filling
- name: HMMER
  role: Produces sequence-similarity E-values used as gap-filling weight support
- name: RAVEN 2.0
  role: Source of draft KEGG-based reconstructions loaded into MATLAB for consensus
    and gap filling
  repo: https://github.com/SysBioChalmers/RAVEN
- name: CarveMe
  role: Source of draft reconstructions loaded into MATLAB for consensus and gap filling
  repo: https://github.com/cdanielmachado/carveme
provenance:
  source_task_ids:
  - task_004
  - task_005
  source_papers:
  - doi: 10.1371/journal.pcbi.1009906
    title: 'COMMIT: Consideration of metabolite leakage and community composition
      improves microbial community reconstructions'
  source_content_hash: sha256:1c183276c723191672f58bfe612929afb95a6b5e13813cafd13c064e5bb7d48e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/opencobra/cobratoolbox
  tools:
  - COBRA toolbox
  - COMMIT
  - CarveMe
  - Gurobi
  - HMMER
  - IBM CPLEX
  - RAVEN 2.0
  edam_operation: http://edamontology.org/operation_3660
  edam_topics:
  - http://edamontology.org/topic_2259
  - http://edamontology.org/topic_0196
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 18
---

# matlab-cobra-model-manipulation

## Summary

Manipulate and simulate genome-scale metabolic models in MATLAB via the COBRA toolbox: load draft reconstructions (e.g., KBase/RAVEN/CarveMe/AuReMe drafts as Matlab .mat cell arrays), translate identifiers to MNXref, set solver (CPLEX/Gurobi), run FBA/FastGapFilling LPs, and export analysis-ready model objects for community-aware gap filling.

## When to use

Use when draft genome-scale metabolic reconstructions must be loaded, identifier-normalized, solver-configured, and LP-solved in MATLAB with COBRA toolbox for gap filling or FBA.

## When NOT to use

- Input models are already in SBML and must be processed with Python-based tools (e.g., CarveMe or SMETANA) rather than MATLAB COBRA
- The task requires whole-community optimization with community objective functions — use SteadyCom or MICOM instead of per-member COBRA manipulation
- No LP solver (CPLEX/Gurobi) is licensed or available; glpk is explicitly not recommended for this workflow

## Inputs

- Draft genome-scale metabolic reconstruction .mat files (Matlab cell arrays under data/models/_habitat_/_method_)
- MNXref-based universal gap-filling database (Universal-model-MNXref-balanced.mat)
- Community composition table (e.g., 16S rRNA abundance data from At-SPHERE)
- Minimal medium definition (intersection of KBase auxotrophic media, excluding amino acids and carbon sources except glucose)
- HMMER E-value similarity scores (cutoff 10^-6)

## Outputs

- Gap-filled MATLAB model objects ready for downstream SMETANA interaction analysis
- Sets of added reactions per ordering iteration (100 random member orderings)
- Exportable-metabolite lists per member (permeability-filtered: HBD≤5, HBA≤10, MW≤500 Da, TPSA≤140 Å, RB≤10, −0.4≤XLogP3≤5.6)
- Enlarged shared medium after each member gap fill

## How to apply

In MATLAB (tested with R2017b and R2020b), clone the COBRA toolbox with `git clone --depth=1 https://github.com/opencobra/cobratoolbox.git cobratoolbox` and run `initCobraToolbox`; configure an LP solver — IBM ILOG CPLEX v12.9 or Gurobi v9.1.1 (glpk is not recommended for this workflow). Load draft reconstruction .mat files (e.g., data/models/_habitat_/_method_ cell arrays for the Bulgarelli and Schlaeppi Arabidopsis thaliana soil communities) and the MNXref-based gap-filling database Universal-model-MNXref-balanced.mat. Before use, harmonize identifiers: replace 'MNXM01' with 'MNXM1' since both encode H+, and ignore generic compartments; exclude export/boundary reactions (metabolites in the '@BOUNDARY' compartment) and reactions with stoichiometric coefficients above 20 from the gap-filling database. Then invoke COBRA functions such as FastGapFilling with defined penalty weights (transport 100, metabolic non-transport 50, directionality change 25, permeable transport 50, sequence support 25 with HMMER E-value cutoff 10^-6, uptake of secreted metabolites 1, exchange 10^5) and biomass flux bounds of 10^-3 lower and 2.81 h^-1 upper limit per isolate model. Verify solver configuration and model consistency by checking that FBA yields biomass production before downstream gap-filling iterations.

## Related tools

- **COBRA toolbox** (MATLAB framework for loading, manipulating, and LP-solving metabolic models (FastGapFilling, FBA)) — https://github.com/opencobra/cobratoolbox
- **IBM CPLEX** (Preferred LP solver for COBRA toolbox gap-filling and FBA in this workflow) — https://www.ibm.com/analytics/cplex-optimizer
- **Gurobi** (Alternative LP solver tested for COBRA toolbox gap-filling)
- **HMMER** (Produces sequence-similarity E-values used as gap-filling weight support)
- **RAVEN 2.0** (Source of draft KEGG-based reconstructions loaded into MATLAB for consensus and gap filling) — https://github.com/SysBioChalmers/RAVEN
- **CarveMe** (Source of draft reconstructions loaded into MATLAB for consensus and gap filling) — https://github.com/cdanielmachado/carveme

## Examples

```
git clone --depth=1 https://github.com/opencobra/cobratoolbox.git cobratoolbox; cd cobratoolbox; matlab -r "initCobraToolbox; changeCobraSolver('ibm_cplex', 'LP')"
```

## Evaluation signals

- FBA on the loaded model produces nonzero biomass flux within the 10^-3 to 2.81 h^-1 bounds before gap filling is attempted
- MNXM01 has been fully replaced by MNXM1 so H+ identifiers are consistent across the model
- No '@BOUNDARY' compartment metabolites appear in the model's reaction list; no reaction has stoichiometric coefficients above 20
- FastGapFilling LP solves without error under CPLEX or Gurobi, and the total number of added reactions for the community-aware ordering is smaller than for individual gap filling
- Reproduced precision and recall on the D. vulgaris / M. maripaludis validation show the reported slight improvement of community-aware over individual gap filling

## Limitations

- Penalty weights for gap-filling reactions were tuned ad hoc: 'The penalties applied in this study resulted from the comparison of different weights that were altered until satisfying results were obtained for selected reconstructions'
- Comparison to reference models is hampered by lack of reaction cross-references to other namespaces; translations only done 'whenever possible'
- Only 26% of InChI keys could be translated to PubChem CIDs and only 11% had XLogP3 values, leaving many metabolites without properties and thus classified as not likely permeable
- glpk is not recommended as the LP solver for this workflow
- The full COMMIT procedure specification is deferred to supplementary S1 Text

## Evidence

- [readme] COBRA toolbox git clone and initCobraToolbox: [evidence span withheld — anchor status: not_found]
- [readme] COBRA toolbox requirement for COMMIT: "[COBRA toolbox v3.0](https://github.com/opencobra/cobratoolbox)"
- [readme] LP solver tested for COBRA toolbox in COMMIT: "LP solver for the COBRA toolbox, tested with - [IBM CPLEX v12.9](https://www.ibm.com/analytics/cplex-optimizer)"
- [abstract] COMMIT overview in abstract: "We then devise an approach for gap filling, termed COMMIT, that considers metabolites for secretion based on their permeability and the composition of the community."
- [discussion] Bias of glpk solver in article: "The medium that was used to create the models was, however, removed in a later step to start the gap-filling procedure with a minimal medium."
- [discussion] MNXM01 replacement rationale: "the metabolite ID ’MNXM01’ was replaced with ’MNXM1’ since both encode H+."
- [discussion] Penalty weights documented: "used specific weights for transport reactions (100), metabolic reactions (non-transport, 50), changing a reaction’s directionality (25)"
- [discussion] Exclusion of export reactions from gap-filling database: "export reactions i.e. reactions that include metabolites associated to the ’@BOUNDARY’ compartment were excluded from the gap-filling database"
- [discussion] HMMER E-value cutoff: "An E-value of 10−6 was used as a cut-off."
- [results] Validation community for gap-filling comparison: [evidence span withheld — anchor status: not_found]
- [results] Precision and recall defined for validation: [evidence span withheld — anchor status: not_found]
- [discussion] Boundary compartment exclusion rationale: "export reactions i.e. reactions that include metabolites associated to the ’@BOUNDARY’ compartment were excluded from the gap-filling database"
