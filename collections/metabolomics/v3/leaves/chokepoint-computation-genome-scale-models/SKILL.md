---
name: chokepoint-computation-genome-scale-models
description: Use when the goal is to identify potential drug targets in an SBML genome-scale
  metabolic model by computing chokepoint reactions whose flux bounds and dead-end-metabolite
  status have been refined via FVA.
when_to_use_negative:
- The model has not been refined with FVA and DEM removal — chokepoint counts on the
  raw input model differ substantially from the FVA-refined, DEM-free variant and
  are not comparable to the paper's reported sets
- The goal is general pathway enrichment or gene essentiality without flux/topology
  coupling — MEMOTE quality control or plain FBA/FVA alone is the appropriate skill
- The input is not an SBML genome-scale metabolic model parseable by COBRApy (findCPcli
  takes SBML as input)
edam_operation: http://edamontology.org/operation_3660
edam_topics:
- http://edamontology.org/topic_2259
- http://edamontology.org/topic_0602
tools:
- name: findCPcli
  role: Implements the chokepoint computation; computes chokepoints, dead-end metabolites,
    essential reactions, and FVA-updated flux bounds; exports results spreadsheet
  repo: https://github.com/findCP/findCPcli
- name: COBRApy
  role: Loads and manipulates the SBML model for findCPcli's computations
  repo: https://github.com/opencobra/cobrapy
- name: GLPK
  role: LP solver underlying COBRApy/FVA flux bound computation
- name: MEMOTE
  role: Quality control scoring of the genome-scale model before bottleneck analysis
provenance:
  source_task_ids:
  - task_002
  - task_003
  source_papers:
  - doi: 10.26508/lsa.202000954
    title: Integrated human/SARS-CoV-2 metabolic models present novel treatment strategies
      against COVID-19
  source_content_hash: sha256:ceed14d6c9b6032347a34f6fc0c1381d63d71b98d50d2193609cbb516805d874
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/findCP/findCPcli
  tools:
  - COBRApy
  - GLPK
  - MEMOTE
  - Python
  - findCPcli
  edam_operation: http://edamontology.org/operation_3660
  edam_topics:
  - http://edamontology.org/topic_2259
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# chokepoint-computation-genome-scale-models

## Summary

Computes growth-dependent chokepoint (bottleneck) reactions in genome-scale metabolic models using findCPcli, which combines network topology with FVA-derived flux dynamics to flag reactions required for organism/viral growth and survival. Applied here to the integrated human airway epithelial cell/SARS-CoV-2 model (iHsaEC21_SARS-CoV-2) and the macrophage model to predict drug targets.

## When to use

Use when the goal is to identify potential drug targets in an SBML genome-scale metabolic model by computing chokepoint reactions whose flux bounds and dead-end-metabolite status have been refined via FVA.

## When NOT to use

- The model has not been refined with FVA and DEM removal — chokepoint counts on the raw input model differ substantially from the FVA-refined, DEM-free variant and are not comparable to the paper's reported sets
- The goal is general pathway enrichment or gene essentiality without flux/topology coupling — MEMOTE quality control or plain FBA/FVA alone is the appropriate skill
- The input is not an SBML genome-scale metabolic model parseable by COBRApy (findCPcli takes SBML as input)

## Inputs

- Genome-scale metabolic model in SBML format (e.g. iHsaEC21_SARS-CoV-2, BioModels MODEL2007210001; iAB-AMØ-1410-based macrophage/SARS-CoV-2 model)
- COBRApy-parseable model with GLPK LP solver configured

## Outputs

- Results spreadsheet (e.g. results.xls) with a 'chokepoints' sheet listing bottleneck reaction identifiers and the metabolite(s) each produces/consumes
- FVA-refined reaction flux bounds and reaction classifications (reversible, non-reversible, dead)
- Set of growth-dependent bottleneck reactions (e.g. 1,598 in the epithelial model; 1,595 in the macrophage model)

## How to apply

Load the genome-scale model in SBML format (e.g. iHsaEC21_SARS-CoV-2 from BioModels MODEL2007210001, or the iAB-AMØ-1410-based macrophage model) into findCPcli, which parses models via COBRApy with GLPK as the LP solver. Run Flux Variability Analysis to update the lower and upper flux bounds of every reaction, then use these refined bounds to classify reactions as reversible, non-reversible, or dead. Remove reactions with dead-end metabolites (DEM) so each bottleneck reaction is balanced by at least one other consuming/producing reaction. Compute chokepoints by identifying consumer and producer reactions for each metabolite; optionally repeat across all four model variants (input model, DEM-free, FVA-refined, FVA-refined and DEM-free) for comparison, and export results to a spreadsheet containing a 'chokepoints' sheet.

## Related tools

- **findCPcli** (Implements the chokepoint computation; computes chokepoints, dead-end metabolites, essential reactions, and FVA-updated flux bounds; exports results spreadsheet) — https://github.com/findCP/findCPcli
- **COBRApy** (Loads and manipulates the SBML model for findCPcli's computations) — https://github.com/opencobra/cobrapy
- **GLPK** (LP solver underlying COBRApy/FVA flux bound computation)
- **MEMOTE** (Quality control scoring of the genome-scale model before bottleneck analysis)

## Examples

```
pip install findCPcli
```

## Evaluation signals

- Output spreadsheet must contain a 'chokepoints' sheet populated with bottleneck reaction identifiers consistent with the FVA-refined model
- For the epithelial model, the computed bottleneck set should match the 1,598 reactions reported in Supplementary Table S4; for the macrophage model, the 1,595 reactions of Table S3
- Computed chokepoint identifiers should overlap with and cover the paper's reported reaction set across amino acid, fatty acid, carbohydrate, amine, and cofactor pathways
- Reaction classification (reversible/non-reversible/dead) must derive from FVA-updated lower and upper flux bounds, not the original SBML bounds
- No remaining bottleneck reaction should consume or produce a metabolite balanced only by dead-end-metabolite reactions (DEM reactions removed before chokepoint computation)

## Limitations

- Results depend on the model variant chosen — the paper computed chokepoints on all four variants (input model, DEM-free, FVA-refined, FVA-refined and DEM-free), and counts differ across variants
- Reactions with dead-end metabolites must be excluded because each bottleneck reaction should be balanced by at least one other reaction that produces or consumes that metabolite
- In silico drug target predictions (e.g. statins, opaganib, glutathione) were not experimentally validated in the study
- No repository or accession identifiers were provided for the revised iHsaEC21 or integrated models beyond the BioModels entry

## Evidence

- [readme] findCPcli is a command line python-tool for the computation of chokepoint reactions in genome-scale metabolic models.: "findCPcli is a command line python-tool for the computation of chokepoint reactions in genome-scale metabolic models."
- [intro] FVA flux bounds determine reversibility: "FVA is used to compute flux bounds of the reactions, and in turn, to determine whether a given reaction is reversible or not."
- [methods] DEM exclusion rationale: "Because each bottleneck reaction should be balanced by at least one other reaction that produces or consumes that metabolite, we have excluded reactions in the model with dead-end metabolites."
- [abstract] findCPcli availability: "the software tool, findCPcli, that implements the computational method is available at https://github.com/findCP/findCPcli"
