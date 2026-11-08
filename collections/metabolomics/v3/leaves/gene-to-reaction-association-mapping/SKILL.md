---
name: gene-to-reaction-association-mapping
description: Use when experimental protein/gene lists (e.g. host–virus PPI data) must
  be linked to enzymatic reactions in a constraint-based model whose SBML defines
  GPR rules, in order to annotate or filter reactions by their catalysing genes.
when_to_use_negative:
- The external gene identifiers cannot be reconciled with the model's gene naming
  scheme (e.g. Entrez vs. gene symbols), so GPR matching would produce spurious mappings
- The goal is to map interactions to reactions in a model lacking GPR rules or with
  uncurated gene annotations
- Input is already a reaction-level list (e.g. an essential-reaction sheet), where
  no gene-to-reaction resolution is needed
edam_operation: http://edamontology.org/operation_3660
edam_topics:
- http://edamontology.org/topic_2258
- http://edamontology.org/topic_3307
tools:
- name: COBRApy
  role: Parses the SBML model's GPR associations and exposes gene-to-reaction links
    used to map the PPI dataset's human proteins onto enzymatic reactions
  repo: https://github.com/opencobra/cobrapy
- name: Python
  role: Scripting language in which the GPR mapping and intersection with bottleneck/essential
    reaction lists is implemented
- name: findCPcli
  role: Produces the chokepoint, essential-reaction, and comparison spreadsheets whose
    reaction IDs the PPI-mapped reactions are intersected against
  repo: https://github.com/findCP/findCPcli
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.26508/lsa.202000954
    title: Integrated human/SARS-CoV-2 metabolic models present novel treatment strategies
      against COVID-19
  source_content_hash: sha256:c854ceb7af3ef1aad7fd55993a77dac1f970d0b01ed111ba292e9ab4b288d12f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/opencobra/cobrapy
  tools:
  - COBRApy
  - GLPK
  - Python
  - findCPcli
  edam_operation: http://edamontology.org/operation_3660
  edam_topics:
  - http://edamontology.org/topic_2258
  - http://edamontology.org/topic_3307
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# Gene-to-reaction association mapping

## Summary

Maps external gene/protein identifiers (e.g. human proteins from the Gordon et al (2020) SARS-CoV-2 PPI dataset) onto metabolic reactions using the gene–protein–reaction (GPR) associations of a genome-scale model, here iHsaEC21 with 4,660 genes and 3,752 reactions, to link protein-interaction evidence to specific enzymatic reactions.

## When to use

Use when experimental protein/gene lists (e.g. host–virus PPI data) must be linked to enzymatic reactions in a constraint-based model whose SBML defines GPR rules, in order to annotate or filter reactions by their catalysing genes.

## When NOT to use

- The external gene identifiers cannot be reconciled with the model's gene naming scheme (e.g. Entrez vs. gene symbols), so GPR matching would produce spurious mappings
- The goal is to map interactions to reactions in a model lacking GPR rules or with uncurated gene annotations
- Input is already a reaction-level list (e.g. an essential-reaction sheet), where no gene-to-reaction resolution is needed

## Inputs

- Genome-scale metabolic model in SBML format (iHsaEC21+SARS-CoV-2, BioModels MODEL2007210001) with GPR associations (4,660 genes, 3,752 reactions)
- Gene/protein identifier list from an external experimental dataset, e.g. host proteins from the Gordon et al (2020) human/SARS-CoV-2 PPI dataset

## Outputs

- Set of model reactions whose GPRs map to the external gene/protein list
- Annotated intersection table (e.g. table1_ppi_pathway_nodes.csv) with enzyme, viral interaction partner, and pathway annotation columns

## How to apply

Load the genome-scale SBML model (e.g. iHsaEC21+SARS-CoV-2, BioModels MODEL2007210001) with COBRApy so that GPR strings are parsed into gene objects linked to reaction objects. Read the GPR logic (e.g. 'geneA and geneB' or 'geneA or geneB') to decide whether a reaction is catalysed by one or more members of the external protein set — a reaction is mapped if at least one of its genes appears in the external list (OR logic) or if the full AND-complex requirement is satisfied, depending on the downstream use. For this paper's PPi-pathway intersection task, human proteins from Gordon et al (2020) were mapped onto iHsaEC21 enzymes via GPRs, and only reactions catalysed by PPI-mapped enzymes that additionally appear in the findCPcli bottleneck/chokepoint and/or essential-reaction sets were retained. Annotate each mapped reaction with the interacting viral partner and the iHsaEC21 pathway membership before export.

## Related tools

- **COBRApy** (Parses the SBML model's GPR associations and exposes gene-to-reaction links used to map the PPI dataset's human proteins onto enzymatic reactions) — https://github.com/opencobra/cobrapy
- **Python** (Scripting language in which the GPR mapping and intersection with bottleneck/essential reaction lists is implemented)
- **findCPcli** (Produces the chokepoint, essential-reaction, and comparison spreadsheets whose reaction IDs the PPI-mapped reactions are intersected against) — https://github.com/findCP/findCPcli

## Examples

```
python -c "import cobra; m=cobra.io.read_sbml_model('MODEL2007210001.xml'); print({g.id:[r.id for r in g.reactions] for g in m.genes})"
```

## Evaluation signals

- Every exported intersection row carries non-empty enzyme, viral-partner, and pathway fields (field_present check)
- The pathway set of the mapped intersection reactions matches the 48 hijacked metabolic pathways reported in Table 1 of the paper
- Model statistics match the expected iHsaEC21 content (4,660 genes, 3,614–3,752 reactions, 4,052 metabolites) confirming the GPR mapping was done on the correct SBML revision
- Every mapped reaction appears in at least one of the findCPcli bottleneck/chokepoint or essential-reaction sheets across the 4 model variants (original, no-DEM, FVA-refined, FVA-refined-no-DEM), reconciled via the 'comparison' sheet

## Limitations

- Mapping depends on consistent gene identifiers between the external dataset (Gordon et al 2020) and the model's GPR strings; identifier mismatches silently drop mappings
- GPR OR-logic means a single mapped gene marks the whole reaction as interacting even when the full enzyme complex involves unmapped subunits
- Results must be reconciled across the 4 model variants (original, no-DEM, FVA-refined, FVA-refined-no-DEM), since dead-end metabolite removal and FVA flux-bound updates change which reactions exist and are classified as essential/bottleneck

## Evidence

- [other] Map the human proteins from the Gordon et al (2020) PPI dataset onto model enzymes using the gene-to-reaction (GPR) associations of iHsaEC21 (4,660 genes, 3,752 reactions).: [evidence span withheld — anchor status: not_found]
- [intro] We have mapped the experimentally characterized human/SARS-CoV-2 protein–protein interaction data from Gordon et al (2020) on the in silico virus-integrated human macrophage and epithelial cells.: "We have mapped the experimentally characterized human/SARS-CoV-2 protein–protein interaction data from Gordon et al (2020) on the in silico virus-integrated human macrophage and epithelial cells."
- [methods] We manipulated and simulated the model, including the addition of the gene expression data, with COBRApy (Ebrahim et al, 2013) and GLPK: "We manipulated and simulated the model, including the addition of the gene expression data, with COBRApy (Ebrahim et al, 2013) and GLPK"
- [discussion] Our revised iHsaEC21 epithelial cell model contains 4,660 genes, 3,614 reactions, and 4,052 metabolites: "Our revised iHsaEC21 epithelial cell model contains 4,660 genes, 3,614 reactions, and 4,052 metabolites"
