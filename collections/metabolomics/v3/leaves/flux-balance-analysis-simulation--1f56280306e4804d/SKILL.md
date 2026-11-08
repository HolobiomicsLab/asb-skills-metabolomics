---
name: flux-balance-analysis-simulation
description: Use when a steady-state genome-scale metabolic model in SBML format needs
  its maximal biomass or growth objective computed, or when host/virus trade-offs
  must be quantified by optimizing alternate biomass objectives.
when_to_use_negative:
- The question concerns reaction reversibility or flux ranges rather than a single
  optimal objective value — use Flux Variability Analysis (FVA) to update flux bounds
  instead
- The model requires prior refinement (e.g. dead-end metabolite removal or FVA-refined
  bounds) before meaningful growth optimization is possible
- The goal is identifying chokepoint or bottleneck drug-target reactions, which findCPcli
  computes from topology plus FVA dynamics rather than a single FBA optimum
edam_operation: http://edamontology.org/operation_3660
edam_topics:
- http://edamontology.org/topic_2258
tools:
- name: COBRApy
  role: Loads the SBML host/virus models, sets and swaps biomass objectives, fixes
    reaction flux bounds, and solves the LP for host and viral growth rates
  repo: https://github.com/opencobra/cobrapy
- name: GLPK
  role: Linear programming solver used by COBRApy to solve the FBA optimization problems
- name: Python
  role: Scripting environment in which the COBRApy FBA simulations and results-table
    assembly are run
provenance:
  source_task_ids:
  - task_005
  - task_001
  source_papers:
  - doi: 10.26508/lsa.202000954
    title: Integrated human/SARS-CoV-2 metabolic models present novel treatment strategies
      against COVID-19
  source_content_hash: sha256:5ae0d3de1145deb5a3a948a0adb34c64d3521326ec8b0a1c83ecb5f755167b19
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/opencobra/cobrapy
  tools:
  - COBRApy
  - GLPK
  - PyCoMo
  - Python
  - Python 3
  - findCPcli
  edam_operation: http://edamontology.org/operation_3660
  edam_topics:
  - http://edamontology.org/topic_2258
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# flux-balance-analysis-simulation

## Summary

Run constraint-based Flux Balance Analysis (FBA) on genome-scale metabolic models (e.g. the iHsaEC21+SARS-CoV-2 and iAB-AMØ-1410+SARS-CoV-2 integrated models) with COBRApy 0.16.0 and the GLPK solver to compute maximal host biomass maintenance and maximum viral growth rates, and to test viral non-viability by fixing viral flux at its optimum.

## When to use

Use when a steady-state genome-scale metabolic model in SBML format needs its maximal biomass or growth objective computed, or when host/virus trade-offs must be quantified by optimizing alternate biomass objectives.

## When NOT to use

- The question concerns reaction reversibility or flux ranges rather than a single optimal objective value — use Flux Variability Analysis (FVA) to update flux bounds instead
- The model requires prior refinement (e.g. dead-end metabolite removal or FVA-refined bounds) before meaningful growth optimization is possible
- The goal is identifying chokepoint or bottleneck drug-target reactions, which findCPcli computes from topology plus FVA dynamics rather than a single FBA optimum

## Inputs

- Genome-scale metabolic model in SBML format (integrated host+virus model, e.g. iHsaEC21+SARS-CoV-2 from BioModels MODEL2007210001)
- Macrophage/SARS-CoV-2 integrated model from Renz et al (2020), iAB-AMØ-1410 based
- Identifiers of the host biomass maintenance reaction and the viral biomass reaction

## Outputs

- Maximal host biomass maintenance rate without virus (h−1 per model)
- Maximum viral biomass growth rate (h−1 per model)
- Host biomass value under fixed-maximum-viral-flux constraint (non-viability check)
- Results table (e.g. fba_growth_results.csv) of growth values and the zero-biomass check

## How to apply

Load the SBML model (e.g. iHsaEC21+SARS-CoV-2 from BioModels MODEL2007210001, or the Renz et al 2020 macrophage/SARS-CoV-2 model) into COBRApy version 0.16.0 using the GLPK linear programming solver. First remove or disable the viral biomass reaction and maximize the host biomass maintenance objective to obtain host growth in the absence of virus (0.012 h−1 for iHsaEC21 epithelial, 0.0269 for iAB-AMØ-1410 macrophage). Next maximize the viral biomass objective to obtain maximum viral growth (0.0181 h−1 epithelial, 0.0144 h−1 macrophage). Finally, fix the viral biomass reaction flux at its computed maximum and re-optimize the host biomass objective: if the host biomass drops to zero in both compartments, the virus renders the host non-viable at maximal viral replication. Assemble the four growth values plus the zero-host-biomass check into a results table for downstream comparison across cell compartments.

## Related tools

- **COBRApy** (Loads the SBML host/virus models, sets and swaps biomass objectives, fixes reaction flux bounds, and solves the LP for host and viral growth rates) — https://github.com/opencobra/cobrapy
- **GLPK** (Linear programming solver used by COBRApy to solve the FBA optimization problems)
- **Python** (Scripting environment in which the COBRApy FBA simulations and results-table assembly are run)

## Examples

```
import cobra; m = cobra.io.read_sbml_model('MODEL2007210001.xml'); m.solver = 'glpk'; sol = m.optimize()
```

## Evaluation signals

- Host-optimum FBA (viral biomass reaction removed) reproduces 0.012 h−1 for the epithelial iHsaEC21 model and 0.0269 for the macrophage iAB-AMØ-1410 model
- Virus-optimum FBA reproduces maximum viral growth of 0.0181 h−1 (epithelial) and 0.0144 h−1 (macrophage)
- Fixing the viral biomass flux at its maximum and re-optimizing host biomass yields exactly zero host biomass in both models (non-viability result)
- The model loads without error in COBRApy version 0.16.0 before simulation
- The four growth values and the zero-biomass check assemble into the expected results table (fba_growth_results.csv) without missing fields

## Limitations

- FBA gives a single optimal objective value per run; flux ranges and reversibility require FVA, and growth-dependent analyses require FVA-refined flux bounds
- Host biomass computed by FBA depends on the medium/constraint settings of the context-specific model; the non-viability result holds only under these constraints
- FBA predictions are in silico; the article provides no experimental validation of growth rates or of the drug-target predictions derived from these models

## Evidence

- [intro] FBA used to compute host biomass maintenance and maximum viral growth rate: "The Flux Balance Analysis (FBA) method (Orth et al, 2010) was used to compute both the biomass maintenance of the cell in the absence of virus and the maximum growth rate of the virus in the cell"
- [methods] COBRApy and GLPK used for model simulation: "We manipulated and simulated the model, including the addition of the gene expression data, with COBRApy (Ebrahim et al, 2013) and GLPK"
- [methods] COBRApy version 0.16.0 loads the model: "iHsaEC21, can be instantiated without error on the COBRA software (version 0.16.0) (Ebrahim et al, 2013)"
- [intro] FVA determines flux bounds and reversibility, distinct from single-optimum FBA: "FVA is used to compute flux bounds of the reactions, and in turn, to determine whether a given reaction is reversible or not."
