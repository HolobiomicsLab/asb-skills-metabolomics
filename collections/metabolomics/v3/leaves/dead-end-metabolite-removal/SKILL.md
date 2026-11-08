---
name: dead-end-metabolite-removal
description: Use when computing chokepoint/bottleneck reactions on an SBML genome-scale
  metabolic model and you need each bottleneck reaction to be balanced by at least
  one other producing or consuming reaction.
when_to_use_negative:
- Input is not a metabolic model but, e.g., a gene expression or PPI dataset — DEM
  removal is defined only on stoichiometric reaction networks
- The analysis goal is flux distribution simulation (FBA/FVA) itself rather than topological
  chokepoint identification, where DEM reactions can carry legitimate flux in some
  contexts
- You need the unfiltered bottleneck counts from the input model variant for comparison
  with the DEM-free variant (the paper computes all four variants in parallel)
edam_operation: http://edamontology.org/operation_3660
edam_topics:
- http://edamontology.org/topic_0602
tools:
- name: findCPcli
  role: Computes and removes dead-end metabolites from the model and then computes
    chokepoint reactions on the DEM-free variant
  repo: https://github.com/findCP/findCPcli
- name: COBRApy
  role: Parses the SBML model and manipulates/simulates it (reaction removal) under
    findCPcli
  repo: https://github.com/opencobra/cobrapy
- name: GLPK
  role: LP solver used by COBRApy for FVA-based flux bound refinement preceding DEM
    removal
provenance:
  source_task_ids:
  - task_002
  - task_003
  source_papers:
  - doi: 10.26508/lsa.202000954
    title: Integrated human/SARS-CoV-2 metabolic models present novel treatment strategies
      against COVID-19
  source_content_hash: sha256:59b368e4e5b512fd3b84f7cfff0b31f2d9116fe7e361d500d1dbdce0fe07913e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/findCP/findCPcli
  tools:
  - COBRApy
  - GLPK
  - Python
  - findCPcli
  edam_operation: http://edamontology.org/operation_3660
  edam_topics:
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# dead-end-metabolite-removal

## Summary

A preprocessing step in chokepoint (bottleneck) analysis where reactions containing dead-end metabolites (DEMs) — metabolites that cannot be balanced by any other producing or consuming reaction — are identified and removed from a genome-scale metabolic model using findCPcli before computing chokepoints.

## When to use

Use when computing chokepoint/bottleneck reactions on an SBML genome-scale metabolic model and you need each bottleneck reaction to be balanced by at least one other producing or consuming reaction.

## When NOT to use

- Input is not a metabolic model but, e.g., a gene expression or PPI dataset — DEM removal is defined only on stoichiometric reaction networks
- The analysis goal is flux distribution simulation (FBA/FVA) itself rather than topological chokepoint identification, where DEM reactions can carry legitimate flux in some contexts
- You need the unfiltered bottleneck counts from the input model variant for comparison with the DEM-free variant (the paper computes all four variants in parallel)

## Inputs

- Genome-scale metabolic model in SBML format (e.g. BioModels MODEL2007210001, iHsaEC21-based integrated model)
- FVA-refined model variant with updated lower and upper flux bounds (optional but recommended)
- findCPcli results spreadsheet settings (output .xls file)

## Outputs

- DEM-free metabolic model variant used for downstream chokepoint computation
- Chokepoints sheet in the output spreadsheet listing bottleneck reaction identifiers and the metabolite(s) each produces/consumes
- List of removed dead-end-metabolite reactions (reported among the model variants: input model, DEM-free, FVA-refined, FVA-refined and DEM-free)

## How to apply

Load the SBML genome-scale metabolic model (e.g. the iHsaEC21 airway epithelial/SARS-CoV-2 model from BioModels MODEL2007210001) into findCPcli, which parses models via COBRApy with GLPK as the LP solver. Run findCPcli's dead-end metabolite detection to find metabolites with no mechanism for production or consumption, then remove all reactions containing those DEMs from the model. Apply this removal on the FVA-refined model variant (flux bounds updated by Flux Variability Analysis) so that reaction reversibility classifications and chokepoint computations reflect flux-dependent information. The rationale is that each bottleneck reaction should be balanced by at least one other reaction that produces or consumes that metabolite; unbalanced DEM reactions would otherwise inflate the chokepoint set. Re-running the analysis on the FVA-refined, DEM-free variant is the configuration that reproduces the published counts (e.g. 1,598 bottlenecks in the airway epithelial model).

## Related tools

- **findCPcli** (Computes and removes dead-end metabolites from the model and then computes chokepoint reactions on the DEM-free variant) — https://github.com/findCP/findCPcli
- **COBRApy** (Parses the SBML model and manipulates/simulates it (reaction removal) under findCPcli) — https://github.com/opencobra/cobrapy
- **GLPK** (LP solver used by COBRApy for FVA-based flux bound refinement preceding DEM removal)

## Examples

```
pip install findCPcli
```

## Evaluation signals

- The output spreadsheet contains a 'chokepoints' sheet populated with bottleneck reaction identifiers consistent with the FVA-refined, DEM-free model
- For the airway epithelial iHsaEC21_SARS-CoV-2 model, the FVA-refined DEM-free variant yields 1,598 bottleneck reactions matching Supplementary Table S4
- For the macrophage iAB-AMØ-1410-based model, the computed bottleneck set matches the 1,595 reactions reported in Table S3
- Every retained bottleneck reaction has at least one other reaction producing or consuming each metabolite it touches (no DEM-related unbalanced reactions remain)
- Computed bottleneck identifiers overlap with the published reaction identifier sets

## Limitations

- DEM removal is a topology-driven filter; reactions excluded for containing dead-end metabolites are not evaluated as potential drug targets in the DEM-free variant
- The chokepoint counts depend on the model variant chosen (input, DEM-free, FVA-refined, FVA-refined and DEM-free); different variants give different bottleneck sets
- Results depend on the LP solver and COBRApy version (paper used COBRApy 0.16.0 with GLPK); reproducing exact counts may require the same versions

## Evidence

- [intro] Because each bottleneck reaction should be balanced by at least one other reaction that produces or consumes that metabolite, we have excluded reactions in the model with dead-end metabolites.: "Because each bottleneck reaction should be balanced by at least one other reaction that produces or consumes that metabolite, we have excluded reactions in the model with dead-end metabolites."
- [methods] we removed the reactions with dead-end metabolites, which were previously found in the automated model: "we removed the reactions with dead-end metabolites, which were previously found in the automated model"
- [readme] In addition to the computation of chokepoints, findCPcli can compute and remove dead-end metabolites, find essential reactions and update the flux bounds of the reactions according to the results of Flux Variability Analysis.: "In addition to the computation of chokepoints, findCPcli can compute and remove dead-end metabolites, find essential reactions and update the flux bounds of the reactions according to"
- [methods] iv) model refined with FVA and without DEM: "ii) model without DEM; iii) model refined with FVA, i.e. with flux bounds updated according to FVA; and iv) model refined with FVA and without DEM"
