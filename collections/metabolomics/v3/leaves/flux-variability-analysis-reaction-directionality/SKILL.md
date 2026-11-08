---
name: flux-variability-analysis-reaction-directionality
description: Use when a genome-scale metabolic model in SBML format (e.g. BioModels
  MODEL2007210001) must have its flux bounds refined by FVA before reaction directionality
  classification or chokepoint/drug-target analysis.
when_to_use_negative:
- When the goal is basic Flux Balance Analysis of biomass or growth rate only — FVA
  refinement of flux bounds is unnecessary
- When the model has not been checked for dead-end metabolites and the analysis requires
  that every bottleneck reaction be balanced by at least one other reaction
edam_topics:
- http://edamontology.org/topic_2259
tools:
- name: findCPcli
  role: Command-line tool that runs FVA to update reaction flux bounds, classifies
    reactions as reversible/non-reversible/dead, removes dead-end metabolites, and
    computes chokepoint reactions
  repo: https://github.com/findCP/findCPcli
- name: COBRApy
  role: Underlying Python library (via findCPcore low-level API) that parses the SBML
    model and supports FBA/FVA simulation
  repo: https://github.com/opencobra/cobrapy
- name: GLPK
  role: Linear programming solver used by COBRApy to solve the FVA optimization problems
- name: findCPcore
  role: Low-level Python API for programmatic FVA refinement and chokepoint computation
    (model.fva(update_flux=True); model.find_chokepoints())
  repo: https://github.com/findCP/findCPcli
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.26508/lsa.202000954
    title: Integrated human/SARS-CoV-2 metabolic models present novel treatment strategies
      against COVID-19
  source_content_hash: sha256:866fb98232dc4838435237dbaae59b348cd1c249d19b2ee80837422dc655fac9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/findCP/findCPcli
  tools:
  - COBRApy
  - GLPK
  - findCPcli
  - findCPcore
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_2259
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# flux-variability-analysis-reaction-directionality

## Summary

Compute per-reaction flux bounds via Flux Variability Analysis (FVA) and use the refined bounds to classify each reaction of a genome-scale metabolic model (e.g. iHsaEC21) as reversible, non-reversible, or dead, which is the prerequisite for computing growth-dependent chokepoint (bottleneck) reactions with findCPcli.

## When to use

Use when a genome-scale metabolic model in SBML format (e.g. BioModels MODEL2007210001) must have its flux bounds refined by FVA before reaction directionality classification or chokepoint/drug-target analysis.

## When NOT to use

- When the goal is basic Flux Balance Analysis of biomass or growth rate only — FVA refinement of flux bounds is unnecessary
- When the model has not been checked for dead-end metabolites and the analysis requires that every bottleneck reaction be balanced by at least one other reaction

## Inputs

- Genome-scale metabolic model in SBML (.xml), JSON, or YAML format (e.g. iHsaEC21_SARS-CoV-2, BioModels MODEL2007210001)
- Objective reaction ID (e.g. host biomass or viral growth reaction) for Flux Balance Analysis
- Optional fraction of optimum growth for FVA (0.0–1.0)

## Outputs

- FVA-refined model file (.xml/./.yml) with updated reaction flux bounds
- FVA-refined, DEM-free model file (-swDF)
- Output spreadsheet (.xls/.xlsx/.ods) with sheets classifying reactions and listing chokepoint reactions and the metabolites they produce/consume
- Growth-dependent chokepoints spreadsheet (-cp)

## How to apply

Load the SBML model with COBRApy using GLPK as the LP solver (findCPcli wraps both). Run FVA with findCPcli to update the lower and upper flux bounds of every reaction, optionally constraining to a fraction of optimal growth (e.g. -fraction 0.95 for 95% of optimal biomass). Use the refined bounds to classify each reaction as reversible (can carry flux in both directions), non-reversible, or dead, since chokepoint producer/consumer logic depends on this directionality. Optionally remove reactions with dead-end metabolites (DEM) so each bottleneck reaction is balanced by at least one other consuming/producing reaction; the article evaluates four variants: input model, DEM-free, FVA-refined, and FVA-refined and DEM-free. Save the FVA-refined model (-sF) or the FVA-refined DEM-free model (-swDF), and write the chokepoint results, including the 'chokepoints' sheet, to the output spreadsheet.

## Related tools

- **findCPcli** (Command-line tool that runs FVA to update reaction flux bounds, classifies reactions as reversible/non-reversible/dead, removes dead-end metabolites, and computes chokepoint reactions) — https://github.com/findCP/findCPcli
- **COBRApy** (Underlying Python library (via findCPcore low-level API) that parses the SBML model and supports FBA/FVA simulation) — https://github.com/opencobra/cobrapy
- **GLPK** (Linear programming solver used by COBRApy to solve the FVA optimization problems)
- **findCPcore** (Low-level Python API for programmatic FVA refinement and chokepoint computation (model.fva(update_flux=True); model.find_chokepoints())) — https://github.com/findCP/findCPcli

## Examples

```
findCPcli -i iHsaEC21_SARS-CoV-2.xml -o output.xls -fraction 0.95
```

## Evaluation signals

- The output spreadsheet contains a populated 'chokepoints' sheet listing each chokepoint reaction with the metabolite(s) it produces/consumes
- For the iHsaEC21_SARS-CoV-2 model, the FVA-refined, DEM-free chokepoint set matches the 1,598 bottleneck reactions reported in Supplementary Table S4
- Every reaction is classified as reversible, non-reversible, or dead based on the FVA-updated bounds, with no unclassified reactions
- Reactions with dead-end metabolites are excluded from the refined model before chokepoint computation
- Chokepoint results are reproducible across all four model variants (input, DEM-free, FVA-refined, FVA-refined and DEM-free)

## Limitations

- Chokepoint results differ substantially between the four model variants (input, DEM-free, FVA-refined, FVA-refined and DEM-free), so the FVA-refined DEM-free variant must be chosen deliberately
- Reactions with dead-end metabolites must be removed, since a bottleneck reaction should be balanced by at least one other reaction producing or consuming the metabolite
- The paper's drug-target predictions (20 unique reactions) are in silico only, with no experimental validation

## Evidence

- [intro] FVA determines reaction directionality: "FVA is used to compute flux bounds of the reactions, and in turn, to determine whether a given reaction is reversible or not."
- [readme] findCPcli FVA capabilities: "findCPcli can compute and remove dead-end metabolites, find essential reactions and update the flux bounds of the reactions according to the results of Flux Variability Analysis."
- [methods] Four model variants evaluated: "ii) model without DEM; iii) model refined with FVA, i.e. with flux bounds updated according to FVA; and iv) model refined with FVA and without DEM"
- [intro] DEM removal rationale: "Because each bottleneck reaction should be balanced by at least one other reaction that produces or consumes that metabolite, we have excluded reactions in the model with dead-end metabolites."
- [methods] Growth-dependent chokepoints: "findCPcli can be used to compute `Growth Dependent Chokepoints <https://doi.org/10.1007/978-3-030-60327-4_6>`_"
