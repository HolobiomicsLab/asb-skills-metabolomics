---
name: sbml-model-flux-bound-refinement
description: Use when an SBML genome-scale metabolic model has default or inconsistent
  reaction bounds and downstream analyses (chokepoints, reversibility, bottlenecks)
  require bounds updated to steady-state FVA-computed ranges.
when_to_use_negative:
- The task requires raw FVA flux ranges as a report/spreadsheet rather than an updated
  model file — use the default output spreadsheet (-o) instead of model refinement
- The downstream analysis does not depend on reaction reversibility or flux bounds
  (e.g. purely topological or gene-level analyses)
- You intend to alter the model's stoichiometry or network structure — FVA refinement
  only updates reaction bounds, it does not add or remove reactions (dead-end metabolite
  removal is a separate option)
edam_operation: http://edamontology.org/operation_3660
edam_topics:
- http://edamontology.org/topic_2259
- http://edamontology.org/topic_0203
tools:
- name: findCPcli
  role: Command-line tool that runs FVA on the SBML model and saves the refined model
    with flux bounds updated via the -sF (or -swDF) option
  repo: https://github.com/findCP/findCPcli
- name: COBRApy 0.16.0
  role: Python toolbox underlying findCPcli for loading and simulating the model;
    used for validation that the refined model instantiates without error
  repo: https://github.com/opencobra/cobrapy
- name: GLPK
  role: Linear solver used with COBRApy to solve the FBA/FVA LP problems
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.26508/lsa.202000954
    title: Integrated human/SARS-CoV-2 metabolic models present novel treatment strategies
      against COVID-19
  source_content_hash: sha256:388720c706c95ecfeb633c6c8b8c24cbc27a264134bf5d8c8b7943dc91f4a925
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/findCP/findCPcli
  tools:
  - COBRApy
  - COBRApy 0.16.0
  - GLPK
  - findCPcli
  edam_operation: http://edamontology.org/operation_3660
  edam_topics:
  - http://edamontology.org/topic_2259
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# sbml-model-flux-bound-refinement

## Summary

Refine a genome-scale metabolic model in SBML format by replacing every reaction's flux bounds with the min/max flux ranges computed via Flux Variability Analysis, producing an FVA-refined model (e.g. model_fva.xml) that supports reliable reversibility determination and flux-dependent chokepoint analyses with findCPcli.

## When to use

Use when an SBML genome-scale metabolic model has default or inconsistent reaction bounds and downstream analyses (chokepoints, reversibility, bottlenecks) require bounds updated to steady-state FVA-computed ranges.

## When NOT to use

- The task requires raw FVA flux ranges as a report/spreadsheet rather than an updated model file — use the default output spreadsheet (-o) instead of model refinement
- The downstream analysis does not depend on reaction reversibility or flux bounds (e.g. purely topological or gene-level analyses)
- You intend to alter the model's stoichiometry or network structure — FVA refinement only updates reaction bounds, it does not add or remove reactions (dead-end metabolite removal is a separate option)

## Inputs

- Genome-scale metabolic model in SBML format (e.g. iHsaEC21_SARS-CoV-2 integrated model, BioModels MODEL2007210001; .xml/./.yml accepted)
- Optional objective reaction ID for FBA/FVA
- Optional fraction of optimum growth for FVA (between 0.0 and 1.0)

## Outputs

- FVA-refined SBML model (e.g. model_fva.xml) with each reaction's flux bounds replaced by its FVA-computed min/max flux range
- Optionally, FVA-refined model without dead-end metabolites (via -swDF)

## How to apply

Load the SBML genome-scale model (e.g. the integrated iHsaEC21+SARS-CoV-2 model, BioModels MODEL2007210001) with findCPcli, which runs on COBRApy with the GLPK linear solver, and run Flux Variability Analysis (Orth et al 2010) to compute the lower and upper steady-state flux of every reaction. Optionally set an objective reaction and a fraction of optimal growth (e.g. -fraction 0.95) used by FVA. Use the -sF option to save a new SBML model whose reaction bounds are updated with the FVA results — the 'model refined with FVA' variant of the findCPcli pipeline; -swDF additionally removes dead-end metabolites, which is required before bottleneck analysis since each bottleneck must be balanced by another reaction. Validate by reloading model_fva.xml in COBRApy (version 0.16.0 per the article) and checking each reaction's [lower, upper] bounds equal its FVA min/max flux range.

## Related tools

- **findCPcli** (Command-line tool that runs FVA on the SBML model and saves the refined model with flux bounds updated via the -sF (or -swDF) option) — https://github.com/findCP/findCPcli
- **COBRApy 0.16.0** (Python toolbox underlying findCPcli for loading and simulating the model; used for validation that the refined model instantiates without error) — https://github.com/opencobra/cobrapy
- **GLPK** (Linear solver used with COBRApy to solve the FBA/FVA LP problems)

## Examples

```
findCPcli -i iHsaEC21_SARS-CoV-2.xml -sF model_fva.xml -fraction 0.95
```

## Evaluation signals

- model_fva.xml loads without error in COBRApy version 0.16.0
- Every reaction's [lower, upper] bounds in the refined model equal its FVA-computed min/max flux range
- A reaction is classified reversible only when both FVA-derived bounds are non-zero in magnitude (positive lower bound and/or negative upper bound)
- The refined model still produces the expected FBA objective value when the fraction-of-optimum setting (e.g. 0.95) is respected
- Downstream chokepoint/bottleneck computations on the refined model run without dead-end metabolite errors (when -swDF was used)

## Limitations

- FVA refinement depends on the chosen objective and fraction of optimum growth; bounds change if these settings differ
- Reactions with dead-end metabolites yield degenerate FVA ranges and are excluded from downstream bottleneck analyses (DEM removal is a separate step)
- The article's scope is genome-scale metabolic models in SBML format processed on top of COBRApy/GLPK; behaviour on other formats or solvers is not described

## Evidence

- [methods] FVA refines bounds: "iii) model refined with FVA, i.e. with flux bounds updated according to FVA"
- [readme] findCPcli updates bounds: "In addition to the computation of chokepoints, findCPcli can compute and remove dead-end metabolites, find essential reactions and update the flux bounds of the reactions according to the results of"
- [readme] -sF parameter: "-sF <output file>    Save output model with reactions bounds updated with
                       Flux Variability Analysis. Allowed file formats: .xml
                       . .yml"
- [intro] FVA determines reversibility: "FVA is used to compute flux bounds of the reactions, and in turn, to determine whether a given reaction is reversible or not."
- [intro] DEM exclusion rationale: "Because each bottleneck reaction should be balanced by at least one other reaction that produces or consumes that metabolite, we have excluded reactions in the model with dead-end metabolites."
- [methods] COBRApy/GLPK stack: "We manipulated and simulated the model, including the addition of the gene expression data, with COBRApy (Ebrahim et al, 2013) and GLPK"
