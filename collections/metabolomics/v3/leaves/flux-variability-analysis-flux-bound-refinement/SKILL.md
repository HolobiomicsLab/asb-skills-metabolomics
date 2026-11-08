---
name: flux-variability-analysis-flux-bound-refinement
description: Use when an SBML genome-scale metabolic model (e.g. iHsaEC21+SARS-CoV-2
  or the Renz et al macrophage model) needs flux-bound refinement before reaction
  reversibility classification, bottleneck or essential-reaction computation.
when_to_use_negative:
- The analysis goal is purely topological chokepoint computation on the unrefined
  model variant — the article explicitly contrasts the plain model with the FVA-refined
  variant
- No LP solver infrastructure is available, since FVA requires repeated LP solves
  (GLPK in this workflow)
- The objective is host biomass maintenance in the absence of the virus, which is
  computed by plain FBA rather than FVA-based refinement
tools:
- name: COBRApy
  role: Manipulates and simulates the model and runs FVA to update reaction flux bounds
    (version 0.16.0 used for iHsaEC21)
  repo: https://github.com/opencobra/cobrapy
- name: GLPK
  role: LP solver underlying the FBA/FVA optimization problems
- name: findCPcli
  role: Consumes the FVA-refined model variant and exports chokepoint/essential-reaction
    results to a spreadsheet; can also update flux bounds according to FVA results
  repo: https://github.com/findCP/findCPcli
provenance:
  source_task_ids:
  - task_003
  - task_004
  - task_008
  source_papers:
  - doi: 10.26508/lsa.202000954
    title: Integrated human/SARS-CoV-2 metabolic models present novel treatment strategies
      against COVID-19
  source_content_hash: sha256:7eeec63011d42873d3f454cd2f41365eea75e20edbb9b50e5f2613e8e9c4c900
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/opencobra/cobrapy
  tools:
  - COBRApy
  - GLPK
  - Python
  - findCPcli
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# flux-variability-analysis-flux-bound-refinement

## Summary

Runs Flux Variability Analysis (FVA) on SBML genome-scale metabolic models with COBRApy and GLPK to compute lower and upper flux bounds for every reaction, producing FVA-refined model variants that support downstream findCPcli analyses (bottleneck/chokepoint and essential-reaction identification).

## When to use

Use when an SBML genome-scale metabolic model (e.g. iHsaEC21+SARS-CoV-2 or the Renz et al macrophage model) needs flux-bound refinement before reaction reversibility classification, bottleneck or essential-reaction computation.

## When NOT to use

- The analysis goal is purely topological chokepoint computation on the unrefined model variant — the article explicitly contrasts the plain model with the FVA-refined variant
- No LP solver infrastructure is available, since FVA requires repeated LP solves (GLPK in this workflow)
- The objective is host biomass maintenance in the absence of the virus, which is computed by plain FBA rather than FVA-based refinement

## Inputs

- SBML file of an integrated genome-scale metabolic model (e.g. iHsaEC21+SARS-CoV-2 or macrophage iAB-AMØ-1410-based model)
- Objective function definition (viral biomass maintenance function or host biomass)
- Optional host-optimal growth rate μmax constraint from prior FBA

## Outputs

- FVA-refined model variant with updated lower and upper flux bounds per reaction
- Reaction classification (reversible / non-reversible / dead) and producer/consumer reactions
- Per-reaction flux bounds from FVA runs under host-optimum and virus-optimum conditions (exported e.g. as flux_comparison.xls)

## How to apply

Load the SBML-encoded integrated model (e.g. iHsaEC21+SARS-CoV-2 with 3,752 reactions, 3,914 metabolites, 4,660 genes) with COBRApy 0.16.0 and the GLPK LP solver. Run FVA to compute lower and upper flux bounds of every reaction, using the viral biomass maintenance function as the objective when the goal is virus-oriented refinement, or the host biomass objective constrained to μmax for host-optimum comparisons. Use the FVA-derived bounds to classify reactions as reversible, non-reversible, or dead, and to identify producer/consumer reactions feeding into chokepoint analysis. Produce an FVA-refined model variant (flux bounds updated according to FVA), optionally combined with dead-end metabolite (DEM) removal, and feed it to findCPcli for growth-dependent chokepoint or essential-reaction (single-reaction deletion) computations. The rationale is that static topology alone misclassifies reactions: FVA dynamics determine reversibility and exclude reactions that cannot carry flux under the chosen objective.

## Related tools

- **COBRApy** (Manipulates and simulates the model and runs FVA to update reaction flux bounds (version 0.16.0 used for iHsaEC21)) — https://github.com/opencobra/cobrapy
- **GLPK** (LP solver underlying the FBA/FVA optimization problems)
- **findCPcli** (Consumes the FVA-refined model variant and exports chokepoint/essential-reaction results to a spreadsheet; can also update flux bounds according to FVA results) — https://github.com/findCP/findCPcli

## Examples

```
findCPcli -i model.xml -o results.xls
```

## Evaluation signals

- The FVA-refined model variant yields exactly 1,595 bottleneck reactions for the macrophage model, matching Table S3 of the paper
- Essential-reaction deletion on the FVA-refined variants reproduces 10 essential reactions in iHsaEC21 and 52 in iAB-AMØ-1410 (Tables S1 and S2)
- Under virus-optimum conditions, FVA shows zero host biomass maintenance flux in the integrated epithelial model, confirming host non-viability
- Host-vs-virus flux comparisons grouped by pathway reproduce reported averages of 277% (32 lipid reactions), 228% (14 sphingolipid reactions), and ~190% (4 fatty-acid biosynthesis reactions)
- The flux comparison table covers all reactions returned by both FVA runs

## Limitations

- FVA-refined results depend on the chosen objective: virus-optimum conditions drive host biomass flux to zero, so host-pathway flux interpretations only hold under host-optimum constraints (vgrowth = μmax)
- Reactions with dead-end metabolites must be excluded before bottleneck analysis because each bottleneck reaction should be balanced by at least one other reaction producing or consuming that metabolite
- No version number was reported for MEMOTE quality scoring, and the models lack repository/accession identifiers beyond BioModels MODEL2007210001, limiting reproducibility of the exact refined variants

## Evidence

- [intro] FVA use for reversibility determination: "FVA is used to compute flux bounds of the reactions, and in turn, to determine whether a given reaction is reversible or not."
- [methods] FVA-refined model variant definition: "iii) model refined with FVA, i.e. with flux bounds updated according to FVA; and iv) model refined with FVA and without DEM"
- [readme] FVA purpose within findCPcli: "find essential reactions and update the flux bounds of the reactions according to the results of Flux Variability Analysis."
- [methods] Simulation toolchain: "We manipulated and simulated the model, including the addition of the gene expression data, with COBRApy (Ebrahim et al, 2013) and GLPK"
- [intro] Dead-end metabolite exclusion rationale: "Because each bottleneck reaction should be balanced by at least one other reaction that produces or consumes that metabolite, we have excluded reactions in the model with dead-end metabolites."
