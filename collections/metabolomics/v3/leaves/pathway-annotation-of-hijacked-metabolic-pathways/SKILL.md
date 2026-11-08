---
name: pathway-annotation-of-hijacked-metabolic-pathways
description: Use when you must annotate host metabolic reactions — identified as viral-PPI,
  chokepoint/bottleneck, or essential-reaction intersections — with their metabolic
  pathway membership in iHsaEC21 to catalogue SARS-CoV-2-hijacked pathways.
when_to_use_negative:
- The input is a raw PPI table alone without an integrated host/virus genome-scale
  metabolic model and findCPcli chokepoint/essential-reaction results — no intersection
  can be defined
- The goal is generic pathway-enrichment statistics on a gene list rather than annotating
  specific bottleneck/essential reaction nodes with viral partners
- The metabolic model lacks GPR associations, so PPI-mapped human proteins cannot
  be traced to catalysed reactions
edam_topics:
- http://edamontology.org/topic_0602
- http://edamontology.org/topic_2258
tools:
- name: findCPcli
  role: computes chokepoints, essential reactions, and dead-end metabolite removal;
    produces the results spreadsheet whose sheets are reconciled across model variants
    for node selection
  repo: https://github.com/findCP/findCPcli
- name: COBRApy
  role: loads and manipulates the SBML integrated model and provides GPR associations
    used to map PPI-mapped human proteins onto catalysed reactions
  repo: https://github.com/opencobra/cobrapy
- name: GLPK
  role: LP solver underlying the COBRApy model manipulations and simulations
- name: Python
  role: scripting environment for loading the model, intersecting reaction sets, and
    exporting the annotated node table
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.26508/lsa.202000954
    title: Integrated human/SARS-CoV-2 metabolic models present novel treatment strategies
      against COVID-19
  source_content_hash: sha256:b796370b2a529667c9c6e9dd1ea24fda5819c3f1fce68feb24b5248191eb4446
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/findCP/findCPcli
  tools:
  - COBRApy
  - GLPK
  - Python
  - findCPcli
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0602
  - http://edamontology.org/topic_2258
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# pathway-annotation-of-hijacked-metabolic-pathways

## Summary

A procedure for labelling reactions at PPi (protein–protein-interaction/pathway) intersection nodes with their iHsaEC21 pathway membership, viral interaction partner, and catalysing enzyme, yielding the set of metabolic pathways hijacked by SARS-CoV-2 (48 of 334 human metabolic pathways). It links experimental PPI data (Gordon et al, 2020) to the in silico integrated host/virus genome-scale metabolic models.

## When to use

Use when you must annotate host metabolic reactions — identified as viral-PPI, chokepoint/bottleneck, or essential-reaction intersections — with their metabolic pathway membership in iHsaEC21 to catalogue SARS-CoV-2-hijacked pathways.

## When NOT to use

- The input is a raw PPI table alone without an integrated host/virus genome-scale metabolic model and findCPcli chokepoint/essential-reaction results — no intersection can be defined
- The goal is generic pathway-enrichment statistics on a gene list rather than annotating specific bottleneck/essential reaction nodes with viral partners
- The metabolic model lacks GPR associations, so PPI-mapped human proteins cannot be traced to catalysed reactions

## Inputs

- SBML file of the integrated iHsaEC21+SARS-CoV-2 model (BioModels MODEL2007210001)
- findCPcli results spreadsheet with chokepoint, essential-reaction, and comparison sheets from 4 model variants (original, no-DEM, FVA-refined, FVA-refined-no-DEM)
- Experimentally characterized human/SARS-CoV-2 protein–protein interaction dataset (Gordon et al, 2020)
- iHsaEC21 gene-to-reaction (GPR) association map (4,660 genes, 3,752 reactions)

## Outputs

- table1_ppi_pathway_nodes.csv with enzyme, viral partner, and pathway annotation columns per intersection node
- Set of metabolic pathways hijacked by SARS-CoV-2 (48 pathways of iHsaEC21, out of 334 in the human metabolic network)

## How to apply

Load the iHsaEC21+SARS-CoV-2 SBML model (BioModels MODEL2007210001) with COBRApy/Python and the findCPcli results spreadsheet containing chokepoint, essential-reaction, and comparison sheets computed on the 4 model variants (original, no-DEM, FVA-refined, FVA-refined-no-DEM). Map human proteins from the Gordon et al (2020) PPI dataset onto model enzymes via iHsaEC21's gene-to-reaction (GPR) associations (4,660 genes, 3,752 reactions). Select reactions catalysed by PPI-mapped enzymes that also appear in the bottleneck/chokepoint set and/or essential reaction set, using the 'comparison' sheet to reconcile membership across the 4 model variants. Annotate each intersection reaction with its viral interaction partner and its membership among the 48 hijacked metabolic pathways of iHsaEC21, then export the node table with enzyme, viral-partner, and pathway columns (table1_ppi_pathway_nodes.csv).

## Related tools

- **findCPcli** (computes chokepoints, essential reactions, and dead-end metabolite removal; produces the results spreadsheet whose sheets are reconciled across model variants for node selection) — https://github.com/findCP/findCPcli
- **COBRApy** (loads and manipulates the SBML integrated model and provides GPR associations used to map PPI-mapped human proteins onto catalysed reactions) — https://github.com/opencobra/cobrapy
- **GLPK** (LP solver underlying the COBRApy model manipulations and simulations)
- **Python** (scripting environment for loading the model, intersecting reaction sets, and exporting the annotated node table)

## Examples

```
import cobra; model = cobra.io.read_sbml_model('MODEL2007210001.xml')
```

## Evaluation signals

- The exported pathway set matches the 48 hijacked metabolic pathways reported in Table 1 of the paper (verified against the reported pathway list)
- Every node row carries non-empty enzyme, viral-partner, and pathway fields (field_present check)
- Intersection membership is consistent when reconciled across the 4 model variants using the findCPcli 'comparison' sheet
- Enzyme-to-reaction mapping respects iHsaEC21 GPR counts (4,660 genes, 3,752 reactions); the paper's revised iHsaEC21 has 4,660 genes, 3,614 reactions, and 4,052 metabolites
- Each annotated pathway is one of the 334 metabolic pathways in the human metabolic network

## Limitations

- Reactions with dead-end metabolites (DEMs) were excluded from bottleneck analysis, so nodes depending on DEM-containing reactions will be missed; results differ across the 4 model variants (original, no-DEM, FVA-refined, FVA-refined-no-DEM)
- Pathway annotation is limited to the pathways represented in iHsaEC21, not the full 334-pathway human metabolic network
- No experimental validation is presented for the derived drug-target/pathway predictions; proposals rest on prior literature

## Evidence

- [intro] 48 hijacked pathways of iHsaEC21: "Of the 334 metabolic pathways in the human metabolic network, 48 pathways including the biosynthesis and degradation pathways of amino acids"
- [intro] Three-step node labelling procedure: "The following steps were taken to label the reactions which occur at the nodes of intersections between the referenced points above: (1) we initially identified both the bottleneck and essential"
- [intro] PPI data mapped onto in silico models: "We have mapped the experimentally characterized human/SARS-CoV-2 protein–protein interaction data from Gordon et al (2020) on the in silico virus-integrated human macrophage and epithelial cells."
- [readme] findCPcli capabilities underlying the node sets: "In addition to the computation of chokepoints, findCPcli can compute and remove dead-end metabolites, find essential reactions and update the flux bounds"
- [methods] Tool used to identify host-dependency factors: "identify the host-dependency factors for the SARS-CoV-2 virus by using the novel software tool findCPcli"
