---
name: constraint-based-flux-perturbation-analysis
description: Use when an integrated host–virus genome-scale metabolic model (SBML)
  is available and you need to quantify host-optimum vs virus-optimum growth rates,
  or test whether maximizing viral biomass renders the host cell non-viable.
when_to_use_negative:
- The goal is chokepoint, bottleneck, or drug-target prediction on the metabolic network
  — use findCPcli-style chokepoint/dead-end-metabolite analysis instead of paired
  FBA optimizations
- The model is a standalone host GEM with no integrated viral biomass reaction, so
  no virus-optimum / host–virus trade-off can be computed
- Flux bounds need updating or reaction reversibility determination — that requires
  Flux Variability Analysis, not host/virus objective perturbation
edam_operation: http://edamontology.org/operation_3660
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3407
tools:
- name: COBRApy
  role: Manipulates and simulates the integrated host/virus SBML models and performs
    the FBA optimizations (version 0.16.0)
  repo: https://github.com/opencobra/cobrapy
- name: GLPK
  role: Linear programming solver used by COBRApy to solve the FBA problems
- name: Python
  role: Language in which the FBA workflow is implemented via the COBRApy toolbox
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.26508/lsa.202000954
    title: Integrated human/SARS-CoV-2 metabolic models present novel treatment strategies
      against COVID-19
  source_content_hash: sha256:fdaee4f349fb4ae1911c8e051a2451e9ec40c13b254d58a28f99deb29de73429
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
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3407
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 6
---

# constraint-based-flux-perturbation-analysis

## Summary

Runs paired Flux Balance Analysis optimizations on integrated host/virus genome-scale metabolic models (e.g. iHsaEC21+SARS-CoV-2 and the Renz et al 2020 macrophage/SARS-CoV-2 model) to quantify host biomass maintenance in the absence of virus, maximum viral growth rate, and the collapse of host biomass when viral flux is fixed at its optimum.

## When to use

Use when an integrated host–virus genome-scale metabolic model (SBML) is available and you need to quantify host-optimum vs virus-optimum growth rates, or test whether maximizing viral biomass renders the host cell non-viable.

## When NOT to use

- The goal is chokepoint, bottleneck, or drug-target prediction on the metabolic network — use findCPcli-style chokepoint/dead-end-metabolite analysis instead of paired FBA optimizations
- The model is a standalone host GEM with no integrated viral biomass reaction, so no virus-optimum / host–virus trade-off can be computed
- Flux bounds need updating or reaction reversibility determination — that requires Flux Variability Analysis, not host/virus objective perturbation

## Inputs

- Integrated host/virus genome-scale metabolic model in SBML format (e.g. iHsaEC21+SARS-CoV-2, BioModels MODEL2007210001)
- Integrated macrophage/SARS-CoV-2 model from Renz et al (2020) (host model iAB-AMØ-1410)
- Objective reaction identifiers for host biomass and viral biomass reactions

## Outputs

- Host-optimum biomass maintenance rate in the absence of virus (h−1)
- Maximum viral growth rate (h−1)
- Host biomass under virus-optimum constraint (expected zero; non-viability result)
- Results table (e.g. fba_growth_results.csv) with the four growth values and zero-biomass check

## How to apply

Load the integrated SBML models (iHsaEC21+SARS-CoV-2 and the macrophage iAB-AMØ-1410/SARS-CoV-2 model from Renz et al 2020) with COBRApy 0.16.0 using the GLPK linear programming solver. First run FBA maximizing the host biomass maintenance objective with the viral biomass reaction removed to obtain host-optimum growth (0.012 h−1 for the epithelial cell, 0.0269 for the macrophage). Second, run FBA maximizing the viral biomass objective to get maximum viral growth (0.0181 h−1 epithelial, 0.0144 h−1 macrophage). Third, fix the viral biomass reaction flux at its computed maximum and re-run FBA maximizing the host biomass objective; the host biomass should drop to zero in both compartments, demonstrating the virus-optimum renders the host non-viable. Assemble the four growth values plus the zero-biomass check into a results table (e.g. fba_growth_results.csv).

## Related tools

- **COBRApy** (Manipulates and simulates the integrated host/virus SBML models and performs the FBA optimizations (version 0.16.0)) — https://github.com/opencobra/cobrapy
- **GLPK** (Linear programming solver used by COBRApy to solve the FBA problems)
- **Python** (Language in which the FBA workflow is implemented via the COBRApy toolbox)

## Examples

```
import cobra; model = cobra.io.read_sbml_model('iHsaEC21_SARS2.xml'); sol = model.optimize(); print(sol.objective_value)
```

## Evaluation signals

- Host-optimum FBA with viral biomass removed reproduces 0.012 h−1 for iHsaEC21 and 0.0269 for iAB-AMØ-1410
- Virus-optimum FBA reproduces maximum viral growth of 0.0181 h−1 (epithelial) and 0.0144 h−1 (macrophage)
- Fixing viral biomass flux at its maximum and re-optimizing host biomass yields exactly zero in both compartments
- Models instantiate without error on COBRA version 0.16.0 and solve with the GLPK solver
- Results table contains all four growth values plus the zero-biomass check before being saved

## Limitations

- Computed growth rates depend on the model version and media/constraint setup; values must match the reported benchmarks (0.012 h−1 epithelial host, 0.0269 macrophage host, 0.0181/0.0144 h−1 viral) to be considered valid reproductions
- The non-viability result is an in silico FBA finding; the article notes no experimental validation of in silico predictions was presented
- MEMOTE version used for model quality scoring of iHsaEC21 (51% quality score) was not reported

## Evidence

- [intro] The Flux Balance Analysis (FBA) method (Orth et al, 2010) was used to compute both the biomass maintenance of the cell in the absence of virus and the maximum growth rate of the virus in the cell: "The Flux Balance Analysis (FBA) method (Orth et al, 2010) was used to compute both the biomass maintenance of the cell in the absence of virus and the maximum growth rate of the virus in the cell"
- [methods] We manipulated and simulated the model, including the addition of the gene expression data, with COBRApy (Ebrahim et al, 2013) and GLPK: "We manipulated and simulated the model, including the addition of the gene expression data, with COBRApy (Ebrahim et al, 2013) and GLPK"
- [methods] iHsaEC21, can be instantiated without error on the COBRA software (version 0.16.0) (Ebrahim et al, 2013): "iHsaEC21, can be instantiated without error on the COBRA software (version 0.16.0) (Ebrahim et al, 2013)"
- [abstract] the entry of SARS-CoV-2 into the human cell causes biochemical and structural changes, leading to a change of cell functions or cell death: "the entry of SARS-CoV-2 into the human cell causes biochemical and structural changes, leading to a change of cell functions or cell death."
- [readme] findCPcli can also compute and remove dead-end metabolites, find essential reactions, and update the flux bounds of the reactions according to: "In addition to the computation of chokepoints, findCPcli can compute and remove dead-end metabolites, find essential reactions and update the flux bounds of the reactions according to the results of"
