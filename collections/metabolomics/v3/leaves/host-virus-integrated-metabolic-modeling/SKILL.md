---
name: host-virus-integrated-metabolic-modeling
description: Use when quantifying host cell and viral growth rates from an integrated
  host-pathogen genome-scale metabolic model in SBML format, or when reproducing the
  reported FBA growth values for SARS-CoV-2-infected epithelial and macrophage models.
when_to_use_negative:
- The goal is identifying drug targets or bottleneck/chokepoint reactions rather than
  computing growth rates — use findCPcli's chokepoint and dead-end-metabolite analysis
  instead
- Only a host-only (uninfected) genome-scale model is available, so no viral biomass
  reaction or host-virus interaction exists to model
- Kinetic or dynamic simulation of viral infection is required — FBA only provides
  steady-state flux optima
edam_operation: http://edamontology.org/operation_3660
edam_topics:
- http://edamontology.org/topic_2259
- http://edamontology.org/topic_0769
tools:
- name: COBRApy
  role: Loads the SBML integrated host-virus models, sets biomass objectives, and
    performs FBA simulations
  repo: https://github.com/opencobra/cobrapy
- name: GLPK
  role: Linear programming solver underlying the COBRApy FBA optimizations
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.26508/lsa.202000954
    title: Integrated human/SARS-CoV-2 metabolic models present novel treatment strategies
      against COVID-19
  source_content_hash: sha256:cb0588f36ac5cf6839ae8bd971da03dece7d637f94847c15359228968d07af37
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/opencobra/cobrapy
  tools:
  - COBRApy
  - GLPK
  - Python
  edam_operation: http://edamontology.org/operation_3660
  edam_topics:
  - http://edamontology.org/topic_2259
  - http://edamontology.org/topic_0769
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# host-virus-integrated-metabolic-modeling

## Summary

Runs Flux Balance Analysis on host-virus integrated genome-scale metabolic models (iHsaEC21+SARS-CoV-2, iAB-AMØ-1410 macrophage model) to compute host biomass maintenance rates, maximum viral growth rates, and the non-viability result when viral flux is fixed at its maximum.

## When to use

Use when quantifying host cell and viral growth rates from an integrated host-pathogen genome-scale metabolic model in SBML format, or when reproducing the reported FBA growth values for SARS-CoV-2-infected epithelial and macrophage models.

## When NOT to use

- The goal is identifying drug targets or bottleneck/chokepoint reactions rather than computing growth rates — use findCPcli's chokepoint and dead-end-metabolite analysis instead
- Only a host-only (uninfected) genome-scale model is available, so no viral biomass reaction or host-virus interaction exists to model
- Kinetic or dynamic simulation of viral infection is required — FBA only provides steady-state flux optima

## Inputs

- Integrated host-virus genome-scale metabolic model in SBML format (e.g. iHsaEC21+SARS-CoV-2, BioModels MODEL2007210001)
- Macrophage/SARS-CoV-2 integrated model from Renz et al (2020)
- Host biomass maintenance reaction identifier and viral biomass reaction identifier for each model
- Linear programming solver configuration (GLPK)

## Outputs

- Host biomass maintenance rate without virus (h−1): 0.012 epithelial, 0.0269 macrophage
- Maximum viral growth rate (h−1): 0.0181 epithelial, 0.0144 macrophage
- Fixed-viral-flux FBA result demonstrating zero host biomass in both models
- Results table (fba_growth_results.csv) with the four growth values and the zero-biomass check

## How to apply

Load the SBML models (iHsaEC21+SARS-CoV-2 from BioModels MODEL2007210001 and the Renz et al 2020 macrophage/SARS-CoV-2 model) with COBRApy 0.16.0 using the GLPK linear programming solver. First run FBA maximizing the host biomass maintenance objective with the viral biomass reaction removed, yielding 0.012 h−1 for the epithelial cell and 0.0269 for the macrophage in the absence of virus. Second, run FBA maximizing the viral biomass objective to obtain maximum viral growth of 0.0181 h−1 (epithelial) and 0.0144 h−1 (macrophage). Third, fix the viral biomass reaction flux at its maximum value and re-run FBA maximizing host biomass to verify host biomass drops to zero in both compartments — the paper's non-viability result. Assemble the four growth values plus the zero-biomass check into a results table (e.g. fba_growth_results.csv).

## Related tools

- **COBRApy** (Loads the SBML integrated host-virus models, sets biomass objectives, and performs FBA simulations) — https://github.com/opencobra/cobrapy
- **GLPK** (Linear programming solver underlying the COBRApy FBA optimizations)

## Examples

```
import cobra; model = cobra.io.read_sbml_model('iHsaEC21_SARS-CoV-2.xml'); sol = model.optimize(); print(sol.objective_value)
```

## Evaluation signals

- Host-optimum FBA (viral biomass reaction removed) reproduces 0.012 h−1 for iHsaEC21 and 0.0269 for iAB-AMØ-1410
- Viral-objective FBA reproduces maximum viral growth of 0.0181 h−1 (epithelial) and 0.0144 h−1 (macrophage)
- Fixing the viral biomass reaction at its maximum flux yields host biomass exactly zero in both models
- Models instantiate without error on COBRApy version 0.16.0 with GLPK
- Results table fba_growth_results.csv contains all four growth values plus the zero-biomass check

## Limitations

- FBA assumes steady-state flux and does not capture dynamic infection processes; results depend on the specific biomass reaction definitions of the models
- A nonzero maximal viral growth rate is incompatible with host biomass in these models — the virus cannot grow at its optimum without killing the host cell, so single-objective optima are not simultaneously achievable
- Model quality depends on curation status (iHsaEC21 MEMOTE score of 51%) and reactions with dead-end metabolites may affect flux solutions

## Evidence

- [intro] The Flux Balance Analysis (FBA) method (Orth et al, 2010) was used to compute both the biomass maintenance of the cell in the absence of virus and the maximum growth rate of the virus in the cell: "The Flux Balance Analysis (FBA) method (Orth et al, 2010) was used to compute both the biomass maintenance of the cell in the absence of virus and the maximum growth rate of the virus in the cell"
- [methods] We manipulated and simulated the model, including the addition of the gene expression data, with COBRApy (Ebrahim et al, 2013) and GLPK: "We manipulated and simulated the model, including the addition of the gene expression data, with COBRApy (Ebrahim et al, 2013) and GLPK (https://www.gnu.org/software/glpk/)"
- [methods] iHsaEC21, can be instantiated without error on the COBRA software (version 0.16.0): "iHsaEC21, can be instantiated without error on the COBRA software (version 0.16.0) (Ebrahim et al, 2013)"
- [discussion] iHsaEC21 now has a MEMOTE quality score of 51% (Supplemental Data 1): "iHsaEC21 now has a MEMOTE quality score of 51% (Supplemental Data 1)"
- [abstract] the entry of SARS-CoV-2 into the human cell causes biochemical and structural changes, leading to a change of cell functions or cell death: "the entry of SARS-CoV-2 into the human cell causes biochemical and structural changes, leading to a change of cell functions or cell death."
