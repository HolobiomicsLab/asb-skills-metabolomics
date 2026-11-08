---
name: genome-scale-metabolic-model-simulation
description: Use when an SBML-encoded genome-scale metabolic model (e.g. BioModels
  MODEL2007210001, iHsaEC21, iAB-AMØ-1410) must be loaded, FBA-simulated for host/virus
  biomass objectives, or FVA-refined so reaction reversibility and flux ranges are
  determined.
when_to_use_negative:
- The model is not in SBML format or cannot be instantiated in COBRApy 0.16.0 — fix
  encoding first rather than simulating an unloadable model
- The analysis target is kinetic or dynamical simulation rather than steady-state
  constraint-based analysis (FBA/FVA), which is what COBRApy/GLPK and findCPcli provide
- Only pathway enrichment or gene-set analysis is needed — no flux computation is
  required
edam_operation: http://edamontology.org/operation_3660
edam_topics:
- http://edamontology.org/topic_2259
- http://edamontology.org/topic_3400
tools:
- name: COBRApy
  role: Loads the SBML GEM and runs FBA and FVA simulations (version 0.16.0)
  repo: https://github.com/opencobra/cobrapy
- name: GLPK
  role: Linear programming solver underlying the FBA/FVA optimization problems
- name: findCPcli
  role: Generates FVA-refined model variants (-sF option) and exports analysis spreadsheets
    such as essential-reaction results
  repo: https://github.com/findCP/findCPcli
provenance:
  source_task_ids:
  - task_001
  - task_004
  - task_008
  source_papers:
  - doi: 10.26508/lsa.202000954
    title: Integrated human/SARS-CoV-2 metabolic models present novel treatment strategies
      against COVID-19
  source_content_hash: sha256:22bd0d5468de925849659494a7e2f104b082932664e0641c9cb4bd18913a4720
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
  - http://edamontology.org/topic_2259
  - http://edamontology.org/topic_3400
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# genome-scale-metabolic-model-simulation

## Summary

Simulate genome-scale metabolic models (GEMs) in SBML format with COBRApy 0.16.0 and the GLPK linear solver, running Flux Balance Analysis (FBA) for biomass/growth objectives and Flux Variability Analysis (FVA) to compute per-reaction flux bounds, as done for the integrated iHsaEC21+SARS-CoV-2 model.

## When to use

Use when an SBML-encoded genome-scale metabolic model (e.g. BioModels MODEL2007210001, iHsaEC21, iAB-AMØ-1410) must be loaded, FBA-simulated for host/virus biomass objectives, or FVA-refined so reaction reversibility and flux ranges are determined.

## When NOT to use

- The model is not in SBML format or cannot be instantiated in COBRApy 0.16.0 — fix encoding first rather than simulating an unloadable model
- The analysis target is kinetic or dynamical simulation rather than steady-state constraint-based analysis (FBA/FVA), which is what COBRApy/GLPK and findCPcli provide
- Only pathway enrichment or gene-set analysis is needed — no flux computation is required

## Inputs

- SBML file of a genome-scale metabolic model (e.g. BioModels MODEL2007210001, integrated iHsaEC21+SARS-CoV-2, iAB-AMØ-1410 macrophage model)
- Objective specification (host biomass maintenance function or viral biomass maintenance function)
- Optional flux constraint (e.g. vgrowth = μmax from a prior FBA run)

## Outputs

- FBA solution with host-optimal growth rate μmax and virus-optimal growth rate
- Per-reaction lower/upper flux bounds from FVA for every reaction at steady state
- FVA-refined SBML model (model_fva.xml) with bounds replaced by FVA min/max ranges
- Flux comparison table (e.g. flux_comparison.xls) with host vs virus fluxes and percentage increases per reaction/pathway

## How to apply

Download the SBML GEM (e.g. the integrated iHsaEC21+SARS-CoV-2 model, 3,752 reactions, 3,914 metabolites, 4,660 genes, 48 pathways) from BioModels and instantiate it with COBRApy 0.16.0 using the GLPK solver, verifying it loads without error. Run FBA (Orth et al 2010) with the host biomass objective to obtain the host-optimal growth rate μmax, and with the viral biomass maintenance function to obtain the virus-optimal solution. Then run FVA to compute the lower and upper steady-state flux bounds of every reaction, optionally constraining vgrowth = μmax when comparing host-optimum versus virus-optimum conditions. An FVA-refined variant of the model can be produced where each reaction's bounds are replaced by its FVA min/max range (findCPcli '-sF' option), which is required before downstream essential-reaction or chokepoint computations. Validate that the refined model reloads in COBRApy and that every reaction's [lower, upper] bounds equal its FVA-computed flux range.

## Related tools

- **COBRApy** (Loads the SBML GEM and runs FBA and FVA simulations (version 0.16.0)) — https://github.com/opencobra/cobrapy
- **GLPK** (Linear programming solver underlying the FBA/FVA optimization problems)
- **findCPcli** (Generates FVA-refined model variants (-sF option) and exports analysis spreadsheets such as essential-reaction results) — https://github.com/findCP/findCPcli

## Examples

```
findCPcli -i model.xml -o results.xls
```

## Evaluation signals

- model_fva.xml loads without error in COBRApy 0.16.0
- Every reaction's [lower, upper] bounds in the refined model equal its FVA-computed min/max flux range
- Host-optimum FBA of the iHsaEC21 network reproduces the reported μmax for the 3,752-reaction model
- Under virus-optimum conditions, FVA shows zero host biomass maintenance flux, confirming host non-viability at maximum viral growth
- Downstream essential-reaction counts on FVA-refined models reproduce the reported 10 (iHsaEC21) and 52 (iAB-AMØ-1410) essential reactions (Tables S1 and S2)

## Limitations

- FVA-refined bounds are only valid under the specific medium/objective constraints used for the FVA run; changing the objective invalidates the refinement
- Reactions with dead-end metabolites must be excluded from downstream bottleneck/chokepoint analyses because each bottleneck reaction should be balanced by at least one other reaction
- Results depend on solver choice (GLPK) and COBRApy version (0.16.0); other versions may behave differently

## Evidence

- [intro] FVA is used to compute flux bounds of the reactions, and in turn, to determine whether a given reaction is reversible or not.: "FVA is used to compute flux bounds of the reactions, and in turn, to determine whether a given reaction is reversible or not."
- [methods] Manipulation and simulation with COBRApy and GLPK: "We manipulated and simulated the model, including the addition of the gene expression data, with COBRApy (Ebrahim et al, 2013) and GLPK"
- [readme] findCPcli FVA flux-bound update capability: "find essential reactions and update the flux bounds of the reactions according to the results of Flux Variability Analysis"
- [methods] Model instantiation check in COBRApy 0.16.0: "iHsaEC21, can be instantiated without error on the COBRA software (version 0.16.0)"
