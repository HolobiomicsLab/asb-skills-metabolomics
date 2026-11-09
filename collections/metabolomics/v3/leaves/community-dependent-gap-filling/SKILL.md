---
name: community-dependent-gap-filling
description: Use when a set of community-member GEMs (e.g. consensus reconstructions
  from KBase/CarveMe/RAVEN/AuReMe) must be gap-filled together under a defined minimal
  medium and you want to minimize the number of added reactions while preserving genomic
  support.
when_to_use_negative:
- When a single isolate model is being gap-filled in isolation and community composition/leakage
  is irrelevant — individual gap filling is the appropriate baseline.
- When community members already grow on the defined medium and no gap filling is
  needed — adding reactions would only inflate the solution.
- When you need community-level steady-state abundance optimization rather than gap
  filling — SteadyCom or MICOM-style whole-community optimization addresses that instead.
edam_operation: http://edamontology.org/operation_3660
edam_topics:
- http://edamontology.org/topic_0602
tools:
- name: COMMIT
  role: Conditional, community-composition- and permeability-aware gap-filling algorithm
    executed iteratively over the member models
  repo: https://github.com/pwendering/COMMIT
- name: COBRA toolbox
  role: MATLAB framework loading models via readCbModel and running the LP-based gap
    filling; requires an LP solver (IBM CPLEX v12.9 or Gurobi)
  repo: https://github.com/opencobra/cobratoolbox
- name: Gurobi
  role: LP solver used by the COBRA toolbox for the gap-filling optimization
- name: IBM CPLEX
  role: Alternative LP solver (v12.9) tested with the COBRA toolbox for gap filling
- name: memote
  role: Test suite used to score the gap-filled functional models (score, consistency,
    GPR fraction)
- name: CarveMe
  role: Source of draft reconstructions whose reconstruction-time medium must be removed
    before gap filling starts from minimal medium
  repo: https://github.com/cdanielmachado/carveme
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1371/journal.pcbi.1009906
    title: 'COMMIT: Consideration of metabolite leakage and community composition
      improves microbial community reconstructions'
  source_content_hash: sha256:06bcef3f19f7086de9ebbf94bfe9cf77d157d2fd4f2e4e04bba1130f273df3d8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/pwendering/COMMIT
  tools:
  - COBRA toolbox
  - COMMIT
  - CarveMe
  - Gurobi
  - IBM CPLEX
  - memote
  edam_operation: http://edamontology.org/operation_3660
  edam_topics:
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6875
n_claims: 16
---

# community-dependent-gap-filling

## Summary

COMMIT-based community-dependent gap filling: conditional, iterative gap filling of consensus genome-scale metabolic reconstructions that respects microbial community composition and metabolite permeability, so that added-reaction sets are smaller than those from gap filling each member individually.

## When to use

Use when a set of community-member GEMs (e.g. consensus reconstructions from KBase/CarveMe/RAVEN/AuReMe) must be gap-filled together under a defined minimal medium and you want to minimize the number of added reactions while preserving genomic support.

## When NOT to use

- When a single isolate model is being gap-filled in isolation and community composition/leakage is irrelevant — individual gap filling is the appropriate baseline.
- When community members already grow on the defined medium and no gap filling is needed — adding reactions would only inflate the solution.
- When you need community-level steady-state abundance optimization rather than gap filling — SteadyCom or MICOM-style whole-community optimization addresses that instead.

## Inputs

- Consensus genome-scale metabolic reconstructions for community members (SBML, e.g. Schlaeppi 24-isolate and Bulgarelli 20-isolate sets from data/models/consensus)
- Universal-model-MNXref-balanced gap-filling database (MNXref identifiers; MNXM01 replaced by MNXM1)
- Defined D-glucose minimal medium (intersection of KBase auxotrophic media, excluding amino acids and non-glucose carbon sources)
- Metabolite molecular properties from PubChem (HBD, HBA, MW, TPSA, RB, XLogP3) for permeability classification
- Sequence-similarity evidence (HMMER hits against KO sequences, E-value <= 10^-6) for reaction weights

## Outputs

- Gap-filled, biomass-producing functional community-member models per ordering iteration
- Added-reaction sets per member and method (COMMIT vs individual), with reaction categories and genomic support annotations
- memote quality reports (score ~35%, consistency ~44%, GPR fraction ~90%)
- K-means (K=3) clustering and heatmaps/graphs of added-reaction sets (pheatmap, igraph, ape)

## How to apply

Configure paths in code/matlab/options.m, load the Schlaeppi (24 isolates) and Bulgarelli (20 isolates) consensus reconstructions from data/models/consensus with COBRA toolbox readCbModel, and define the D-glucose minimal medium (intersection of KBase auxotrophic media excluding amino acids and all carbon sources except glucose), removing the medium used during CarveMe reconstruction. Run COMMIT conditional gap filling via code/matlab/gap_filling/run_iterative_gap_filling.m with 100 iterations against the Universal-model-MNXref-balanced database, applying permeability rules (HBD<=5, HBA<=10, MW<=500 Da, TPSA<=140 Angstrom, RB<=10, -0.4<=XLogP3<=5.6, unclassified metabolites treated as not permeable), reaction weights (transport 100, metabolic 50, directionality change 25, permeable transport 50, sequence support 25 with HMMER E-value cut-off 10^-6, uptake of secreted metabolites 1, exchange 10^5), and biomass flux limits of 2.81 h^-1 (upper) and 10^-3 (lower). Gap filling is ordered across community members so each earlier member's secreted metabolites enlarge the medium for later members, which is the rationale for smaller added-reaction sets; exclude export/boundary reactions and reactions with stoichiometric coefficients above 20 from the gap-filling database. As a control, run individual gap filling (gap_fill_individual_models.m) on the same reconstructions without community composition, then statistically test that COMMIT yields significantly smaller added-reaction sets without changing genomic support. Assess the resulting functional models with the memote test suite (expected score ~35%, consistency ~44%, GPR fraction ~90%) and cluster added-reaction sets by reconstruction type and gap-filling method with K-means (K=3) using R's pheatmap, igraph, and ape.

## Related tools

- **COMMIT** (Conditional, community-composition- and permeability-aware gap-filling algorithm executed iteratively over the member models) — https://github.com/pwendering/COMMIT
- **COBRA toolbox** (MATLAB framework loading models via readCbModel and running the LP-based gap filling; requires an LP solver (IBM CPLEX v12.9 or Gurobi)) — https://github.com/opencobra/cobratoolbox
- **Gurobi** (LP solver used by the COBRA toolbox for the gap-filling optimization)
- **IBM CPLEX** (Alternative LP solver (v12.9) tested with the COBRA toolbox for gap filling)
- **memote** (Test suite used to score the gap-filled functional models (score, consistency, GPR fraction))
- **CarveMe** (Source of draft reconstructions whose reconstruction-time medium must be removed before gap filling starts from minimal medium) — https://github.com/cdanielmachado/carveme

## Examples

```
git clone https://github.com/pwendering/COMMIT && run code/matlab/gap_filling/run_iterative_gap_filling.m from MATLAB after configuring code/matlab/options.m
```

## Evaluation signals

- COMMIT added-reaction sets are significantly smaller than those from individual gap filling of the same reconstructions (same algorithm), while genomic support of added reactions is unchanged
- memote score of gap-filled functional models is ~35%, consistency ~44%, and GPR fraction ~90%
- Gap-filled members produce biomass within the constrained flux window (lower limit 10^-3, upper limit 2.81 h^-1)
- Only non-boundary reactions with stoichiometric coefficients <= 20 appear in the added sets (export/'@BOUNDARY' reactions were excluded from the gap-filling database)
- K-means clustering (K=3) of added-reaction sets separates solutions by reconstruction type and gap-filling method

## Limitations

- Gap-filling penalty weights were tuned ad hoc ('altered until satisfying results were obtained for selected reconstructions'); no systematic weight-selection procedure is documented.
- The full COMMIT procedure specification is deferred to supplementary S1 Text, and criteria for choosing the best member ordering are referenced but not specified in the text.
- Only 11% of PubChem compounds had XLogP3 values and 26% of InChI keys translated to CIDs, so many metabolites must be classified as not likely permeable by default.

## Evidence

- [intro] COMMIT decreased the number of added reactions compared to the individual gap filling of the reconstructions: "COMMIT decreased the number of added reactions compared to the individual gap filling of the reconstructions"
- [discussion] Reaction weights for gap filling: "used specific weights for transport reactions (100), metabolic reactions (non-transport, 50), changing a reaction’s directionality (25)"
- [discussion] Biomass flux constraint: "The upper limit for the biomass flux for each isolate model was constrained to 2.81 h−1"
- [discussion] Permeability rules: "the applied rules were: HBD≤5,HBA≤10, MW≤500 Da, TPSA≤140 Å, RB≤10, −0.4≤XLogP3≤5.6."
- [readme] Repository scope: "This repository contains all files used for community-dependent gap-filling and subsequent analysis of consensus genome-scale metabolic reconstructions for two soil communities"
- [methods] Repository acquisition: "git clone https://github.com/pwendering/COMMIT"
