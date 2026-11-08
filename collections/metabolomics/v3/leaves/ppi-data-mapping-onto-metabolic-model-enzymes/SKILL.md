---
name: ppi-data-mapping-onto-metabolic-model-enzymes
description: Use when you have an experimentally derived host–pathogen PPI dataset
  and a genome-scale metabolic model with GPR rules, and you need to identify which
  model reactions are catalysed by enzymes that physically interact with viral proteins.
when_to_use_negative:
- The PPI data are predicted/computational rather than experimentally characterized
  (e.g. Yeast-two-hybrid-free in silico predictions), since the method anchors on
  the Gordon et al (2020) experimental interactions
- The metabolic model lacks gene–reaction (GPR) associations, so viral-interacting
  human proteins cannot be linked to catalysed reactions
- The goal is to compute chokepoints or essential reactions themselves rather than
  to intersect them with PPI-mapped enzymes — use findCPcli's own computations directly
  instead
edam_operation: http://edamontology.org/operation_0276
edam_topics:
- http://edamontology.org/topic_0602
- http://edamontology.org/topic_0736
tools:
- name: COBRApy
  role: Loads and manipulates/simulates the iHsaEC21+SARS-CoV-2 SBML model and provides
    the GPR associations used to map PPI proteins onto reactions
  repo: https://github.com/opencobra/cobrapy
- name: findCPcli
  role: Computes the chokepoint, dead-end metabolite, essential reaction, and FVA-refined
    results whose spreadsheets define the bottleneck/essential reaction sets intersected
    with PPI-mapped enzymes
  repo: https://github.com/findCP/findCPcli
- name: GLPK
  role: Linear programming solver used alongside COBRApy to manipulate and simulate
    the model
- name: Python
  role: Scripting environment (with COBRApy) used to join the PPI-mapped enzyme set
    with chokepoint/essential reaction lists and export the intersection table
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.26508/lsa.202000954
    title: Integrated human/SARS-CoV-2 metabolic models present novel treatment strategies
      against COVID-19
  source_content_hash: sha256:69c1bbfc4df73762fafb91d1b3fe150bca1f513676c17f27cf9c62e847dab173
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/opencobra/cobrapy
  tools:
  - COBRApy
  - GLPK
  - Python
  - findCPcli
  edam_operation: http://edamontology.org/operation_0276
  edam_topics:
  - http://edamontology.org/topic_0602
  - http://edamontology.org/topic_0736
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# ppi-data-mapping-onto-metabolic-model-enzymes

## Summary

Maps experimentally characterized human/SARS-CoV-2 protein–protein interaction data (Gordon et al, 2020) onto the enzymes of an integrated genome-scale metabolic model (iHsaEC21 + SARS-CoV-2, BioModels MODEL2007210001) via gene–reaction (GPR) associations, to label reactions at PPi-pathway intersection nodes as candidate drug targets.

## When to use

Use when you have an experimentally derived host–pathogen PPI dataset and a genome-scale metabolic model with GPR rules, and you need to identify which model reactions are catalysed by enzymes that physically interact with viral proteins.

## When NOT to use

- The PPI data are predicted/computational rather than experimentally characterized (e.g. Yeast-two-hybrid-free in silico predictions), since the method anchors on the Gordon et al (2020) experimental interactions
- The metabolic model lacks gene–reaction (GPR) associations, so viral-interacting human proteins cannot be linked to catalysed reactions
- The goal is to compute chokepoints or essential reactions themselves rather than to intersect them with PPI-mapped enzymes — use findCPcli's own computations directly instead

## Inputs

- SBML file of the integrated human/SARS-CoV-2 genome-scale metabolic model (iHsaEC21 + SARS-CoV-2, BioModels MODEL2007210001)
- Experimentally characterized human/SARS-CoV-2 protein–protein interaction dataset from Gordon et al (2020)
- Gene–reaction (GPR) associations of iHsaEC21 (4,660 genes)
- findCPcli results spreadsheet with chokepoint, essential reaction, and comparison sheets for 4 model variants (original, no-DEM, FVA-refined, FVA-refined-no-DEM)

## Outputs

- PPi-pathway intersection node table (table1_ppi_pathway_nodes.csv) with enzyme, viral partner, and metabolic pathway annotation columns
- Set of reactions at intersection nodes labelled as candidate drug targets, annotated against the 48 hijacked metabolic pathways of iHsaEC21

## How to apply

Load the iHsaEC21+SARS-CoV-2 SBML model (BioModels MODEL2007210001) with COBRApy/Python and manipulate or simulate it with COBRApy 0.16.0 and GLPK. Map the human protein partners from the Gordon et al (2020) PPI dataset onto model enzymes using the gene-to-reaction (GPR) associations of iHsaEC21 (4,660 genes, 3,752 reactions in the task context; 4,660 genes, 3,614 reactions and 4,052 metabolites per the article). Run findCPcli on the SBML input to obtain chokepoint, dead-end metabolite, essential reaction, and FVA-refined results as a spreadsheet, computed on the 4 model variants (original, no-DEM, FVA-refined, FVA-refined-no-DEM). Select reactions catalysed by PPI-mapped enzymes that also appear in the bottleneck/chokepoint set and/or the essential reaction set, using the comparison sheet to reconcile across the 4 variants. Annotate each intersection reaction with its viral interaction partner and its membership in the 48 hijacked metabolic pathways of iHsaEC21 (out of 334 in the human metabolic network), and export as a table with enzyme, viral partner, and pathway columns.

## Related tools

- **COBRApy** (Loads and manipulates/simulates the iHsaEC21+SARS-CoV-2 SBML model and provides the GPR associations used to map PPI proteins onto reactions) — https://github.com/opencobra/cobrapy
- **findCPcli** (Computes the chokepoint, dead-end metabolite, essential reaction, and FVA-refined results whose spreadsheets define the bottleneck/essential reaction sets intersected with PPI-mapped enzymes) — https://github.com/findCP/findCPcli
- **GLPK** (Linear programming solver used alongside COBRApy to manipulate and simulate the model)
- **Python** (Scripting environment (with COBRApy) used to join the PPI-mapped enzyme set with chokepoint/essential reaction lists and export the intersection table)

## Examples

```
import cobra; model = cobra.io.read_sbml_model('MODEL2007210001.xml'); genes = {g.id for g in model.genes}; ppi_rxns = [r for r in model.reactions if any(g.id in ppi_genes for g in r.genes)]
```

## Evaluation signals

- The pathway set in the exported intersection table matches the 48 hijacked metabolic pathways reported in Table 1 of the paper (verified against the reported pathway list)
- Every node row in the output table carries non-empty enzyme, viral-partner, and pathway fields
- Each selected reaction is present in the chokepoint/bottleneck set and/or the essential reaction set in at least one of the 4 model variants, reconciled via the 'comparison' sheet
- Every selected reaction's enzyme is traceable to a human protein in the Gordon et al (2020) PPI dataset through a valid iHsaEC21 GPR association
- The iHsaEC21 model loads without error on COBRA 0.16.0, and counts match the reported scale (4,660 genes; ~3,614 reactions; 4,052 metabolites)

## Limitations

- Results depend on the model variant chosen: dead-end metabolites (DEMs) were removed and flux bounds updated by FVA because each bottleneck reaction should be balanced by at least one other reaction, so intersection sets differ across the original, no-DEM, FVA-refined, and FVA-refined-no-DEM variants
- Only reactions with PPI-mapped enzymes AND bottleneck/essential membership are labelled; PPI interactions not touching metabolic enzymes, or essential reactions lacking experimental PPI evidence, are excluded
- The method is in silico: no experimental validation of the resulting drug-target predictions (e.g. statins, opaganib, glutathione) is presented in the study

## Evidence

- [intro] PPi mapping rationale: "We have mapped the experimentally characterized human/SARS-CoV-2 protein–protein interaction data from Gordon et al (2020) on the in silico virus-integrated human macrophage and epithelial cells."
- [intro] Three-step intersection labelling: "The following steps were taken to label the reactions which occur at the nodes of intersections between the referenced points above: (1) we initially identified both the bottleneck and essential"
- [methods] Model manipulation tools: "We manipulated and simulated the model, including the addition of the gene expression data, with COBRApy (Ebrahim et al, 2013) and GLPK"
- [readme] findCPcli output capabilities: "findCPcli can compute and remove dead-end metabolites, find essential reactions and update the flux bounds of the reactions according to the"
- [supplementary] Tool availability: "The source code of findCPcli together with its documentation is available at https://github.com/findCP/findCPcli"
