---
name: flux-balance-analysis-of-virus-host-models
description: Use when an SBML genome-scale model integrates a virus and host (e.g.
  iHsaEC21+SARS-CoV-2) and you need host-optimum vs virus-optimum FBA/FVA flux comparisons
  to quantify which host pathways the virus hijacks.
when_to_use_negative:
- Input is a single-organism model with no integrated viral biomass/production reaction
  — there is no virus-optimum condition to compare against.
- The goal is chokepoint or dead-end-metabolite analysis for drug-target prioritisation
  — use findCPcli's chokepoint/DEM workflow instead of host-vs-virus FVA comparison.
- Kinetic or dynamical analysis of the model is required — FBA/FVA only yields steady-state
  flux bounds, not time-course behaviour.
edam_operation: http://edamontology.org/operation_3660
edam_topics:
- http://edamontology.org/topic_0602
tools:
- name: COBRApy
  role: Loads the SBML model and runs FBA and FVA to compute host biomass maintenance,
    viral maximum growth rate, and per-reaction flux bounds
  repo: https://github.com/opencobra/cobrapy
- name: GLPK
  role: LP solver used by COBRApy for the FBA/FVA optimisations
- name: findCPcli
  role: 'Companion tool for the same SBML genome-scale models: computes chokepoints,
    removes dead-end metabolites, finds essential reactions, and updates flux bounds
    according to FVA results'
  repo: https://github.com/findCP/findCPcli
- name: Python
  role: Scripting environment for the COBRApy workflow and flux-increase computations
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.26508/lsa.202000954
    title: Integrated human/SARS-CoV-2 metabolic models present novel treatment strategies
      against COVID-19
  source_content_hash: sha256:3592373f5d8c9beefb12522c02269bcf4d9709d8655ece159a99ee32efb76945
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
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# flux-balance-analysis-of-virus-host-models

## Summary

Runs FBA and FVA with COBRApy/GLPK on the SBML-encoded integrated human airway epithelial cell/SARS-CoV-2 model (iHsaEC21+virus, 3,752 reactions, 4,660 genes) to compare host fluxes at host-optimum versus virus-optimum growth, revealing host viability collapse and pathway-level flux increases (e.g. 277% in lipid metabolism).

## When to use

Use when an SBML genome-scale model integrates a virus and host (e.g. iHsaEC21+SARS-CoV-2) and you need host-optimum vs virus-optimum FBA/FVA flux comparisons to quantify which host pathways the virus hijacks.

## When NOT to use

- Input is a single-organism model with no integrated viral biomass/production reaction — there is no virus-optimum condition to compare against.
- The goal is chokepoint or dead-end-metabolite analysis for drug-target prioritisation — use findCPcli's chokepoint/DEM workflow instead of host-vs-virus FVA comparison.
- Kinetic or dynamical analysis of the model is required — FBA/FVA only yields steady-state flux bounds, not time-course behaviour.

## Inputs

- SBML file of the integrated host/virus genome-scale metabolic model (e.g. iHsaEC21+SARS-CoV-2; 3,752 reactions, 3,914 metabolites, 4,660 genes, 48 pathways; BioModels MODEL2007210001)
- Host biomass objective function and viral biomass maintenance function defined in the model
- Pathway annotations to group reactions (lipid metabolism, sphingolipid metabolism, fatty acid biosynthesis)

## Outputs

- Host-optimal growth rate μmax from FBA on the host-only network
- Virus-optimal growth rate and host biomass maintenance flux (zero under virus optimum)
- Per-reaction lower/upper flux bounds from FVA under host-optimum and virus-optimum constraints
- flux_comparison.xls: per-reaction host vs virus fluxes, percentage increases, and pathway-level summary averages

## How to apply

Instantiate the SBML model with COBRApy 0.16.0 and the GLPK solver. First run FBA maximizing the host biomass reaction (without the viral production reaction) to obtain μmax, the host-optimal growth rate. Then run FBA with the viral biomass maintenance function as objective to obtain the virus-optimal solution — this reproduces the finding that host biomass maintenance flux is zero under virus-optimum conditions. Run FVA on the host network both with and without the viral production reaction, constraining v_growth = μmax for the host case, to get lower/upper flux bounds per reaction. For each reaction compute the percentage increase in flux between host-optimum and virus-optimum solutions, group perturbed reactions by pathway (lipid, sphingolipid, fatty-acid biosynthesis), and save per-reaction fluxes plus pathway-level averages as flux_comparison.xls.

## Related tools

- **COBRApy** (Loads the SBML model and runs FBA and FVA to compute host biomass maintenance, viral maximum growth rate, and per-reaction flux bounds) — https://github.com/opencobra/cobrapy
- **GLPK** (LP solver used by COBRApy for the FBA/FVA optimisations)
- **findCPcli** (Companion tool for the same SBML genome-scale models: computes chokepoints, removes dead-end metabolites, finds essential reactions, and updates flux bounds according to FVA results) — https://github.com/findCP/findCPcli
- **Python** (Scripting environment for the COBRApy workflow and flux-increase computations)

## Examples

```
import cobra; m = cobra.io.read_sbml_model('iHsaEC21_SARS2.xml'); m.solver = 'glpk'; fva = cobra.flux_analysis.flux_variability_analysis(m, fraction_of_optimum=1.0)
```

## Evaluation signals

- Pathway-level percentage increases in flux_comparison.xls reproduce the reported values: 277% (32 lipid metabolism reactions), 228% (14 sphingolipid reactions), and ~190% (4 fatty-acid biosynthesis reactions).
- Under virus-optimum conditions, FVA shows zero host biomass maintenance flux, confirming the host cell is not viable when the virus runs at its maximum growth rate.
- The flux comparison table covers all reactions returned by both FVA runs (host network with and without the viral production reaction).
- Model instantiation succeeds without error on COBRApy version 0.16.0 with the GLPK solver.
- The FBA host-only objective yields a positive μmax, establishing the host-optimum baseline against which virus-optimum fluxes are compared.

## Limitations

- Under virus-optimum conditions the host biomass maintenance flux is zero, so the host-optimum vs virus-optimum flux comparison reflects a non-viable host state.
- findCPcli assumes SBML input of genome-scale models; other formats require conversion.
- FVA-derived flux bounds were used to refine the model (update flux bounds) and exclude reactions with dead-end metabolites, so results depend on which of the four model variants (original, without DEM, FVA-refined, FVA-refined without DEM) is analysed.

## Evidence

- [intro] FBA/FVA purpose in the integrated host/virus models: "The Flux Balance Analysis (FBA) method (Orth et al, 2010) was used to compute both the biomass maintenance of the cell in the absence of virus and the maximum growth rate of the virus in the cell"
- [intro] FVA computes flux bounds per reaction: "FVA is used to compute flux bounds of the reactions, and in turn, to determine whether a given reaction is reversible or not."
- [methods] COBRApy and GLPK as the simulation stack: "We manipulated and simulated the model, including the addition of the gene expression data, with COBRApy (Ebrahim et al, 2013) and GLPK"
- [methods] Model instantiation on COBRApy 0.16.0: "iHsaEC21, can be instantiated without error on the COBRA software (version 0.16.0) (Ebrahim et al, 2013)"
- [readme] findCPcli SBML input and spreadsheet output: "findCPcli takes as input an SBML files of genome-scale models, and provides as output a spreadsheet file with the results of the chokepoint"
