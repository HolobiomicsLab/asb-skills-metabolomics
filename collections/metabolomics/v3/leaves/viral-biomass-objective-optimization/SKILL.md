---
name: viral-biomass-objective-optimization
description: Use when you have an SBML integrated host-virus genome-scale metabolic
  model and need to compute maximal host biomass or viral biomass growth rates via
  FBA, or verify that forcing maximal viral flux abolishes host biomass.
when_to_use_negative:
- The model contains no distinct viral biomass maintenance reaction, so the host/virus
  objective separation cannot be made
- The goal is drug-target prioritisation (chokepoints, bottleneck reactions, dead-end
  metabolite removal) rather than growth-rate phenotypes — use findCPcli's chokepoint/bottleneck
  workflow instead
- The model has not passed quality control (e.g. MEMOTE scoring) or cannot be instantiated
  without error in COBRApy, so FBA results would not be trustworthy
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_3307
- http://edamontology.org/topic_3391
tools:
- name: COBRApy
  role: Loads the integrated SBML models, sets the host or viral biomass objective,
    and runs FBA simulations
  repo: https://github.com/opencobra/cobrapy
- name: GLPK
  role: Linear programming solver used by COBRApy to solve the FBA optimization problems
- name: Python
  role: Scripting environment for assembling the growth values and zero-biomass check
    into fba_growth_results.csv
provenance:
  source_task_ids:
  - task_004
  - task_005
  source_papers:
  - doi: 10.26508/lsa.202000954
    title: Integrated human/SARS-CoV-2 metabolic models present novel treatment strategies
      against COVID-19
  source_content_hash: sha256:35b2a6d8a388c5f40d89e684778c17bac44ec90dbeb59db2b0e9338389bc2904
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/opencobra/cobrapy
  tools:
  - COBRApy
  - GLPK
  - Python
  - findCPcli
  edam_operation: http://edamontology.org/operation_3927
  edam_topics:
  - http://edamontology.org/topic_3307
  - http://edamontology.org/topic_3391
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# viral-biomass-objective-optimization

## Summary

Runs Flux Balance Analysis on integrated human/SARS-CoV-2 genome-scale metabolic models (iHsaEC21 and iAB-AMØ-1410 variants) with COBRApy and GLPK, alternating between host biomass maintenance and viral biomass maintenance objectives to quantify host-optimum and virus-optimum growth rates and test host-viral viability trade-offs.

## When to use

Use when you have an SBML integrated host-virus genome-scale metabolic model and need to compute maximal host biomass or viral biomass growth rates via FBA, or verify that forcing maximal viral flux abolishes host biomass.

## When NOT to use

- The model contains no distinct viral biomass maintenance reaction, so the host/virus objective separation cannot be made
- The goal is drug-target prioritisation (chokepoints, bottleneck reactions, dead-end metabolite removal) rather than growth-rate phenotypes — use findCPcli's chokepoint/bottleneck workflow instead
- The model has not passed quality control (e.g. MEMOTE scoring) or cannot be instantiated without error in COBRApy, so FBA results would not be trustworthy

## Inputs

- Integrated human/SARS-CoV-2 SBML model (e.g. iHsaEC21+SARS-CoV-2, BioModels MODEL2007210001)
- Integrated macrophage/SARS-CoV-2 SBML model from Renz et al (2020), based on iAB-AMØ-1410
- Reaction identifiers for the host biomass maintenance function and the viral biomass maintenance function
- Solver specification (GLPK) and COBRApy version (0.16.0)

## Outputs

- Host biomass maintenance rates under host-optimum FBA (0.012 h−1 epithelial, 0.0269 macrophage)
- Maximum viral growth rates under virus-optimum FBA (0.0181 h−1 epithelial, 0.0144 h−1 macrophage)
- Zero host biomass result from FBA with viral biomass flux fixed at its maximum
- Results table fba_growth_results.csv

## How to apply

Load the integrated SBML model (iHsaEC21+SARS-CoV-2, or the Renz et al macrophage model) with COBRApy 0.16.0 using the GLPK linear programming solver. First remove or deactivate the viral biomass reaction and run FBA maximizing the host biomass maintenance objective to obtain host growth (0.012 h−1 for iHsaEC21, 0.0269 for iAB-AMØ-1410). Then set the objective to viral biomass maintenance and maximize to obtain maximum viral growth (0.0181 h−1 epithelial, 0.0144 h−1 macrophage). The key decision point is the non-viability check: fix the viral biomass reaction flux at its computed maximum and re-run FBA maximizing host biomass — the rationale from the article is that SARS-CoV-2 entry drives the host cell toward death, so host biomass must drop to zero in both models. Assemble the four growth values and the zero-biomass check into a results table.

## Related tools

- **COBRApy** (Loads the integrated SBML models, sets the host or viral biomass objective, and runs FBA simulations) — https://github.com/opencobra/cobrapy
- **GLPK** (Linear programming solver used by COBRApy to solve the FBA optimization problems)
- **Python** (Scripting environment for assembling the growth values and zero-biomass check into fba_growth_results.csv)

## Examples

```
from cobra.io import read_sbml_model; m = read_sbml_model('iHsaEC21_SARS-CoV-2.xml'); m.objective = 'viral_biomass_maintenance'; print(m.optimize().objective_value)
```

## Evaluation signals

- Host-optimum FBA reproduces 0.012 h−1 for iHsaEC21 and 0.0269 for iAB-AMØ-1410 in the absence of virus
- Virus-optimum FBA reproduces maximum viral growth of 0.0181 h−1 (epithelial) and 0.0144 h−1 (macrophage)
- FBA with viral biomass flux fixed at its maximum yields host biomass exactly zero in both host compartments
- The iHsaEC21 model instantiates without error on COBRApy version 0.16.0 before optimization
- Results table fba_growth_results.csv contains all four growth values plus the zero-biomass check

## Limitations

- FBA gives optimal growth-rate phenotypes only; it does not identify which specific reactions are essential — that requires single-reaction deletion analysis (e.g. via findCPcli)
- Predicted growth values depend on the model's objective definitions and the GLPK solver; alternative solvers or objective formulations may yield different optima
- The host-non-viability conclusion is an in silico result of the fixed-maximum-viral-flux constraint, not an experimentally validated outcome for a specific drug treatment

## Evidence

- [intro] FBA computes host and viral growth rates: "The Flux Balance Analysis (FBA) method (Orth et al, 2010) was used to compute both the biomass maintenance of the cell in the absence of virus and the maximum growth rate"
- [methods] Simulation toolchain: "We manipulated and simulated the model, including the addition of the gene expression data, with COBRApy (Ebrahim et al, 2013) and GLPK"
- [methods] COBRApy version grounding: "iHsaEC21, can be instantiated without error on the COBRA software (version 0.16.0) (Ebrahim et al, 2013)"
- [abstract] Viral entry drives host cell death (rationale for non-viability check): "the entry of SARS-CoV-2 into the human cell causes biochemical and structural changes, leading to a change of cell functions or cell death."
