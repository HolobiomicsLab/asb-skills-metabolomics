---
name: biomass-reduction-heatmap-visualization
description: Use when gap-filled community GEMs (e.g. COBRA workspaces from run_iterative_gap_filling.m)
  are available and you need to identify which exchanged metabolites each member depends
  on and visualize the resulting dependency matrix.
when_to_use_negative:
- When the goal is to predict directed cross-feeding interactions from the models
  via a dedicated metric rather than knock-out-based dependency profiling (use SMETANA
  instead)
- When analyzing a single isolate model with no community context, since the matrix
  of member-by-uptake dependencies and its helper/beneficiary interpretation only
  exists across community members
- When the input is raw FBA growth rates rather than paired baseline vs uptake-blocked
  fluxes; the biomass-reduction values must first be computed from blocked-uptake
  simulations
edam_operation: http://edamontology.org/operation_0570
edam_topics:
- http://edamontology.org/topic_0602
- http://edamontology.org/topic_0091
tools:
- name: COBRA toolbox
  role: Runs FBA with individual uptake reactions blocked to compute biomass flux
    reductions; LP solved with CPLEX or Gurobi
  repo: https://github.com/opencobra/cobratoolbox
- name: pheatmap
  role: R package used to render the member-by-uptake-reaction biomass-reduction matrix
    as a clustered heatmap with wesanderson/scales palettes
- name: COMMIT
  role: Produces the gap-filled community models whose exchange reactions form the
    substrate of the dependency analysis
  repo: https://github.com/pwendering/COMMIT
- name: SMETANA
  role: Complementary tool for downstream prediction of directed metabolic interactions
    in the gap-filled models
  repo: https://github.com/cdanielmachado/smetana
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1371/journal.pcbi.1009906
    title: 'COMMIT: Consideration of metabolite leakage and community composition
      improves microbial community reconstructions'
  source_content_hash: sha256:92d4a7ed69692822e9b004e07ab41a371f4839aee6dc36642c1d7c67ac2eb8ff
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/opencobra/cobratoolbox
  tools:
  - COBRA toolbox
  - COMMIT
  - SMETANA
  - pheatmap
  edam_operation: http://edamontology.org/operation_0570
  edam_topics:
  - http://edamontology.org/topic_0602
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# biomass-reduction-heatmap-visualization

## Summary

Quantifies how strongly each community member depends on each exchanged metabolite by running FBA with every uptake reaction blocked and recording the biomass flux reduction, then renders the member-by-reaction reduction matrix as a clustered R pheatmap heatmap to reveal helper/beneficiary structure (e.g. Fe2+, Fe3+ and SO4^2- dependencies) in the Bulgarelli and Schlaeppi soil communities.

## When to use

Use when gap-filled community GEMs (e.g. COBRA workspaces from run_iterative_gap_filling.m) are available and you need to identify which exchanged metabolites each member depends on and visualize the resulting dependency matrix.

## When NOT to use

- When the goal is to predict directed cross-feeding interactions from the models via a dedicated metric rather than knock-out-based dependency profiling (use SMETANA instead)
- When analyzing a single isolate model with no community context, since the matrix of member-by-uptake dependencies and its helper/beneficiary interpretation only exists across community members
- When the input is raw FBA growth rates rather than paired baseline vs uptake-blocked fluxes; the biomass-reduction values must first be computed from blocked-uptake simulations

## Inputs

- Gap-filled community model workspaces for the Bulgarelli and Schlaeppi soil communities (Matlab workspaces, output of run_iterative_gap_filling.m, loaded into COBRA toolbox v3.0)
- List of extracellular exchange/uptake reactions per community member
- Solver configuration (IBM CPLEX v12.9 or Gurobi) for COBRA LP problems

## Outputs

- Member-by-uptake-reaction biomass flux reduction matrix (numeric matrix of growth-rate losses after each uptake block)
- Clustered heatmap of the biomass-reduction matrix produced with R pheatmap (wesanderson/scales palettes)
- Calls of helper/beneficiary dependencies per member, including mineral dependencies (Fe2+, Fe3+, SO4^2-)

## How to apply

Load the gap-filled community models for the Bulgarelli and Schlaeppi soil communities into the COBRA toolbox, enumerate the extracellular exchange/uptake reactions, and compute each member's maximal secretion/leakage contribution to the exchanged-metabolite pool using sink-reaction maximization with suboptimal biomass factor f = 0.9. For each uptake reaction, run FBA with that single uptake blocked (LP solved with CPLEX or Gurobi) and record the biomass flux reduction for every community member. Assemble these values into a member-by-uptake-reaction biomass-reduction matrix and visualize it as a clustered heatmap with the R package pheatmap using wesanderson/scales color palettes. Cross-check the computed dependencies against the reported mineral dependencies (Fe2+, Fe3+, SO4^2-) and community-specific amino-acid and within-family exchange patterns; the reproduced heatmap must match the published helper/beneficiary interaction pattern.

## Related tools

- **COBRA toolbox** (Runs FBA with individual uptake reactions blocked to compute biomass flux reductions; LP solved with CPLEX or Gurobi) — https://github.com/opencobra/cobratoolbox
- **pheatmap** (R package used to render the member-by-uptake-reaction biomass-reduction matrix as a clustered heatmap with wesanderson/scales palettes)
- **COMMIT** (Produces the gap-filled community models whose exchange reactions form the substrate of the dependency analysis) — https://github.com/pwendering/COMMIT
- **SMETANA** (Complementary tool for downstream prediction of directed metabolic interactions in the gap-filled models) — https://github.com/cdanielmachado/smetana

## Examples

```
Rscript -e 'library(pheatmap); pheatmap(biomass_reduction_matrix, cluster_rows=TRUE, cluster_cols=TRUE, color=wes_palette("Zissou1", 100, "continuous"))'
```

## Evaluation signals

- Reproduced heatmap matches the published helper/beneficiary interaction pattern for both the Bulgarelli and Schlaeppi communities
- Computed uptake dependencies reproduce the reported mineral dependencies (Fe2+, Fe3+, SO4^2- provided by and required from members of most bacterial families)
- Community-specific amino-acid and specialized within-family exchange patterns are recovered in the dependency matrix
- Every biomass-reduction value traces back to a completed FBA run with a single uptake reaction blocked; unblocked-control biomass should match the baseline solution
- Sink-reaction maximization consistently uses suboptimal biomass factor f = 0.9 across all members

## Limitations

- The full COMMIT procedure specification is deferred to supplementary S1 Text and is not present in the main document
- Gap-filling penalty weights underlying the models were tuned ad hoc until satisfying results, with no systematic selection procedure documented
- Only sensitivity and precision could be computed for model quality since true negatives could not be determined
- Comparison to reference models is hampered by lack of reaction cross-references to other name spaces; identifier translation was only done 'whenever possible'

## Evidence

- [methods] Visualize the member-by-uptake-reaction biomass-reduction matrix as a clustered heatmap with the R package pheatmap (wesanderson/scales palettes): [evidence span withheld — anchor status: not_found]
- [methods] For each uptake reaction, run FBA with that uptake blocked (COBRA toolbox LP solved with CPLEX or Gurobi) and record the resulting biomass flux reduction for every community member.: [evidence span withheld — anchor status: not_found]
- [discussion] In this study, f = 0.9 has been used (please see S1 Text for more detail).: "In this study, f = 0.9 has been used (please see S1 Text for more detail)."
- [abstract] Inspection of the metabolic interactions in the soil communities allows us to identify microbes with community roles of helpers and beneficiaries: "Inspection of the metabolic interactions in the soil communities allows us to identify microbes with community roles of helpers and beneficiaries"
