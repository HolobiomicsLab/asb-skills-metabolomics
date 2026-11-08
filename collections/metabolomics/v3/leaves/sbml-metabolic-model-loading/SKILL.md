---
name: sbml-metabolic-model-loading
description: Use when the input is an SBML genome-scale metabolic model (e.g. iHsaEC21
  or the macrophage iAB-AMØ-1410 SARS-CoV-2 integrated models) that must be parsed
  and instantiated in Python before FBA/FVA/bottleneck analyses.
when_to_use_negative:
- The input is not an SBML genome-scale model (e.g. gene expression tables or protein–protein
  interaction data), which are added only after model instantiation
- The analysis only requires model quality scoring rather than simulation — use MEMOTE
  checks instead of loading into a solver-driven pipeline
- A non-Python constraint-based framework is mandated, since the paper's workflow
  depends on the COBRApy toolbox with GLPK
edam_operation: http://edamontology.org/operation_3660
tools:
- name: COBRApy
  role: Python toolbox used to manipulate and simulate the model, i.e. parse and instantiate
    the SBML genome-scale metabolic model
  repo: https://github.com/opencobra/cobrapy
- name: GLPK
  role: Linear programming solver used alongside COBRApy when instantiating and simulating
    the loaded models
- name: findCPcli
  role: Command-line wrapper that parses the SBML input via COBRApy and exports chokepoint/essential-reaction
    results to a spreadsheet
  repo: https://github.com/findCP/findCPcli
provenance:
  source_task_ids:
  - task_002
  - task_003
  - task_004
  - task_005
  source_papers:
  - doi: 10.26508/lsa.202000954
    title: Integrated human/SARS-CoV-2 metabolic models present novel treatment strategies
      against COVID-19
  source_content_hash: sha256:7072b3c65010b050d36f6d93df1220320ba293b7239a83b6c3ef2ee3bb4f3462
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
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# sbml-metabolic-model-loading

## Summary

Load genome-scale metabolic models in SBML format into COBRApy (backed by the GLPK LP solver) so they can be instantiated without error for downstream constraint-based analyses such as FBA, FVA, chokepoint and essential-reaction computations.

## When to use

Use when the input is an SBML genome-scale metabolic model (e.g. iHsaEC21 or the macrophage iAB-AMØ-1410 SARS-CoV-2 integrated models) that must be parsed and instantiated in Python before FBA/FVA/bottleneck analyses.

## When NOT to use

- The input is not an SBML genome-scale model (e.g. gene expression tables or protein–protein interaction data), which are added only after model instantiation
- The analysis only requires model quality scoring rather than simulation — use MEMOTE checks instead of loading into a solver-driven pipeline
- A non-Python constraint-based framework is mandated, since the paper's workflow depends on the COBRApy toolbox with GLPK

## Inputs

- Genome-scale metabolic model in SBML format (e.g. iHsaEC21+SARS-CoV-2 from BioModels MODEL2007210001; macrophage iAB-AMØ-1410/SARS-CoV-2 model from Renz et al, 2020)

## Outputs

- In-memory COBRApy model object ready for FBA/FVA simulations
- findCPcli results spreadsheet (results.xls) derived from the loaded model

## How to apply

Instantiate the SBML file with COBRApy (version 0.16.0 as used in the paper), specifying GLPK as the linear programming solver. The epithelial model iHsaEC21 is available from BioModels as MODEL2007210001, while the macrophage/SARS-CoV-2 model comes from Renz et al (2020); both must load without error before analysis. findCPcli wraps this parsing step via COBRApy, so models can be passed directly to 'findCPcli -i model.xml -o results.xls' for chokepoint/essential-reaction computations. Rationale: a successful load plus solver availability is the prerequisite for all downstream steps (FVA-refined bounds, dead-end metabolite removal, biomass objective FBA), and the paper validates instantiation by requiring the model to be created 'without error' in COBRA. Always confirm the loaded model's scale against published values (revised iHsaEC21: 4,660 genes, 3,614 reactions, 4,052 metabolites) to catch truncation or parse failures.

## Related tools

- **COBRApy** (Python toolbox used to manipulate and simulate the model, i.e. parse and instantiate the SBML genome-scale metabolic model) — https://github.com/opencobra/cobrapy
- **GLPK** (Linear programming solver used alongside COBRApy when instantiating and simulating the loaded models)
- **findCPcli** (Command-line wrapper that parses the SBML input via COBRApy and exports chokepoint/essential-reaction results to a spreadsheet) — https://github.com/findCP/findCPcli

## Examples

```
findCPcli -i model.xml -o results.xls
```

## Evaluation signals

- iHsaEC21 'can be instantiated without error on the COBRA software (version 0.16.0)' — no parse or import exceptions raised
- Loaded iHsaEC21 model matches the reported scale: 4,660 genes, 3,614 reactions, 4,052 metabolites
- Host-optimum FBA on the loaded epithelial and macrophage models reproduces 0.012 h−1 and 0.0269 host biomass maintenance before any virus steps
- findCPcli output spreadsheet contains a populated 'chokepoints' sheet whose reaction identifiers are consistent with the FVA-refined model
- Model file provenance is traceable: iHsaEC21 loads from BioModels accession MODEL2007210001

## Limitations

- No repository or accession identifiers other than BioModels MODEL2007210001 are given for the revised iHsaEC21 or fully integrated models in the discussion, complicating reproducible loading
- Specific COBRApy version (0.16.0) is required for error-free instantiation of iHsaEC21; newer versions may break parsing
- SBML files must be complete genome-scale models; findCPcli takes SBML as input and any parse failure cascades to all downstream chokepoint/essential-reaction results

## Evidence

- [methods] COBRApy instantiation requirement: "iHsaEC21, can be instantiated without error on the COBRA software (version 0.16.0) (Ebrahim et al, 2013)"
- [methods] Simulation tooling: "We manipulated and simulated the model, including the addition of the gene expression data, with COBRApy (Ebrahim et al, 2013) and GLPK"
- [readme] findCPcli input/output contract: "findCPcli takes as input an SBML files of genome-scale models, and provides as output a spreadsheet file with the results of the chokepoint"
- [abstract] Model availability: "The model is available in BioModels ( https://www.ebi.ac.uk/biomodels/MODEL2007210001 )"
