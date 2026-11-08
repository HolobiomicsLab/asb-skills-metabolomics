---
name: bottleneck-drug-target-identification
description: Use when you have an SBML genome-scale metabolic model (e.g., iHsaEC21
  or iAB-AMØ-1410 integrated with SARS-CoV-2) and need to identify flux-dependent
  chokepoint reactions as candidate drug targets.
when_to_use_negative:
- The model still contains dead-end metabolites and has not been FVA-refined — unrefined
  bottleneck counts will not be comparable to the growth-dependent sets (e.g., 1,598
  for the airway epithelial model, 1,595 for the macrophage model)
- The analysis goal is pathway enrichment or gene-expression deconvolution rather
  than identifying topologically essential reactions in a constraint-based metabolic
  model
- The input is not a constraint-based metabolic model in SBML format (findCPcli expects
  genome-scale SBML models)
edam_operation: http://edamontology.org/operation_3660
edam_topics:
- http://edamontology.org/topic_2259
- http://edamontology.org/topic_0605
tools:
- name: findCPcli
  role: 'Primary tool: computes chokepoint reactions, removes dead-end metabolites,
    and updates flux bounds according to FVA; outputs the results spreadsheet with
    the chokepoints sheet'
  repo: https://github.com/findCP/findCPcli
- name: COBRApy
  role: Python toolbox used by findCPcli to parse the SBML model, manipulate/simulate
    it, and run Flux Variability Analysis with GLPK as LP solver
  repo: https://github.com/opencobra/cobrapy
- name: GLPK
  role: Linear programming solver underlying the FBA/FVA optimization steps in COBRApy
provenance:
  source_task_ids:
  - task_003
  - task_002
  source_papers:
  - doi: 10.26508/lsa.202000954
    title: Integrated human/SARS-CoV-2 metabolic models present novel treatment strategies
      against COVID-19
  source_content_hash: sha256:baa6526336a45e66a366e29620cd3d36f3c32ae99ad9f2bb825e43561e80ff74
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
  - http://edamontology.org/topic_2259
  - http://edamontology.org/topic_0605
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# bottleneck-drug-target-identification

## Summary

Computes chokepoint (bottleneck) reactions in genome-scale metabolic models using findCPcli, combining network topology with FVA-derived dynamic flux information to identify reactions required for SARS-CoV-2 maintenance and replication as candidate drug targets.

## When to use

Use when you have an SBML genome-scale metabolic model (e.g., iHsaEC21 or iAB-AMØ-1410 integrated with SARS-CoV-2) and need to identify flux-dependent chokepoint reactions as candidate drug targets.

## When NOT to use

- The model still contains dead-end metabolites and has not been FVA-refined — unrefined bottleneck counts will not be comparable to the growth-dependent sets (e.g., 1,598 for the airway epithelial model, 1,595 for the macrophage model)
- The analysis goal is pathway enrichment or gene-expression deconvolution rather than identifying topologically essential reactions in a constraint-based metabolic model
- The input is not a constraint-based metabolic model in SBML format (findCPcli expects genome-scale SBML models)

## Inputs

- Genome-scale metabolic model in SBML format (e.g., iHsaEC21_SARS-CoV-2 from BioModels MODEL2007210001, or the macrophage/SARS-CoV-2 model based on iAB-AMØ-1410 from Renz et al, 2020)
- LP solver configuration (GLPK)

## Outputs

- Spreadsheet file (e.g., results.xls) containing a 'chokepoints' sheet with bottleneck reaction identifiers and the metabolite(s) each reaction produces/consumes
- Model variant classifications (reversible / non-reversible / dead reactions)
- Dead-end metabolite annotations for the refined model

## How to apply

Load the SBML model into findCPcli, which parses it via COBRApy (v0.16.0) with GLPK as the LP solver. Run Flux Variability Analysis to update lower and upper flux bounds of each reaction, then use these refined bounds to classify every reaction as reversible, non-reversible, or dead. Remove reactions containing dead-end metabolites (DEM) so each bottleneck reaction is balanced by at least one other consuming/producing reaction. Compute chokepoints on the FVA-refined, DEM-free model by identifying consumer and producer reactions for each metabolite; optionally repeat on all four model variants (input, DEM-free, FVA-refined, FVA-refined and DEM-free) for comparison. Export the results spreadsheet and read the 'chokepoints' sheet listing bottleneck reaction identifiers with the metabolite(s) each produces/consumes.

## Related tools

- **findCPcli** (Primary tool: computes chokepoint reactions, removes dead-end metabolites, and updates flux bounds according to FVA; outputs the results spreadsheet with the chokepoints sheet) — https://github.com/findCP/findCPcli
- **COBRApy** (Python toolbox used by findCPcli to parse the SBML model, manipulate/simulate it, and run Flux Variability Analysis with GLPK as LP solver) — https://github.com/opencobra/cobrapy
- **GLPK** (Linear programming solver underlying the FBA/FVA optimization steps in COBRApy)

## Examples

```
findCPcli --inputModel model.xml --calculateChokepoints --fva --removeDeadEnds --output results.xls
```

## Evaluation signals

- The output spreadsheet contains a populated 'chokepoints' sheet with reaction identifiers and their produced/consumed metabolites
- For the airway epithelial model (iHsaEC21_SARS-CoV-2), the FVA-refined, DEM-free bottleneck set matches the 1,598 reactions reported in Supplementary Table S4
- For the macrophage model (iAB-AMØ-1410-based), the computed bottleneck set matches the 1,595 reactions in Table S3, with matching set size and identifier overlap
- Reactions classified as 'dead' after FVA refinement are excluded from chokepoint computation
- Every reported bottleneck reaction involving a metabolite formerly at a dead end has been removed or balanced by at least one other consuming/producing reaction after DEM removal

## Limitations

- The method identifies in silico drug target candidates only; no experimental validation of predicted targets (e.g., statins, opaganib, glutathione) was performed in the study
- Bottleneck counts differ across model variants (input, DEM-free, FVA-refined, FVA-refined and DEM-free), so the four variants must be computed explicitly if comparability is required
- Reaction reversibility classification depends entirely on FVA-derived flux bounds, so solver choice and model bounds influence results

## Evidence

- [methods] FVA-refined model variant with updated flux bounds: [evidence span withheld — anchor status: not_found]
- [methods] Remove DEM reactions before chokepoint computation: [evidence span withheld — anchor status: not_found]
- [readme] findCPcli input/output contract: "findCPcli takes as input an SBML files of genome-scale models, and provides as output a spreadsheet file with the results of the chokepoint"
- [intro] DEM exclusion rationale: "Because each bottleneck reaction should be balanced by at least one other reaction that produces or consumes that metabolite, we have excluded reactions in the model with dead-end metabolites."
- [intro] FVA determines reversibility: "FVA is used to compute flux bounds of the reactions, and in turn, to determine whether a given reaction is reversible or not."
