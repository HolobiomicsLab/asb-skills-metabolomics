---
name: uptake-reaction-perturbation-simulation
description: Use when gap-filled community GEMs exist and you must determine which
  exchanged metabolites each member depends on, by blocking one uptake reaction at
  a time and recording biomass-flux reduction.
when_to_use_negative:
- The input is raw draft reconstructions that have not yet been gap-filled with COMMIT
  — dependencies computed on incomplete models will be artifacts of missing reactions
  rather than true cross-feeding dependencies.
- The goal is draft reconstruction quality assessment (e.g. memote test suite or structural
  comparison by distance measures) rather than interrogation of metabolic interactions.
- The analysis concerns gap-filling reaction weights or permeability classification
  of metabolites, which belongs to the COMMIT gap-filling step, not uptake-perturbation
  simulation.
edam_operation: http://edamontology.org/operation_3660
edam_topics: []
tools:
- name: COMMIT
  role: Produces the gap-filled community models (run_iterative_gap_filling.m) that
    serve as input to the uptake-perturbation FBA analysis
  repo: https://github.com/pwendering/COMMIT
- name: COBRA toolbox
  role: Matlab framework for loading the community models, blocking uptake reactions,
    and solving the FBA LPs
  repo: https://github.com/opencobra/cobratoolbox
- name: SMETANA
  role: Downstream tool used for directed interaction prediction on the same gap-filled
    models; uptake-perturbation results can be cross-checked against SMETANA dependencies
  repo: https://github.com/cdanielmachado/smetana
- name: pheatmap
  role: R package used to visualize the member-by-uptake-reaction biomass-reduction
    matrix as a clustered heatmap with wesanderson/scales palettes
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1371/journal.pcbi.1009906
    title: 'COMMIT: Consideration of metabolite leakage and community composition
      improves microbial community reconstructions'
  source_content_hash: sha256:e28e085b531ecad80a5ec19c2a04b9d61e22fcd88ae4cd61ff4b065db7201cd7
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/pwendering/COMMIT
  tools:
  - COBRA toolbox
  - COMMIT
  - SMETANA
  - pheatmap
  edam_operation: http://edamontology.org/operation_3660
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# uptake-reaction-perturbation-simulation

## Summary

Simulates single-uptake-reaction removals in gap-filled community metabolic models using COBRA toolbox FBA (LP solved with CPLEX or Gurobi) to quantify, for each soil-community member, the biomass flux reduction caused by losing access to each exchanged metabolite.

## When to use

Use when gap-filled community GEMs exist and you must determine which exchanged metabolites each member depends on, by blocking one uptake reaction at a time and recording biomass-flux reduction.

## When NOT to use

- The input is raw draft reconstructions that have not yet been gap-filled with COMMIT — dependencies computed on incomplete models will be artifacts of missing reactions rather than true cross-feeding dependencies.
- The goal is draft reconstruction quality assessment (e.g. memote test suite or structural comparison by distance measures) rather than interrogation of metabolic interactions.
- The analysis concerns gap-filling reaction weights or permeability classification of metabolites, which belongs to the COMMIT gap-filling step, not uptake-perturbation simulation.

## Inputs

- Gap-filled community metabolic models for the Bulgarelli and Schlaeppi soil communities, saved as Matlab workspaces (output of run_iterative_gap_filling.m)
- List of extracellular exchange/uptake reactions from the community models (MNXref/MNXM identifiers)
- LP solver configuration (IBM CPLEX v12.9 or Gurobi) for the COBRA toolbox

## Outputs

- Member-by-uptake-reaction matrix of biomass flux reductions (dependency calls per metabolite and per community member)
- Clustered heatmap (R pheatmap, wesanderson/scales palettes) of the dependency matrix
- Validated helper/beneficiary interaction assignments consistent with published mineral (Fe2+, Fe3+, SO4^2-) and amino-acid dependency patterns

## How to apply

Load the gap-filled community models for the Bulgarelli and Schlaeppi soil communities (Matlab workspaces produced by run_iterative_gap_filling.m in the pwendering/COMMIT repository) into the COBRA toolbox v3.0. Enumerate the extracellular exchange/uptake reactions, and for each uptake reaction block it (set lower bound to 0) and re-run FBA, solving the LP with IBM CPLEX v12.9 or Gurobi. Record the resulting biomass flux reduction for every community member, so a member with a large drop is a beneficiary of the metabolite's providers. Keep biomass flux constraints consistent with the COMMIT pipeline (upper biomass limit 2.81 h^-1 per isolate model; suboptimal factor f = 0.9 was used in the study for sink-reaction maximization of secretion). Cross-check computed dependencies against the reported mineral dependencies (Fe2+, Fe3+, SO4^2-) and community-specific amino-acid patterns, then visualize the member-by-uptake-reaction biomass-reduction matrix as a clustered heatmap with R pheatmap (wesanderson/scales palettes). Validation requires reproducing the published helper/beneficiary interaction pattern for both soil communities.

## Related tools

- **COMMIT** (Produces the gap-filled community models (run_iterative_gap_filling.m) that serve as input to the uptake-perturbation FBA analysis) — https://github.com/pwendering/COMMIT
- **COBRA toolbox** (Matlab framework for loading the community models, blocking uptake reactions, and solving the FBA LPs) — https://github.com/opencobra/cobratoolbox
- **SMETANA** (Downstream tool used for directed interaction prediction on the same gap-filled models; uptake-perturbation results can be cross-checked against SMETANA dependencies) — https://github.com/cdanielmachado/smetana
- **pheatmap** (R package used to visualize the member-by-uptake-reaction biomass-reduction matrix as a clustered heatmap with wesanderson/scales palettes)

## Examples

```
modelU = changeRxnBounds(model, uptakeRxn, 0, 'lb'); sol = optimizeCbModel(modelU); % biomass flux reduction = sol.obj vs. unperturbed FBA optimum (COBRA toolbox, CPLEX/Gurobi LP)
```

## Evaluation signals

- Blocking a mineral uptake reaction (Fe2+, Fe3+, or SO4^2-) must reduce biomass flux for members across most bacterial families, reproducing the reported community-wide mineral dependencies.
- The computed uptake dependencies must match the published helper/beneficiary interaction pattern for both the Bulgarelli and Schlaeppi communities.
- Reproduced pheatmap clustered heatmap must match the published member-by-uptake-reaction dependency figure for both soil communities.
- FBA solutions must be feasible for the control model with all uptakes allowed, and biomass flux upper limits must not exceed 2.81 h^-1 per isolate model.
- Amino-acid exchange dependencies must reproduce the community-specific, within-family-limited patterns reported in the article.

## Limitations

- Full COMMIT procedure specification is deferred to supplementary S1 Text, so some procedural details must be obtained from the supplement rather than the main text.
- Gap-filling penalty weights were tuned ad hoc until satisfying results with no systematic selection procedure, so dependency calls may inherit biases from the gap-filling step.
- 16S rRNA abundance data used to contextualize community members were only available via personal provision from Ruben Garrido Oter, not a public accession, limiting reproducibility of abundance-weighted analyses.
- Suboptimal biomass factor f = 0.9 is a study-specific choice; other values may shift which dependencies are called.

## Evidence

- [abstract] Inspection of the metabolic interactions in the soil communities allows us to identify microbes with community roles of helpers and beneficiaries: "Inspection of the metabolic interactions in the soil communities allows us to identify microbes with community roles of helpers and beneficiaries"
- [discussion] In this study, f = 0.9 has been used (please see S1 Text for more detail).: "In this study, f = 0.9 has been used (please see S1 Text for more detail)."
- [discussion] The upper limit for the biomass flux for each isolate model was constrained to 2.81 h−1: "The upper limit for the biomass flux for each isolate model was constrained to 2.81 h−1"
- [intro] we conducted a downstream analysis of the gap-filled models using the SMETANA tool: "we conducted a downstream analysis of the gap-filled models using the SMETANA tool"
- [readme] This repository contains all files used for community-dependent gap-filling and subsequent analysis of consensus genome-scale metabolic reconstructions for two soil communities: "This repository contains all files used for community-dependent gap-filling and subsequent analysis of consensus genome-scale metabolic reconstructions for two soil communities (Bulgarelli et al."
