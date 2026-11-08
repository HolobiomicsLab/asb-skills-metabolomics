---
name: community-aware-metabolic-modeling
description: Use when you must gap-fill draft genome-scale metabolic reconstructions
  for a multi-species community (e.g., D. vulgaris/M. maripaludis or Arabidopsis soil
  communities) and want to exploit cross-feeding/leakage to minimize the added reaction
  set while preserving genomic support.
when_to_use_negative:
- When you have a single isolated model with no community context — plain individual
  gap filling is the correct baseline, and COMMIT's leakage assumptions cannot be
  exploited
- When you cannot supply a valid LP solver (IBM CPLEX or Gurobi); glpk is explicitly
  not recommended and may fail the LP formulations
- When the community models use non-MNXref identifier spaces without translation —
  cross-referencing gaps hampered model comparison in the paper
edam_topics:
- http://edamontology.org/topic_2259
- http://edamontology.org/topic_3391
tools:
- name: COMMIT
  role: Performs the community-dependent, leakage-aware iterative gap filling; the
    core implementation of this skill
  repo: https://github.com/pwendering/COMMIT
- name: COBRA toolbox v3.0
  role: MATLAB framework hosting the gap-filling LP problems and FBA computations
  repo: https://github.com/opencobra/cobratoolbox
- name: IBM CPLEX v12.9 / Gurobi v9.1.1
  role: LP solvers required by the COBRA toolbox for the gap-filling optimization
- name: SMETANA
  role: 'Downstream analysis: computes cross-feeding interaction metrics from the
    gap-filled community models'
  repo: https://github.com/cdanielmachado/smetana
- name: KBase / RAVEN 2.0 / CarveMe / AuReMe / Pathway Tools
  role: Produce the four draft reconstructions that are merged into the consensus
    models before gap filling
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1371/journal.pcbi.1009906
    title: 'COMMIT: Consideration of metabolite leakage and community composition
      improves microbial community reconstructions'
  source_content_hash: sha256:3b9ee71ab07bdbcefdc980882fe775e5c732c3d8b52abddb7a9188cb848870a9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/pwendering/COMMIT
  tools:
  - COBRA toolbox v3.0
  - COMMIT
  - Gurobi
  - IBM CPLEX
  - IBM CPLEX v12.9 / Gurobi v9.1.1
  - KBase / RAVEN 2.0 / CarveMe / AuReMe / Pathway Tools
  - SMETANA
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_2259
  - http://edamontology.org/topic_3391
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9130434782608695
n_claims: 23
---

# community-aware-metabolic-modeling

## Summary

A constraint-based modeling skill that performs community-dependent gap filling of metabolic reconstructions (COMMIT) by considering metabolite leakage (permeability) and community composition, so that fewer reactions are added than when gap filling each member model individually.

## When to use

Use when you must gap-fill draft genome-scale metabolic reconstructions for a multi-species community (e.g., D. vulgaris/M. maripaludis or Arabidopsis soil communities) and want to exploit cross-feeding/leakage to minimize the added reaction set while preserving genomic support.

## When NOT to use

- When you have a single isolated model with no community context — plain individual gap filling is the correct baseline, and COMMIT's leakage assumptions cannot be exploited
- When you cannot supply a valid LP solver (IBM CPLEX or Gurobi); glpk is explicitly not recommended and may fail the LP formulations
- When the community models use non-MNXref identifier spaces without translation — cross-referencing gaps hampered model comparison in the paper

## Inputs

- Draft/consensus genome-scale metabolic reconstructions in SBML or COBRA MATLAB format (e.g., two-species D. vulgaris/M. maripaludis community or Arabidopsis soil community consensus models)
- MNXref-based universal gap-filling database (Universal-model-MNXref-balanced.mat)
- Minimal medium definition (intersection of all auxotrophic media, excluding amino acids and carbon sources except glucose)
- Metabolite molecular properties from PubChem (HBD, HBA, MW, TPSA, rotatable bonds, XLogP3)

## Outputs

- Community-aware gap-filled metabolic models (one per member, respecting community composition and leakage)
- Reaction recovery metrics table (reaction_recovery_precision_recall.csv) with precision and sensitivity per removal fraction (1/2/5/10%)
- SMETANA-based directed metabolic interaction predictions (helpers/beneficiaries roles)
- memote quality-assessment scores for the resulting functional models

## How to apply

Clone pwendering/COMMIT and load consensus reconstructions (e.g., Bulgarelli and Schlaeppi soil communities) together with the MNXref-based gap-filling database (Universal-model-MNXref-balanced.mat) into MATLAB with COBRA toolbox v3.0, using IBM CPLEX v12.9 or Gurobi v9.1.1 as the LP solver (glpk is not recommended). Run the iterative community-aware gap filling (code/matlab/gap_filling/run_iterative_gap_filling.m) with the paper's penalty weights: transport reactions 100, metabolic (non-transport) reactions 50, directionality change 25, permeable transport 50, exchange reactions 10^5, and uptake of metabolites already secreted by other community members given weight 1 — the enlarged medium from secreted metabolites reduces the reactions added to subsequent members. Classify metabolite permeability with the Lipinski-style rules HBD≤5, HBA≤10, MW≤500 Da, TPSA≤140 Å, RB≤10, −0.4≤XLogP3≤5.6 (unknown properties → not permeable), constrain each isolate's biomass flux to ≤2.81 h^-1, and use suboptimal biomass factor f=0.9. For validation, randomly remove 1%, 2%, 5%, and 10% of reactions (50 repetitions each), run COMMIT versus individual gap filling, and compute precision (added reactions matching removed) and recall/sensitivity (removed reactions recovered), reporting only these two metrics since true negatives cannot be determined.

## Related tools

- **COMMIT** (Performs the community-dependent, leakage-aware iterative gap filling; the core implementation of this skill) — https://github.com/pwendering/COMMIT
- **COBRA toolbox v3.0** (MATLAB framework hosting the gap-filling LP problems and FBA computations) — https://github.com/opencobra/cobratoolbox
- **IBM CPLEX v12.9 / Gurobi v9.1.1** (LP solvers required by the COBRA toolbox for the gap-filling optimization)
- **SMETANA** (Downstream analysis: computes cross-feeding interaction metrics from the gap-filled community models) — https://github.com/cdanielmachado/smetana
- **KBase / RAVEN 2.0 / CarveMe / AuReMe / Pathway Tools** (Produce the four draft reconstructions that are merged into the consensus models before gap filling)

## Examples

```
git clone https://github.com/pwendering/COMMIT && run code/matlab/gap_filling/run_iterative_gap_filling.m in MATLAB with COBRA toolbox v3.0 + IBM CPLEX v12.9
```

## Evaluation signals

- Community-aware gap filling should show slightly higher precision and recall than individual gap filling at every removal fraction (1%, 2%, 5%, 10%) over 50 repetitions
- COMMIT should add fewer reactions than individual gap filling without a loss of genomic support for the added reactions
- Only sensitivity and precision can be reported; any output claiming specificity/true negatives is invalid
- memote test suite should confirm the resulting models are functional (no blocked biomass)
- Every gap-filled reaction should be traceable to the MNXref universal database with the paper's weight classes (transport 100, metabolic 50, directionality 25, exchange 10^5)

## Limitations

- Penalty weights were tuned ad hoc ('altered until satisfying results were obtained for selected reconstructions'); no systematic weight-selection procedure is documented
- Only 11% of PubChem entries had XLogP3 values, so many metabolites default to 'not likely to be permeable', potentially underestimating leakage
- The full COMMIT procedure specification and definitions of the ordering criteria are deferred to supplementary S1 Text, not the main text
- True negatives cannot be determined in the removal-recovery validation, so specificity is not computable

## Evidence

- [abstract] COMMIT gap filling considers leakage and community composition: "an approach for gap filling, termed COMMIT, that considers metabolites for secretion based on their permeability and the composition of the community."
- [intro] COMMIT reduces the added reaction set: "COMMIT decreased the number of added reactions compared to the individual gap filling of the reconstructions"
- [discussion] Only precision and sensitivity are computable: "Since the true negatives could not be determined, only sensitivity and precision were calculated as measures to assess the quality of model prediction."
- [readme] Consensus models built from four reconstruction approaches: "The consensus reconstructions were generated by merging draft reconstructions from four different approaches:"
- [readme] Solver requirements for the MATLAB/COBRA implementation: "Matlab (tested with versions R2017b and R2020b) - [COBRA toolbox v3.0](https://github.com/opencobra/cobratoolbox) - LP solver for the COBRA toolbox, tested with"
