---
name: essential-reaction-deletion-analysis
description: Use when an integrated host/virus SBML metabolic model (e.g. iHsaEC21+SARS-CoV-2
  or iAB-AMØ-1410 macrophage) needs its host-dependency factors enumerated as reactions
  whose deletion abolishes the viral biomass objective.
when_to_use_negative:
- The goal is bottleneck or chokepoint prediction for drug-target prioritisation rather
  than enumeration of reactions required for the objective — findCPcli's chokepoint/dead-end
  metabolite operations are the appropriate route
- The input is not a genome-scale metabolic model in SBML (.xml), ., or .yml form
  (findCPcli only accepts these formats)
- FVA refinement of flux bounds has not been performed and the unrefined model would
  give different flux-dependent essentiality results
edam_topics:
- http://edamontology.org/topic_0602
tools:
- name: findCPcli
  role: Command-line tool that computes essential reactions via single-reaction deletion
    on the SBML model and exports results to a spreadsheet; also updates flux bounds
    via FVA
  repo: https://github.com/findCP/findCPcli
- name: COBRApy
  role: Python toolbox underlying findCPcli (findCPcore Low Level API is COBRApy-based)
    used to load and simulate the integrated model with the viral biomass objective
  repo: https://github.com/opencobra/cobrapy
- name: GLPK
  role: Linear programming solver used with COBRApy for the FBA/FVA and deletion simulations
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.26508/lsa.202000954
    title: Integrated human/SARS-CoV-2 metabolic models present novel treatment strategies
      against COVID-19
  source_content_hash: sha256:7042027a487d64263ce8a59aaccc3c60ea0182e1553ede10a43a019e89548ade
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/findCP/findCPcli
  tools:
  - COBRApy
  - GLPK
  - findCPcli
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# essential-reaction-deletion-analysis

## Summary

Identifies host reactions that are essential for viral propagation by running single-reaction deletion analysis (restricting each reaction's flux to zero) on integrated human/SARS-CoV-2 genome-scale metabolic models with the objective set to the viral biomass maintenance function, using findCPcli/COBRApy.

## When to use

Use when an integrated host/virus SBML metabolic model (e.g. iHsaEC21+SARS-CoV-2 or iAB-AMØ-1410 macrophage) needs its host-dependency factors enumerated as reactions whose deletion abolishes the viral biomass objective.

## When NOT to use

- The goal is bottleneck or chokepoint prediction for drug-target prioritisation rather than enumeration of reactions required for the objective — findCPcli's chokepoint/dead-end metabolite operations are the appropriate route
- The input is not a genome-scale metabolic model in SBML (.xml), ., or .yml form (findCPcli only accepts these formats)
- FVA refinement of flux bounds has not been performed and the unrefined model would give different flux-dependent essentiality results

## Inputs

- Integrated host/SARS-CoV-2 SBML model file (e.g. iHsaEC21+SARS-CoV-2 or iAB-AMØ-1410 SARS-CoV-2 model, .xml)
- Objective reaction identifier for the viral biomass maintenance function

## Outputs

- Spreadsheet (.xls/.xlsx/.ods) with an 'essential reactions' sheet listing essential reaction identifiers per model variant
- Count and identity of essential reactions per refined model variant (expected: 10 in iHsaEC21, 52 in iAB-AMØ-1410)

## How to apply

Load the integrated SBML model (e.g. iHsaEC21+SARS-CoV-2, BioModels MODEL2007210001, or the Renz et al macrophage model) and set the objective to the viral biomass maintenance function. First refine the model with Flux Variability Analysis using findCPcli/COBRApy with GLPK so that every reaction's lower and upper flux bounds are updated (the FVA-refined model variant). Then run findCPcli's essential reaction computation, which performs single-reaction deletion: each reaction's flux is restricted to zero and the reaction is flagged essential if the objective drops to zero. Export results to a spreadsheet and extract the 'essential reactions' sheet. Validate by comparing counts and identities against the published results: 10 essential reactions for iHsaEC21 and 52 for iAB-AMØ-1410 (Tables S1 and S2).

## Related tools

- **findCPcli** (Command-line tool that computes essential reactions via single-reaction deletion on the SBML model and exports results to a spreadsheet; also updates flux bounds via FVA) — https://github.com/findCP/findCPcli
- **COBRApy** (Python toolbox underlying findCPcli (findCPcore Low Level API is COBRApy-based) used to load and simulate the integrated model with the viral biomass objective) — https://github.com/opencobra/cobrapy
- **GLPK** (Linear programming solver used with COBRApy for the FBA/FVA and deletion simulations)

## Examples

```
findCPcli -i iHsaEC21_SARS-CoV-2.xml -o results.xls -objective <viral_biomass_reaction_id>
```

## Evaluation signals

- Essential reaction count reproduces the reported values: 10 essential reactions in the iHsaEC21 epithelial model and 52 in the iAB-AMØ-1410 macrophage model
- Essential reaction identifiers match the published Tables S1 and S2 for the FVA-refined model variants
- Deletion of each listed reaction (flux restricted to zero) yields a zero viral biomass objective when re-simulated
- The output spreadsheet contains the 'essential reactions' sheet in a supported format (.xls/.xlsx/.ods)
- The analysis is run on the FVA-refined model (flux bounds updated), not the raw model, so results are consistent with the paper's refined-model variant

## Limitations

- Essentiality is objective-dependent: only reactions required for the viral biomass maintenance function are detected, not those required for host viability
- Results depend on the model variant used (with/without dead-end metabolites, FVA-refined vs raw); the paper distinguishes four model variants and the reference counts apply to the FVA-refined variants
- findCPcli accepts only .xml, ., and .yml model files and outputs only .xls, .xlsx, or .ods spreadsheets

## Evidence

- [readme] Essential reactions definition: "A reaction is considered an essential reaction if its deletion, this is, restricting its flux to zero, causes the objective (e.g. cellular growth) to be zero."
- [readme] FVA and essential reactions in findCPcli: "In addition to the computation of chokepoints, findCPcli can compute and remove dead-end metabolites, find essential reactions and update the flux bounds of the reactions according to the results of"
- [methods] FVA-refined model variant: "iii) model refined with FVA, i.e. with flux bounds updated according to FVA"
- [methods] Host dependency factors with findCPcli: "identify the host-dependency factors for the SARS-CoV-2 virus by using the novel software tool findCPcli"
