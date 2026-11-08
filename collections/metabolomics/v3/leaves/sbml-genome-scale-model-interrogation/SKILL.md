---
name: sbml-genome-scale-model-interrogation
description: Use when input is an SBML genome-scale metabolic model (possibly virus-integrated)
  and the goal is structural/dynamical analysis — chokepoints, bottlenecks, essential
  reactions, FVA flux bounds, or PPI-to-enzyme mapping for drug-target prediction.
when_to_use_negative:
- The input is not an SBML constraint-based metabolic model (e.g. it is a pathway
  graph, PPI network alone, or expression matrix without a GEM) — chokepoint/bottleneck
  topology reasoning requires the reaction network.
- The aim is kinetic or dynamic simulation of metabolism rather than constraint-based
  (FBA/FVA) structural analysis, which is what COBRApy/findCPcli provide.
- You lack solver support (GLPK) or the model cannot be instantiated — e.g. the study
  validated instantiation specifically on COBRA 0.16.0.
edam_operation: http://edamontology.org/operation_3660
edam_topics:
- http://edamontology.org/topic_2259
- http://edamontology.org/topic_0602
tools:
- name: COBRApy
  role: Loads and simulates the SBML GEM (including addition of gene expression data)
    with GLPK as solver; validated on version 0.16.0
  repo: https://github.com/opencobra/cobrapy
- name: findCPcli
  role: Computes chokepoint reactions, removes/finds dead-end metabolites, finds essential
    reactions, and updates flux bounds according to FVA; outputs a results spreadsheet
  repo: https://github.com/findCP/findCPcli
- name: GLPK
  role: Linear-programming solver used with COBRApy for FBA and FVA simulations
- name: MEMOTE
  role: Standardised quality-control scoring of the genome-scale metabolic model (iHsaEC21
    scored 51%)
- name: Python
  role: Language for driving COBRApy and findCPcli in the interrogation workflow
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.26508/lsa.202000954
    title: Integrated human/SARS-CoV-2 metabolic models present novel treatment strategies
      against COVID-19
  source_content_hash: sha256:4ba33b21bb4abd85b0a5c949c44b911588765d59867ddaeed83890f05828bb35
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/opencobra/cobrapy
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
n_claims: 9
---

# sbml-genome-scale-model-interrogation

## Summary

Load and interrogate an SBML genome-scale metabolic model (e.g. the iHsaEC21+SARS-CoV-2 model, BioModels MODEL2007210001) with COBRApy/GLPK and compute chokepoint, essential-reaction, dead-end-metabolite, and FVA-refined analyses with findCPcli to identify host-dependency factors and drug targets.

## When to use

Use when input is an SBML genome-scale metabolic model (possibly virus-integrated) and the goal is structural/dynamical analysis — chokepoints, bottlenecks, essential reactions, FVA flux bounds, or PPI-to-enzyme mapping for drug-target prediction.

## When NOT to use

- The input is not an SBML constraint-based metabolic model (e.g. it is a pathway graph, PPI network alone, or expression matrix without a GEM) — chokepoint/bottleneck topology reasoning requires the reaction network.
- The aim is kinetic or dynamic simulation of metabolism rather than constraint-based (FBA/FVA) structural analysis, which is what COBRApy/findCPcli provide.
- You lack solver support (GLPK) or the model cannot be instantiated — e.g. the study validated instantiation specifically on COBRA 0.16.0.

## Inputs

- SBML file of a genome-scale metabolic model (e.g. BioModels MODEL2007210001, iHsaEC21 or macrophage iAB-AMØ-1410)
- Gene expression data (e.g. from GEO) for context-specificity refinement
- Protein–protein interaction dataset (Gordon et al 2020 human/SARS-CoV-2)
- Gene-to-reaction (GPR) associations of the model

## Outputs

- Spreadsheet with chokepoint, essential-reaction, and comparison sheets across model variants (findCPcli output)
- FVA flux bounds and reversible/irreversible reaction classification
- Bottleneck/chokepoint reaction sets after dead-end-metabolite removal
- Intersection node table (e.g. table1_ppi_pathway_nodes.csv) with enzyme, viral partner, and pathway annotation columns
- MEMOTE quality score report

## How to apply

Load the SBML model (e.g. iHsaEC21 with 4,660 genes, 3,614 reactions, 4,052 metabolites, or its SARS-CoV-2-integrated variant from BioModels MODEL2007210001) with COBRApy 0.16.0 using GLPK as the solver, and manipulate/simulate including addition of gene expression data. Run findCPcli on up to four model variants — (i) original, (ii) without dead-end metabolites (DEM), (iii) refined with FVA (flux bounds updated per FVA), and (iv) FVA-refined without DEM — to obtain a spreadsheet with chokepoint, essential-reaction, and comparison sheets; remove reactions with dead-end metabolites before bottleneck analysis because each bottleneck must be balanced by at least one other reaction consuming/producing that metabolite. Use Flux Balance Analysis to compute host biomass maintenance and maximum viral growth, and FVA flux bounds to determine reaction reversibility and flux-dependent bottleneck reactions. Map external PPI data (Gordon et al 2020 human/SARS-CoV-2 interactions) onto model enzymes via GPR associations, then select reactions catalysed by PPI-mapped enzymes that also lie in the bottleneck/chokepoint and/or essential reaction sets, reconciling across the four variants via the 'comparison' sheet. Quality-check the model with MEMOTE (iHsaEC21 scores 51%) before analysis.

## Related tools

- **COBRApy** (Loads and simulates the SBML GEM (including addition of gene expression data) with GLPK as solver; validated on version 0.16.0) — https://github.com/opencobra/cobrapy
- **findCPcli** (Computes chokepoint reactions, removes/finds dead-end metabolites, finds essential reactions, and updates flux bounds according to FVA; outputs a results spreadsheet) — https://github.com/findCP/findCPcli
- **GLPK** (Linear-programming solver used with COBRApy for FBA and FVA simulations)
- **MEMOTE** (Standardised quality-control scoring of the genome-scale metabolic model (iHsaEC21 scored 51%))
- **Python** (Language for driving COBRApy and findCPcli in the interrogation workflow)

## Examples

```
python -c "import cobra; m = cobra.io.read_sbml_model('MODEL2007210001.xml'); sol = cobra.flux_analysis.flux_variability_analysis(m)"; findCPcli compute_chokepoints --input MODEL2007210001.xml
```

## Evaluation signals

- The model instantiates without error in COBRA (version 0.16.0) and passes a MEMOTE quality check (iHsaEC21 reference score: 51%).
- findCPcli produces a spreadsheet containing chokepoint, essential-reaction, and comparison sheets for all four model variants (original, no-DEM, FVA-refined, FVA-refined-no-DEM).
- Every bottleneck reaction retained is balanced by at least one other reaction producing/consuming the shared metabolite, confirming dead-end-metabolite reactions were excluded.
- Intersection node table rows carry non-empty enzyme, viral-partner, and pathway fields, and the pathway set matches the 48 hijacked metabolic pathways reported in the paper's Table 1.
- Model statistics match expectations: iHsaEC21 contains 4,660 genes, 3,614 reactions, and 4,052 metabolites.

## Limitations

- MEMOTE version was not reported for the 51% quality score, complicating reproducibility of the quality assessment.
- No experimental validation of the in silico drug-target predictions (e.g. statins, opaganib, glutathione) was presented in the study.
- Reactions with dead-end metabolites must be removed or accounted for before bottleneck analysis; leaving them in distorts chokepoint/bottleneck results.
- No repository or accession identifiers were provided for the iHsaEC21 or integrated models beyond the BioModels entry for the integrated model.

## Evidence

- [methods] Load and simulate SBML GEM with COBRApy/GLPK: "We manipulated and simulated the model, including the addition of the gene expression data, with COBRApy (Ebrahim et al, 2013) and GLPK"
- [readme] findCPcli capabilities: "findCPcli can compute and remove dead-end metabolites, find essential reactions and update the flux bounds of the reactions according to the"
- [methods] Four model variants for analysis: "ii) model without DEM; iii) model refined with FVA, i.e. with flux bounds updated according to FVA; and iv) model refined with FVA and without DEM"
- [intro] Rationale for removing DEM reactions before bottleneck analysis: "Because each bottleneck reaction should be balanced by at least one other reaction that produces or consumes that metabolite, we have excluded"
- [discussion] MEMOTE quality score as validation signal: "iHsaEC21 now has a MEMOTE quality score of 51% (Supplemental Data 1)"
- [methods] Model instantiation check on COBRA 0.16.0: "iHsaEC21, can be instantiated without error on the COBRA software (version 0.16.0) (Ebrahim et al, 2013)"
