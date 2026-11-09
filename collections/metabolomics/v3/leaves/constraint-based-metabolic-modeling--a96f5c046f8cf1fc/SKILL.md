---
name: constraint-based-metabolic-modeling
description: Use when draft genome-scale metabolic reconstructions (SBML) must be
  made functional on a defined minimal medium via gap filling, then assessed with
  memote and used for FBA/community interaction analysis.
when_to_use_negative:
- When only a single isolated organism is studied and no community composition or
  metabolite leakage information is available — individual gap filling is the appropriate
  baseline instead of COMMIT
- 'When the reconstructions have not been mapped to a common identifier space (MNXref):
  identifier inconsistency between databases hampered comparison and would invalidate
  cross-model gap filling'
edam_operation: http://edamontology.org/operation_0438
edam_topics:
- http://edamontology.org/topic_0602
- http://edamontology.org/topic_0157
tools:
- name: COMMIT
  role: Performs conditional community-aware gap filling that respects community composition
    and metabolite permeability, minimizing added reactions
  repo: https://github.com/pwendering/COMMIT
- name: COBRA toolbox v3.0
  role: MATLAB framework for loading models (readCbModel), constraint-based simulation,
    and running the gap-filling LPs
  repo: https://github.com/opencobra/cobratoolbox
- name: IBM CPLEX / Gurobi
  role: LP solvers required by the COBRA toolbox for the gap-filling and FBA optimization
    problems
- name: CarveMe
  role: One of the four automated reconstruction approaches whose drafts are merged
    into the consensus models input to gap filling; its reconstruction medium must
    be removed beforehand
  repo: https://github.com/cdanielmachado/carveme
- name: RAVEN 2.0
  role: Automated reconstruction approach (KEGG-based, KO MSA pipeline) contributing
    drafts to the consensus models
  repo: https://github.com/SysBioChalmers/RAVEN
- name: SMETANA
  role: Downstream analysis of gap-filled community models to compute cross-feeding
    interaction metrics
  repo: https://github.com/cdanielmachado/smetana
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1371/journal.pcbi.1009906
    title: 'COMMIT: Consideration of metabolite leakage and community composition
      improves microbial community reconstructions'
  source_content_hash: sha256:7e7a66e8892e907931b4cfb944a6958ab37891abece1b70ec0ddc4d6576a8f0c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/pwendering/COMMIT
  tools:
  - COBRA toolbox v3.0
  - COMMIT
  - CarveMe
  - GLPK
  - GUROBI
  - Gurobi
  - IBM CPLEX
  - IBM CPLEX / Gurobi
  - INTEGRATE
  - MATLAB
  - Python
  - RAVEN 2.0
  - SARTRE
  - SMETANA
  edam_operation: http://edamontology.org/operation_0438
  edam_topics:
  - http://edamontology.org/topic_0602
  - http://edamontology.org/topic_0157
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8636363636363636
n_claims: 22
---

# constraint-based-metabolic-modeling

## Summary

Construct, gap-fill, and simulate genome-scale metabolic reconstructions under steady-state (COBRA-style) constraints, here applied to consensus community models of Arabidopsis soil isolates using the COBRA toolbox and COMMIT community-aware gap filling.

## When to use

Use when draft genome-scale metabolic reconstructions (SBML) must be made functional on a defined minimal medium via gap filling, then assessed with memote and used for FBA/community interaction analysis.

## When NOT to use

- When only a single isolated organism is studied and no community composition or metabolite leakage information is available — individual gap filling is the appropriate baseline instead of COMMIT
- When the reconstructions have not been mapped to a common identifier space (MNXref): identifier inconsistency between databases hampered comparison and would invalidate cross-model gap filling

## Inputs

- Draft or consensus genome-scale metabolic reconstructions in SBML/Matlab-COBRA format (e.g. data/models/consensus, Schlaeppi 24 isolates, Bulgarelli 20 isolates)
- Universal gap-filling database: Universal-model-MNXref-balanced
- MNXref metabolite/reaction identifier files and PubChem-derived molecular properties (HBD, HBA, MW, TPSA, RB, XLogP3) for permeability classification
- Defined minimal medium specification (D-glucose medium, intersection of KBase auxotrophic media)

## Outputs

- Gap-filled, functional community member models (SBML) with added-reaction sets per isolate
- Comparison of added-reaction set sizes between COMMIT (community-aware) and individual gap filling, with statistical test results
- memote quality metrics: overall score (~35%), consistency (~44%), GPR fraction (~90%)
- K-means (K=3) clusters of added-reaction sets visualized with pheatmap, igraph, and ape

## How to apply

Load draft or consensus reconstructions (e.g. the Schlaeppi 24-isolate and Bulgarelli 20-isolate consensus models) with readCbModel in MATLAB using COBRA toolbox v3.0 with an LP solver (Gurobi or IBM CPLEX v12.9). Set a D-glucose minimal medium derived from the intersection of KBase auxotrophic media, excluding amino acids and all carbon sources except glucose, and remove the medium used during CarveMe reconstruction before gap filling. Run COMMIT conditional gap filling for 100 iterations against the Universal-model-MNXref-balanced database, with permeability rules (HBD<=5, HBA<=10, MW<=500 Da, TPSA<=140 Angstrom, RB<=10, -0.4<=XLogP3<=5.6), reaction weights (transport 100, metabolic 50, directionality change 25, permeable transport 50, sequence support 25, uptake of secreted metabolites 1, exchange 10^5), and biomass flux limits of 2.81 h^-1 (upper) and 10^-3 (lower); process reconstructions sequentially in a chosen ordering so the enlarged medium reduces the added-reaction sets of subsequent members. As a baseline, gap-fill each reconstruction individually without community composition and compare the sizes of the added-reaction sets. Assess gap-filled functional models with the memote test suite and optionally probe cross-feeding with SMETANA; note that all metabolite identifiers are in MNXref space, with MNXM01 replaced by MNXM1 (both encode H+).

## Related tools

- **COMMIT** (Performs conditional community-aware gap filling that respects community composition and metabolite permeability, minimizing added reactions) — https://github.com/pwendering/COMMIT
- **COBRA toolbox v3.0** (MATLAB framework for loading models (readCbModel), constraint-based simulation, and running the gap-filling LPs) — https://github.com/opencobra/cobratoolbox
- **IBM CPLEX / Gurobi** (LP solvers required by the COBRA toolbox for the gap-filling and FBA optimization problems)
- **CarveMe** (One of the four automated reconstruction approaches whose drafts are merged into the consensus models input to gap filling; its reconstruction medium must be removed beforehand) — https://github.com/cdanielmachado/carveme
- **RAVEN 2.0** (Automated reconstruction approach (KEGG-based, KO MSA pipeline) contributing drafts to the consensus models) — https://github.com/SysBioChalmers/RAVEN
- **SMETANA** (Downstream analysis of gap-filled community models to compute cross-feeding interaction metrics) — https://github.com/cdanielmachado/smetana

## Examples

```
git clone https://github.com/pwendering/COMMIT; cd COMMIT; # configure code/matlab/options.m, then run run_iterative_gap_filling.m with 100 iterations against Universal-model-MNXref-balanced
```

## Evaluation signals

- COMMIT added-reaction sets are significantly smaller than those from individual gap filling of the same reconstructions, while genomic support is unchanged
- memote score of gap-filled models is ~35%, consistency ~44%, and GPR fraction ~90%
- Biomass flux constraints hold during gap filling: upper limit 2.81 h^-1 and lower limit 10^-3
- All added reactions are selected from Universal-model-MNXref-balanced, with export/boundary reactions and reactions with stoichiometric coefficients above 20 excluded from the gap-filling database
- Cluster structure of added-reaction sets separates by reconstruction type and gap-filling method at K=3 in K-means

## Limitations

- Gap-filling penalty weights (transport 100, metabolic 50, etc.) were tuned ad hoc until satisfying results for selected reconstructions, with no systematic selection procedure documented
- Only 26% of InChI keys could be translated to PubChem CIDs and only 11% had XLogP3 values, so many metabolites were conservatively classified as not likely permeable
- Criteria for choosing the 'best ordering' in COMMIT are referenced but not fully specified in the main text; the full procedure is deferred to supplementary S1 Text
- Comparison of models to reference reconstructions is hampered by lack of reaction cross-references to other identifier name spaces, so only sensitivity and precision could be computed

## Evidence

- [intro] community-aware gap filling yields smaller solutions: "COMMIT decreased the number of added reactions compared to the individual gap filling of the reconstructions"
- [abstract] consensus of automatically generated reconstructions improves draft quality: "the consensus of automatically generated metabolic reconstructions improves the quality of the draft reconstructions, measured by comparison to reference models"
- [discussion] permeability rules for leakage-aware gap filling: "the applied rules were: HBD≤5,HBA≤10, MW≤500 Da, TPSA≤140 Å, RB≤10, −0.4≤XLogP3≤5.6."
- [discussion] biomass flux constraints in gap-filling LP: "The upper limit for the biomass flux for each isolate model was constrained to 2.81 h−1"
- [readme] repository scope and community inputs: "This repository contains all files used for community-dependent gap-filling and subsequent analysis of consensus genome-scale metabolic reconstructions for two soil communities (Bulgarelli et al."
- [readme] solver requirement for COBRA-based gap filling: "LP solver for the COBRA toolbox, tested with - [IBM CPLEX v12.9](https://www.ibm.com/analytics/cplex-optimizer)"
