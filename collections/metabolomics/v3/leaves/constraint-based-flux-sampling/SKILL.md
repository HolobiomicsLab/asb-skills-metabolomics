---
name: constraint-based-flux-sampling
description: Use when you have cell-relative SBML (or irreversible mat) constraint-based
  models and need empirical flux distributions to test which reactions differ between
  samples, rather than relying on a single FBA optimum.
when_to_use_negative:
- You need a single growth-optimal flux vector for one condition — run FBA instead
  of sampling the full feasible region.
- No cell-relative model exists yet (you must first run getGPRsFromModel → getRASscore
  → getNormalizedRAS → rasIntegration to build per-cell-line ENGRO2 models).
- Your model is still fully reversible; the sampler expects the irreversible format
  produced by the MATLAB conversion step.
edam_operation: http://edamontology.org/operation_3663
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_2258
tools:
- name: INTEGRATE
  role: pipeline containing the randomSampling step that wraps optGpSampler for FFD
    generation
  repo: https://github.com/qLSLab/integrate
- name: COBRApy
  role: Python framework through which the optGpSampler algorithm is invoked on the
    irreversible models
- name: optGpSampler
  role: the random-sampling algorithm used to draw steady-state solutions with thinning
    10
- name: GLPK
  role: LP/MILP solver used by COBRApy during sampling
- name: MATLAB
  role: used to convert each reversible SBML model into the irreversible mat format
    the sampler requires
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1371/journal.pcbi.1009337
    title: 'INTEGRATE: Model-based multi-omics data integration to characterize multi-level
      metabolic regulation'
  source_content_hash: sha256:5f062c992f391359ee98c1aa53759dc96ebd9180bc464d4eb01232cbe9261b01
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/qLSLab/integrate
  tools:
  - COBRApy
  - GLPK
  - INTEGRATE
  - MATLAB
  - Python
  - optGpSampler
  edam_operation: http://edamontology.org/operation_3663
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_2258
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6111111111111112
n_claims: 18
---

# constraint-based-flux-sampling

## Summary

Generates Feasible Flux Distribution (FFD) datasets by sampling the feasible flux region of each cell-relative metabolic model (e.g. ENGRO2_MCF7.xml) with the optGpSampler algorithm in COBRApy, producing per-cell-line CSV files of steady-state flux solutions that downstream Mann-Whitney U tests compare.

## When to use

Use when you have cell-relative SBML (or irreversible mat) constraint-based models and need empirical flux distributions to test which reactions differ between samples, rather than relying on a single FBA optimum.

## When NOT to use

- You need a single growth-optimal flux vector for one condition — run FBA instead of sampling the full feasible region.
- No cell-relative model exists yet (you must first run getGPRsFromModel → getRASscore → getNormalizedRAS → rasIntegration to build per-cell-line ENGRO2 models).
- Your model is still fully reversible; the sampler expects the irreversible format produced by the MATLAB conversion step.

## Inputs

- cell-relative SBML metabolic model per sample (e.g. ENGRO2_MCF7.xml from rasIntegration)
- irreversible model in mat format (*_wIrrRxns) generated via MATLAB conversion
- ysi_ratio.csv extracellular flux-ratio constraints
- medium.csv nutrients availability constraints (already integrated at model-construction time)
- biomass reaction name (default 'Biomass'), uptake/demand reaction names (EX_glc__D_e, EX_gln__L_e, EX_lac__L_e, EX_gluOUT__L_e)

## Outputs

- randomSampling_<modelId>_nSol_<nSamples>_<cellLine>_<timeStamp>.csv — one CSV per cell line containing the sampled steady-state flux solutions (10 batches x 100,000 solutions)

## How to apply

Convert each cell-relative SBML model (e.g. ENGRO2_MCF102A.xml, ENGRO2_MCF7.xml, ENGRO2_SKBR3.xml) to a mat file, then use a MATLAB function to convert the reversible model into the irreversible format required by the sampler (yielding *_wIrrRxns models). Reload the irreversible models in Python and run `python pipeline/randomSampling.py nSamples nBatches` with 100,000 samples per batch and ten batches (1,000,000 solutions per cell line), using optGpSampler via COBRApy with thinning of 10 and the GLPK solver. Impose a growth-yield constraint by setting the lower bound epsilon = 1e-4 on the 'Biomass' reaction, and keep the extracellular flux-ratio constraints (ysI) from ysi_ratio.csv on glucose/glutamine uptake and lactate/glutamate excretion (reactions EX_glc__D_e, EX_gln__L_e, EX_lac__L_e, EX_gluOUT__L_e). Sampled solutions are written as 'randomSampling_' + modelId + '_nSol_' + str(nSamples) + '_' + cellLine + '_' + timeStamp + '.csv'. The rationale is that a single FBA optimum underdetermines fluxes; sampled FFDs give the distribution needed for non-parametric comparison of regulation across samples.

## Related tools

- **INTEGRATE** (pipeline containing the randomSampling step that wraps optGpSampler for FFD generation) — https://github.com/qLSLab/integrate
- **COBRApy** (Python framework through which the optGpSampler algorithm is invoked on the irreversible models)
- **optGpSampler** (the random-sampling algorithm used to draw steady-state solutions with thinning 10)
- **GLPK** (LP/MILP solver used by COBRApy during sampling)
- **MATLAB** (used to convert each reversible SBML model into the irreversible mat format the sampler requires)

## Examples

```
python pipeline/randomSampling.py 100000 10
```

## Evaluation signals

- Each of the five randomSampling output CSV files exists and contains the requested number of sampled solutions (10 batches x 100,000).
- The biomass reaction lower bound (epsilon = 1e-4) is respected in all sampled solutions.
- Extracellular flux-ratio constraints from ysi_ratio.csv (glucose/glutamine uptake, lactate/glutamate excretion) hold across sampled solutions.
- Sampled solutions come from the *_wIrrRxns irreversible models, not the original reversible SBML files.

## Limitations

- Sampling is done on irreversible models; reversible reactions must first be split via the MATLAB conversion step.
- The growth-yield constraint (epsilon = 1e-4 on the biomass reaction) restricts the sampled space and is a pipeline-specific choice.
- The pipeline was tested using the GLPK solver; other solvers are untested per the README.

## Evidence

- [methods] optGpSampler sampling of ENGRO2 models: "The Pipeline was tested using GLPK solver (https://www.gnu.org/software/glpk/)."
- [methods] irreversible model conversion is a prerequisite: "Each model needs to be converted to a mat file in order to exploit the MATLAB function to convert model from the reversible into the irreversible format"
- [readme] randomSampling step parameters and outputs: "**Step 6: randomSampling**
* Aim: sample the feasible flux region of each cell relative model
* Usage: `python pipeline/randomSampling.py nSamples nBatches`"
- [readme] growth-yield constraint epsilon: "epsilon: lower bound imposed on the biomass synthesis reaction. Default value: 1e-4."
- [readme] ysI flux-ratio constraints applied during sampling: "ysiFileName: the input extracellular flux ratio dataset. This file consists of the following columns: *Ratio* listing the extracellular flux ratios to impose within the model"
