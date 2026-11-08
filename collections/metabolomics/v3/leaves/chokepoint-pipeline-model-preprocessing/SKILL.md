---
name: chokepoint-pipeline-model-preprocessing
description: 'Use when a steady-state SBML metabolic model (e.g. iHsaEC21_SARS-CoV-2,
  MODEL2007210001) must be refined before chokepoint or bottleneck analysis: flux
  bounds need updating from FVA, dead-end metabolites must be excluded, or reaction
  reversibility must be determined from flux ranges.'
when_to_use_negative:
- The goal is FBA-only biomass or growth-rate computation (Orth et al 2010) without
  downstream chokepoint/bottleneck analysis — flux-bound refinement is unnecessary
- The input is not a constraint-based metabolic model (e.g. it is interaction data,
  expression data, or a non-steady-state model)
- A published curated model is already supplied in a refined variant, so re-running
  FVA refinement would only add computation without changing bounds
edam_operation: http://edamontology.org/operation_3660
edam_topics:
- http://edamontology.org/topic_0605
- http://edamontology.org/topic_2259
tools:
- name: findCPcli
  role: CLI tool that runs FVA to update reaction flux bounds and removes dead-end
    metabolites, saving refined model variants via -sF/-swD/-swDF options
  repo: https://github.com/findCP/findCPcli
- name: COBRApy
  role: Python toolbox (version 0.16.0) on which findCPcli runs; used to manipulate,
    simulate, and validate that the refined SBML model loads without error
  repo: https://github.com/opencobra/cobrapy
- name: GLPK
  role: Linear solver underlying the FBA/FVA flux computations
- name: findCPcore
  role: 'Low-level API for programmatic refinement: model.fva(update_flux=True) updates
    flux bounds and model.find_chokepoints() computes chokepoints'
  repo: https://github.com/findCP/findCPcore
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.26508/lsa.202000954
    title: Integrated human/SARS-CoV-2 metabolic models present novel treatment strategies
      against COVID-19
  source_content_hash: sha256:98979d070a74449a0fc5bb8e358ec206f20427a56182430467d1547740e76908
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/findCP/findCPcli
  tools:
  - COBRApy
  - GLPK
  - findCPcli
  - findCPcore
  edam_operation: http://edamontology.org/operation_3660
  edam_topics:
  - http://edamontology.org/topic_0605
  - http://edamontology.org/topic_2259
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# chokepoint-pipeline-model-preprocessing

## Summary

Preprocesses a genome-scale metabolic model (SBML) before chokepoint/bottleneck analysis by computing FVA-updated flux bounds and/or removing dead-end metabolites with findCPcli, yielding refined model variants (model_fva.xml, model without DEM) on which reaction reversibility and chokepoint computations are more accurate.

## When to use

Use when a steady-state SBML metabolic model (e.g. iHsaEC21_SARS-CoV-2, MODEL2007210001) must be refined before chokepoint or bottleneck analysis: flux bounds need updating from FVA, dead-end metabolites must be excluded, or reaction reversibility must be determined from flux ranges.

## When NOT to use

- The goal is FBA-only biomass or growth-rate computation (Orth et al 2010) without downstream chokepoint/bottleneck analysis — flux-bound refinement is unnecessary
- The input is not a constraint-based metabolic model (e.g. it is interaction data, expression data, or a non-steady-state model)
- A published curated model is already supplied in a refined variant, so re-running FVA refinement would only add computation without changing bounds

## Inputs

- Genome-scale metabolic model in SBML (.xml), JSON (.), or YAML (.yml) format, e.g. the integrated iHsaEC21+SARS-CoV-2 model from BioModels MODEL2007210001
- Optional objective reaction ID and fraction-of-optimum parameter (0.0–1.0) for FBA/FVA

## Outputs

- FVA-refined SBML model (model_fva.xml) with each reaction's [lower, upper] bounds set to its FVA-computed min/max flux range
- Model without dead-end metabolites (-swD) or FVA-refined and DEM-free model (-swDF)
- Spreadsheet (.xls/.xlsx/.ods) with chokepoint computation results

## How to apply

Load the SBML genome-scale model (e.g. downloaded from BioModels MODEL2007210001) into findCPcli, which runs on COBRApy 0.16.0 with the GLPK solver. Choose one of the four pipeline variants: (i) base model, (ii) model without DEM, (iii) model refined with FVA (flux bounds replaced by each reaction's FVA min/max), or (iv) FVA-refined and DEM-free. Use the -sF option to write an FVA-refined SBML model (model_fva.xml) or -swDF to combine FVA refinement with dead-end metabolite removal; optionally set -objective to a specific reaction and -fraction (e.g. 0.95 for 95% of optimal growth) for suboptimal FVA. The rationale is that FVA-computed bounds reveal which reactions can actually carry flux, so chokepoint producer/consumer classification and reversibility determinations reflect dynamics rather than default bounds. Since each bottleneck reaction must be balanced by at least one other reaction producing or consuming the same metabolite, reactions with dead-end metabolites are excluded before bottleneck identification.

## Related tools

- **findCPcli** (CLI tool that runs FVA to update reaction flux bounds and removes dead-end metabolites, saving refined model variants via -sF/-swD/-swDF options) — https://github.com/findCP/findCPcli
- **COBRApy** (Python toolbox (version 0.16.0) on which findCPcli runs; used to manipulate, simulate, and validate that the refined SBML model loads without error) — https://github.com/opencobra/cobrapy
- **GLPK** (Linear solver underlying the FBA/FVA flux computations)
- **findCPcore** (Low-level API for programmatic refinement: model.fva(update_flux=True) updates flux bounds and model.find_chokepoints() computes chokepoints) — https://github.com/findCP/findCPcore

## Examples

```
findCPcli -i MODEL2007210001.xml -sF model_fva.xml -fraction 0.95
```

## Evaluation signals

- model_fva.xml loads without error when instantiated with COBRApy 0.16.0
- Every reaction's [lower, upper] bounds in the saved model equal its FVA-computed min/max flux range
- Reactions with dead-end metabolites are absent from the DEM-free variant (-swD/-swDF output)
- Downstream chokepoint producer/consumer classification is consistent with the refined (not original) flux bounds
- Reversibility assignments of reactions match the sign structure of their FVA bounds (e.g. lower bound < 0 and upper bound > 0 implies reversible)

## Limitations

- FVA refinement only rewrites flux bounds; it does not change the network stoichiometry, so structural artefacts like dead-end metabolites must be handled by a separate removal step
- The fraction-of-optimum choice for FVA (e.g. 0.95 vs 1.0) changes the computed bounds and therefore downstream chokepoint results
- MEMOTE version used for the model quality scoring is not reported, limiting reproducibility of upstream quality checks

## Evidence

- [intro] FVA-refined model via -sF option: "FVA is used to compute flux bounds of the reactions, and in turn, to determine whether a given reaction is reversible or not."
- [methods] FVA-updated bounds rationale: "iii) model refined with FVA, i.e. with flux bounds updated according to FVA"
- [intro] DEM exclusion rationale: "Because each bottleneck reaction should be balanced by at least one other reaction that produces or consumes that metabolite, we have excluded reactions in the model with dead-end metabolites."
- [readme] findCPcli capability: "In addition to the computation of chokepoints, findCPcli can compute and remove dead-end metabolites, find essential reactions and update the flux bounds of the reactions according to the results of"
- [readme] -sF parameter definition: "-sF <output file>    Save output model with reactions bounds updated with Flux Variability Analysis."
- [methods] COBRApy instantiation check: "iHsaEC21, can be instantiated without error on the COBRA software (version 0.16.0) (Ebrahim et al, 2013)"
