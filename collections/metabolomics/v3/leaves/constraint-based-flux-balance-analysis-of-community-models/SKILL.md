---
name: constraint-based-flux-balance-analysis-of-community-models
description: Use when you have gap-filled SBML or Matlab community GEMs and need to
  identify which extracellular metabolites are exchanged among members and which members
  depend on which cross-feeding metabolites.
when_to_use_negative:
- You need whole-community steady-state optimization with fixed member abundances
  — the article notes SteadyCom or MICOM perform such community-level optimization
  instead
- The models are draft reconstructions not yet gap-filled (e.g. raw KBase, CarveMe,
  RAVEN 2.0, AuReMe/Pathway Tools drafts) — exchange/dependency analysis requires
  the COMMIT gap-filled models
- Input is 16S abundance data or genome lists without metabolic models — no flux computation
  is possible
edam_operation: http://edamontology.org/operation_3660
edam_topics:
- http://edamontology.org/topic_2259
- http://edamontology.org/topic_0602
tools:
- name: COBRA toolbox
  role: MATLAB framework for loading the community models and solving FBA LPs (sink
    maximization and uptake blocking)
  repo: https://github.com/opencobra/cobratoolbox
- name: COMMIT
  role: Produces the gap-filled community-dependent models that are the input for
    the exchange/dependency FBA
  repo: https://github.com/pwendering/COMMIT
- name: SMETANA
  role: Downstream tool for directed metabolic interaction prediction over the gap-filled
    models
  repo: https://github.com/cdanielmachado/smetana
- name: pheatmap
  role: R package for visualizing the member-by-uptake biomass-reduction matrix as
    a clustered heatmap
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1371/journal.pcbi.1009906
    title: 'COMMIT: Consideration of metabolite leakage and community composition
      improves microbial community reconstructions'
  source_content_hash: sha256:e48caa84e5f6199791924f6917812a9e694aa1c92c90708c21d23ffdb9393107
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/opencobra/cobratoolbox
  tools:
  - COBRA toolbox
  - COMMIT
  - SMETANA
  - pheatmap
  edam_operation: http://edamontology.org/operation_3660
  edam_topics:
  - http://edamontology.org/topic_2259
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8666666666666667
n_claims: 15
---

# constraint-based-flux-balance-analysis-of-community-models

## Summary

Applies constraint-based flux balance analysis (FBA) to gap-filled genome-scale community models to quantify metabolite exchange and dependency: maximizing sink-reaction secretion at suboptimal biomass (f = 0.9) and blocking individual uptake reactions to measure biomass flux reduction per member.

## When to use

Use when you have gap-filled SBML or Matlab community GEMs and need to identify which extracellular metabolites are exchanged among members and which members depend on which cross-feeding metabolites.

## When NOT to use

- You need whole-community steady-state optimization with fixed member abundances — the article notes SteadyCom or MICOM perform such community-level optimization instead
- The models are draft reconstructions not yet gap-filled (e.g. raw KBase, CarveMe, RAVEN 2.0, AuReMe/Pathway Tools drafts) — exchange/dependency analysis requires the COMMIT gap-filled models
- Input is 16S abundance data or genome lists without metabolic models — no flux computation is possible

## Inputs

- Gap-filled community metabolic models for the Bulgarelli and Schlaeppi soil communities (Matlab workspaces from run_iterative_gap_filling.m)
- List of extracellular exchange/uptake reactions per community member
- Suboptimal biomass factor f (0.9 in this study)

## Outputs

- Maximal secretion/leakage flux per member per exchanged metabolite (sink-reaction maximization at f = 0.9)
- Member-by-uptake-reaction biomass flux reduction matrix (dependency calls)
- Clustered heatmap of the dependency matrix (pheatmap)
- Helper/beneficiary community-role classification per member

## How to apply

Load the gap-filled community models (output of run_iterative_gap_filling.m saved as Matlab workspaces) into the COBRA toolbox v3.0 with an LP solver (CPLEX v12.9 or Gurobi). First, enumerate extracellular exchange/uptake reactions and compute each member's maximal secretion contribution to the exchanged-metabolite pool by maximizing sink reactions subject to a suboptimal biomass constraint with factor f = 0.9. Then, for each uptake reaction, run FBA with that uptake blocked and record the resulting biomass flux reduction for every community member, yielding a member-by-uptake-reaction dependency matrix. Cross-check the computed dependencies against mineral dependencies (Fe2+, Fe3+, SO4^2-) and community-specific amino-acid exchange patterns, then visualize the biomass-reduction matrix as a clustered heatmap with pheatmap. The reduced biomass upon uptake-block identifies the beneficiary; the metabolite's secretor identifies the helper.

## Related tools

- **COBRA toolbox** (MATLAB framework for loading the community models and solving FBA LPs (sink maximization and uptake blocking)) — https://github.com/opencobra/cobratoolbox
- **COMMIT** (Produces the gap-filled community-dependent models that are the input for the exchange/dependency FBA) — https://github.com/pwendering/COMMIT
- **SMETANA** (Downstream tool for directed metabolic interaction prediction over the gap-filled models) — https://github.com/cdanielmachado/smetana
- **pheatmap** (R package for visualizing the member-by-uptake biomass-reduction matrix as a clustered heatmap)

## Examples

```
matlab -r "initCobraToolbox(); load('Bulgarelli_community_gapfilled.mat'); [biomassReduction, uptakeDeps] = analyzeCommunityExchanges(model, f=0.9);"
```

## Evaluation signals

- The reproduced heatmap and dependency calls must match the published helper/beneficiary interaction pattern for both soil communities
- Computed uptake dependencies must include the reported mineral dependencies (Fe2+, Fe3+, SO4^2-), which are provided by members of most bacterial families
- Sink-reaction maximization must use the documented suboptimal biomass factor f = 0.9
- Community-specific amino-acid and specialized within-family-limited exchange patterns must be recoverable from the dependency matrix
- LP solutions must be reproducible with CPLEX or Gurobi via the COBRA toolbox

## Limitations

- Full COMMIT and downstream analysis procedures are deferred to supplementary S1 Text, not the main text
- 16S rRNA abundance data were only available via personal provision from Ruben Garrido Oter, not a public accession
- Penalty weights in gap filling were tuned ad hoc until satisfying results, with no systematic selection procedure documented
- Analyses assume putative dependencies inferred from model structure, not experimentally validated cross-feeding

## Evidence

- [other] Enumerate the extracellular exchange/uptake reactions and compute each member's maximal secretion/leakage flux contribution to the exchanged-metabolite pool using sink-reaction maximization with the suboptimal biomass factor f = 0.9.: [evidence span withheld — anchor status: not_found]
- [discussion] In this study, f = 0.9 has been used (please see S1 Text for more detail).: "In this study, f = 0.9 has been used (please see S1 Text for more detail)."
- [abstract] Inspection of the metabolic interactions in the soil communities allows us to identify microbes with community roles of helpers and beneficiaries: "Inspection of the metabolic interactions in the soil communities allows us to identify microbes with community roles of helpers and beneficiaries"
- [other] Most models across all bacterial families show putative dependencies for Fe2+, Fe3+, and SO42-, which are in turn also provided by members of most bacterial families: [evidence span withheld — anchor status: not_found]
- [readme] The COBRA Toolbox is a comprehensive suite of functions for constraint-based modeling of biological networks. It supports the analysis of genome-scale metabolic reconstructions and other constraint-based models in MATLAB.: [evidence span withheld — anchor status: not_found]
