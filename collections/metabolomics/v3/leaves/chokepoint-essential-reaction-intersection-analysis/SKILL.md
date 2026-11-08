---
name: chokepoint-essential-reaction-intersection-analysis
description: Use when you have an integrated host–pathogen genome-scale metabolic
  model plus experimental PPI data (e.g. Gordon et al 2020 SARS-CoV-2 PPIs) and need
  to pinpoint which host metabolic reactions hijacked via PPIs are also network bottlenecks
  or essential for viral growth.
when_to_use_negative:
- The PPI data cannot be mapped onto model enzymes via GPR associations (no gene identifiers
  shared between the PPI dataset and the model)
- The metabolic model has not had dead-end metabolites handled or flux bounds refined,
  so chokepoint/bottleneck sets are unreliable for intersection analysis
- The goal is de novo drug-target prediction from network topology alone without experimental
  PPI evidence (use plain chokepoint/bottleneck analysis instead)
edam_operation: http://edamontology.org/operation_3660
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_2259
tools:
- name: findCPcli
  role: Computes chokepoint, bottleneck, essential reaction, and dead-end metabolite
    sheets, and updates flux bounds via FVA; its comparison spreadsheet is used to
    reconcile intersection membership across the 4 model variants
  repo: https://github.com/findCP/findCPcli
- name: COBRApy
  role: Loads and manipulates the iHsaEC21+SARS-CoV-2 SBML model and provides the
    GPR associations used to map PPI human proteins onto reactions
  repo: https://github.com/opencobra/cobrapy
- name: GLPK
  role: Linear-programming solver used by COBRApy for FBA/FVA simulations underlying
    the essential and chokepoint computations
- name: Python
  role: Scripting language for the intersection workflow (loading model and spreadsheet,
    mapping PPIs, exporting the CSV node table)
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.26508/lsa.202000954
    title: Integrated human/SARS-CoV-2 metabolic models present novel treatment strategies
      against COVID-19
  source_content_hash: sha256:02cb11b8a5adda3b198bc39ca5f221943e205795ba9cf1726a925dce6155f786
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
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_2259
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# chokepoint-essential-reaction-intersection-analysis

## Summary

A three-step labelling procedure that identifies reactions at the intersection nodes between virus–host protein–protein interactions (PPI) and metabolic pathways by crossing PPI-mapped human metabolic enzymes against bottleneck/chokepoint and essential reaction sets computed with findCPcli on the iHsaEC21+SARS-CoV-2 model.

## When to use

Use when you have an integrated host–pathogen genome-scale metabolic model plus experimental PPI data (e.g. Gordon et al 2020 SARS-CoV-2 PPIs) and need to pinpoint which host metabolic reactions hijacked via PPIs are also network bottlenecks or essential for viral growth.

## When NOT to use

- The PPI data cannot be mapped onto model enzymes via GPR associations (no gene identifiers shared between the PPI dataset and the model)
- The metabolic model has not had dead-end metabolites handled or flux bounds refined, so chokepoint/bottleneck sets are unreliable for intersection analysis
- The goal is de novo drug-target prediction from network topology alone without experimental PPI evidence (use plain chokepoint/bottleneck analysis instead)

## Inputs

- SBML genome-scale metabolic model of the integrated host cell + SARS-CoV-2 (BioModels MODEL2007210001, iHsaEC21+SARS-CoV-2)
- findCPcli results spreadsheet with chokepoint, essential reaction, and comparison sheets from 4 model variants (original, no-DEM, FVA-refined, FVA-refined-no-DEM)
- Experimentally characterized human/SARS-CoV-2 protein–protein interaction dataset (Gordon et al, 2020)

## Outputs

- P-Pi intersection node table (e.g. table1_ppi_pathway_nodes.csv) with enzyme, viral interaction partner, and pathway annotation columns
- Set of intersection reactions annotated with membership in the 48 hijacked metabolic pathways of iHsaEC21

## How to apply

Load the integrated SBML model (e.g. iHsaEC21+SARS-CoV-2, BioModels MODEL2007210001) with COBRApy/Python and load the findCPcli results spreadsheet containing the chokepoint, essential reaction, and comparison sheets computed on the four model variants (original, no-DEM, FVA-refined, FVA-refined-no-DEM). Map the human proteins from the PPI dataset onto model enzymes using the gene-to-reaction (GPR) associations of iHsaEC21 (4,660 genes, 3,752 reactions). Select the reactions catalysed by PPI-mapped enzymes that also appear in the bottleneck/chokepoint set and/or the essential reaction set, reconciling membership across the four variants via the 'comparison' sheet. Annotate each selected intersection reaction with its viral interaction partner and its membership in the 48 metabolic pathways of iHsaEC21, then export the node table (enzyme, viral partner, pathway annotation columns) for reuse as a drug-target candidate list.

## Related tools

- **findCPcli** (Computes chokepoint, bottleneck, essential reaction, and dead-end metabolite sheets, and updates flux bounds via FVA; its comparison spreadsheet is used to reconcile intersection membership across the 4 model variants) — https://github.com/findCP/findCPcli
- **COBRApy** (Loads and manipulates the iHsaEC21+SARS-CoV-2 SBML model and provides the GPR associations used to map PPI human proteins onto reactions) — https://github.com/opencobra/cobrapy
- **GLPK** (Linear-programming solver used by COBRApy for FBA/FVA simulations underlying the essential and chokepoint computations)
- **Python** (Scripting language for the intersection workflow (loading model and spreadsheet, mapping PPIs, exporting the CSV node table))

## Examples

```
findCPcli --input MODEL2007210001.xml --output results.xlsx --computeChokepoints --findEssentialReactions --removeDeadEnds --fva
```

## Evaluation signals

- The output table's pathway set matches the 48 hijacked metabolic pathways reported in Table 1 of the paper (expert review / llm_judge against the reported pathway list)
- Every node row carries non-empty enzyme, viral-partner, and pathway fields (field_present check)
- Each selected reaction is present in the bottleneck/chokepoint set and/or the essential reaction set, verifiable against the findCPcli 'comparison' sheet across the 4 model variants
- Every intersection reaction maps to a human enzyme from the Gordon et al (2020) PPI dataset via iHsaEC21 GPR associations
- Model instantiation is reproducible: iHsaEC21 loads without error on COBRA version 0.16.0 and the SBML is BioModels MODEL2007210001

## Limitations

- Bottleneck analysis requires excluding reactions with dead-end metabolites; results therefore depend on which of the 4 model variants (original, no-DEM, FVA-refined, FVA-refined-no-DEM) is used, hence the comparison sheet
- No version number reported for MEMOTE quality scoring of iHsaEC21 (score 51%), and no repository/accession identifiers are provided for the iHsaEC21 model itself beyond BioModels
- Intersection nodes are limited to PPI-mapped enzymes; reactions hijacked by non-enzymatic viral interactions will be missed

## Evidence

- [intro] Three-step labelling of PPi intersection reactions: "The following steps were taken to label the reactions which occur at the nodes of intersections between the referenced points above: (1) we initially identified both the bottleneck and essential"
- [intro] Gordon PPI mapping onto models: "We have mapped the experimentally characterized human/SARS-CoV-2 protein–protein interaction data from Gordon et al (2020) on the in silico virus-integrated human macrophage and epithelial cells."
- [readme] findCPcli scope: "In addition to the computation of chokepoints, findCPcli can compute and remove dead-end metabolites, find essential reactions and update the flux bounds of the reactions according to the results of"
- [methods] Model simulation stack: "We manipulated and simulated the model, including the addition of the gene expression data, with COBRApy (Ebrahim et al, 2013) and GLPK"
- [methods] Model variants for comparison: "ii) model without DEM; iii) model refined with FVA, i.e. with flux bounds updated according to FVA; and iv) model refined with FVA and without DEM"
