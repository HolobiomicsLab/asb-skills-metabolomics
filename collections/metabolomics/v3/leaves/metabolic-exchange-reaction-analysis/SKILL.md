---
name: metabolic-exchange-reaction-analysis
description: Use when gap-filled community GEMs (e.g., Matlab workspaces from run_iterative_gap_filling.m)
  are available and the goal is to map which metabolites members secrete/uptake and
  derive helper/beneficiary dependency calls.
when_to_use_negative:
- Input models are still draft (pre-gap-filled) reconstructions without a complete
  exchange-reaction structure, so uptake-blocking FBA would produce artifacts rather
  than real dependencies
- The goal is to predict which metabolites are likely permeable/secreted for gap filling
  (COMMIT's leakage model) rather than to quantify community-wide uptake dependencies
- The analysis is single-isolate (no community composition), so there is no cross-feeding
  pool to attribute dependencies to
edam_operation: http://edamontology.org/operation_3660
edam_topics:
- http://edamontology.org/topic_0605
- http://edamontology.org/topic_3174
tools:
- name: COBRA toolbox v3.0
  role: Matlab framework for loading community models, sink-reaction maximization,
    and FBA LPs with blocked uptake (solved with CPLEX or Gurobi)
  repo: https://github.com/opencobra/cobratoolbox
- name: SMETANA
  role: Downstream tool used to predict directed metabolic interactions from the gap-filled
    models, complementing the uptake-blocking dependency analysis
  repo: https://github.com/cdanielmachado/smetana
- name: pheatmap
  role: R package for rendering the member-by-uptake biomass-reduction matrix as a
    clustered heatmap
- name: COMMIT
  role: Upstream pipeline that produces the community-dependent gap-filled models
    whose exchange reactions are analyzed here
  repo: https://github.com/pwendering/COMMIT
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1371/journal.pcbi.1009906
    title: 'COMMIT: Consideration of metabolite leakage and community composition
      improves microbial community reconstructions'
  source_content_hash: sha256:4ed400b3188427a704ede865907127f4ef709133a3eb6d67c2a01deec203903b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/opencobra/cobratoolbox
  tools:
  - COBRA toolbox v3.0
  - COMMIT
  - SMETANA
  - pheatmap
  edam_operation: http://edamontology.org/operation_3660
  edam_topics:
  - http://edamontology.org/topic_0605
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# metabolic-exchange-reaction-analysis

## Summary

Enumerates extracellular exchange/uptake reactions in gap-filled community genome-scale metabolic models and quantifies cross-feeding dependencies by sink-reaction maximization and uptake-blocking FBA, as applied to the Bulgarelli and Schlaeppi Arabidopsis soil communities.

## When to use

Use when gap-filled community GEMs (e.g., Matlab workspaces from run_iterative_gap_filling.m) are available and the goal is to map which metabolites members secrete/uptake and derive helper/beneficiary dependency calls.

## When NOT to use

- Input models are still draft (pre-gap-filled) reconstructions without a complete exchange-reaction structure, so uptake-blocking FBA would produce artifacts rather than real dependencies
- The goal is to predict which metabolites are likely permeable/secreted for gap filling (COMMIT's leakage model) rather than to quantify community-wide uptake dependencies
- The analysis is single-isolate (no community composition), so there is no cross-feeding pool to attribute dependencies to

## Inputs

- Gap-filled community model workspaces (.mat) from run_iterative_gap_filling.m for the Bulgarelli and Schlaeppi communities (COBRA toolbox format)
- SBML genome-scale metabolic models from the consensus reconstruction pipeline (KBase, RAVEN 2.0, CarveMe, AuReMe/Pathway Tools)
- Extracellular exchange/uptake reaction list derived from the community models

## Outputs

- Member-by-uptake-reaction matrix of biomass flux reductions after uptake blocking
- Clustered heatmap (pheatmap, wesanderson/scales palettes) of uptake dependencies
- Secreted-metabolite contribution table per member from sink-reaction maximization at f = 0.9
- Helper/beneficiary community-role calls and mineral dependency calls (Fe2+, Fe3+, SO4^2-)

## How to apply

Load the gap-filled community models for the Bulgarelli and Schlaeppi soil communities (saved as Matlab workspaces) into COBRA toolbox v3.0 with an LP solver (CPLEX or Gurobi). Enumerate the extracellular exchange/uptake reactions and, for each member, compute maximal secretion/leakage flux contribution to the exchanged-metabolite pool using sink-reaction maximization at suboptimal biomass factor f = 0.9. For every uptake reaction, run FBA with that uptake blocked and record the biomass flux reduction for each community member. Cross-check computed dependencies against the reported mineral dependencies (Fe2+, Fe3+, SO4^2-) and the community-specific amino-acid and within-family exchange patterns. Render the member-by-uptake biomass-reduction matrix as a clustered heatmap in R using pheatmap (wesanderson/scales palettes), and validate that helper/beneficiary roles match the published interaction pattern.

## Related tools

- **COBRA toolbox v3.0** (Matlab framework for loading community models, sink-reaction maximization, and FBA LPs with blocked uptake (solved with CPLEX or Gurobi)) — https://github.com/opencobra/cobratoolbox
- **SMETANA** (Downstream tool used to predict directed metabolic interactions from the gap-filled models, complementing the uptake-blocking dependency analysis) — https://github.com/cdanielmachado/smetana
- **pheatmap** (R package for rendering the member-by-uptake biomass-reduction matrix as a clustered heatmap)
- **COMMIT** (Upstream pipeline that produces the community-dependent gap-filled models whose exchange reactions are analyzed here) — https://github.com/pwendering/COMMIT

## Evaluation signals

- The reproduced pheatmap of biomass-reduction values matches the published helper/beneficiary interaction pattern for both communities
- Dependency calls for Fe2+, Fe3+, and SO4^2- are recovered in models across most bacterial families, and these ions are also secreted by most families, confirming mineral exchange as a core interaction
- Sink-reaction maximization was run with suboptimal biomass factor f = 0.9, matching the reported parameter
- Every blocked-uptake FBA returns a solver-feasible (optimal) solution with the biomass flux reduction recorded per member
- Computed dependencies cross-check against community-specific amino-acid and specialized within-family exchange patterns

## Limitations

- Only sensitivity and precision could be computed as quality measures because true negatives could not be determined
- Comparison to reference models was hampered by lack of reaction cross-references to other name spaces; translations were only done whenever possible
- The full COMMIT procedure specification and distance-measure definitions are deferred to supplementary S1 Text, not the main document
- 16S rRNA abundance data were only available via personal provision (from Ruben Garrido Oter), not a public accession

## Evidence

- [abstract] Inspection of the metabolic interactions in the soil communities allows us to identify microbes with community roles of helpers and beneficiaries: "Inspection of the metabolic interactions in the soil communities allows us to identify microbes with community roles of helpers and beneficiaries"
- [discussion] In this study, f = 0.9 has been used (please see S1 Text for more detail).: "In this study, f = 0.9 has been used (please see S1 Text for more detail)."
- [readme] This repository contains all files used for community-dependent gap-filling and subsequent analysis of consensus genome-scale metabolic reconstructions: "This repository contains all files used for community-dependent gap-filling and subsequent analysis of consensus genome-scale metabolic reconstructions for two soil communities"
- [other] Most models across all bacterial families show putative dependencies for Fe2+, Fe3+, and SO42-: [evidence span withheld — anchor status: not_found]
