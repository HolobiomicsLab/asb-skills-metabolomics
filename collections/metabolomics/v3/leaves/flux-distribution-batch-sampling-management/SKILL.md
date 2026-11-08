---
name: flux-distribution-batch-sampling-management
description: Use when you have cell-relative SBML metabolic models (e.g. ENGRO2_MCF102A.xml,
  ENGRO2_MCF7.xml) already split into irreversible format, and you need a large ensemble
  of steady-state flux solutions (order 1e6) per sample to compare flux distributions
  between cell lines.
when_to_use_negative:
- Do not use when the goal is deterministic flux prediction (e.g. FBA) rather than
  characterizing the full feasible flux region via sampling.
- 'Do not use on the generic ENGRO2 model before rasIntegration: the sampling is defined
  per cell-relative model with RAS and ysi constraints already integrated.'
- Do not use when only a small number of flux solutions is needed — the batch machinery
  and thinning parameter are designed for very large (order 1e6) ensembles.
edam_operation: http://edamontology.org/operation_3660
edam_topics:
- http://edamontology.org/topic_3307
- http://edamontology.org/topic_0202
tools:
- name: COBRApy
  role: Python library through which the optGpSampler algorithm is invoked to sample
    the feasible flux region of each irreversible cell-relative model.
- name: optGpSampler
  role: Sampling algorithm executed via COBRApy with thinning of 10 and the GLPK solver
    to draw steady-state solutions.
- name: GLPK
  role: Linear programming solver used by COBRApy/optGpSampler during flux sampling;
    the pipeline was tested with GLPK.
  repo: https://www.gnu.org/software/glpk/
- name: MATLAB
  role: Used (via mat-file conversion of the SBML model) to run the reversible-to-irreversible
    model conversion whose *_wIrrRxns outputs are the sampler's inputs.
- name: INTEGRATE
  role: The enclosing pipeline whose randomSampling step (Step 6) implements this
    skill.
  repo: https://github.com/qLSLab/integrate
- name: Python
  role: Runtime environment in which pipeline/randomSampling.py is executed after
    the irreversible models are reloaded.
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1371/journal.pcbi.1009337
    title: 'INTEGRATE: Model-based multi-omics data integration to characterize multi-level
      metabolic regulation'
  source_content_hash: sha256:8d46ea025eaf4638b61b883e5af01c70f84546c8d59d2e1c650eb2d48ddaf796
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - COBRApy
  - GLPK
  - INTEGRATE
  - MATLAB
  - Python
  - optGpSampler
  edam_operation: http://edamontology.org/operation_3660
  edam_topics:
  - http://edamontology.org/topic_3307
  - http://edamontology.org/topic_0202
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5294117647058824
n_claims: 17
---

# flux-distribution-batch-sampling-management

## Summary

Generates feasible flux distribution (FFD) datasets by sampling the steady-state flux region of cell-relative constraint-based models with optGpSampler via COBRApy, executed in configurable batches with a growth-yield lower bound on the biomass reaction, producing per-cell-line sampled-solution CSV files used downstream for Mann-Whitney U testing.

## When to use

Use when you have cell-relative SBML metabolic models (e.g. ENGRO2_MCF102A.xml, ENGRO2_MCF7.xml) already split into irreversible format, and you need a large ensemble of steady-state flux solutions (order 1e6) per sample to compare flux distributions between cell lines.

## When NOT to use

- Do not use when the goal is deterministic flux prediction (e.g. FBA) rather than characterizing the full feasible flux region via sampling.
- Do not use on the generic ENGRO2 model before rasIntegration: the sampling is defined per cell-relative model with RAS and ysi constraints already integrated.
- Do not use when only a small number of flux solutions is needed — the batch machinery and thinning parameter are designed for very large (order 1e6) ensembles.

## Inputs

- Irreversible cell-relative metabolic model in mat format (e.g. ENGRO2_MCF102A_wIrrRxns) derived from the SBML output of rasIntegration
- Extracellular flux-ratio constraints file 'ysi_ratio.csv' with 'Ratio' column plus per-cell-line replica value columns
- Biomass reaction name (default 'Biomass') with lower bound epsilon = 1e-4
- Sample/cell-line list (default ['MCF102A', 'MDAMB231', 'SKBR3', 'MCF7', 'MDAMB361'])
- nSamples (solutions per batch) and nBatches parameters

## Outputs

- Per-cell-line CSV of sampled steady-state flux solutions named 'randomSampling_' + modelId + '_nSol_' + str(nSamples) + '_' + cellLine + '_' + timeStamp + '.csv' (nSamples x nBatches solutions per cell line)

## How to apply

Convert each cell-relative SBML model produced by rasIntegration (e.g. ENGRO2_MCF102A.xml, ENGRO2_MCF7.xml, ENGRO2_SKBR3.xml, ENGRO2_MDAMB231.xml, ENGRO2_MDAMB361.xml) to a mat file and use the MATLAB reversible-to-irreversible conversion to obtain the *_wIrrRxns models, because the sampler requires irreversible networks. Reload the irreversible models in Python and run optGpSampler through COBRApy with the GLPK solver, keeping the extracellular flux-ratio constraints (ysI) from ysi_ratio.csv on glucose/glutamine uptake and lactate/glutamate excretion. Impose a growth-yield constraint by setting the lower bound epsilon = 1e-4 on the biomass reaction ('Biomass' by default). Execute sampling in batches, e.g. ten batches of nSamples = 100,000 solutions each, using thinning of 10 to decorrelate successive samples; total sampled solutions per cell line equals nSamples x nBatches. Name each output file 'randomSampling_' + modelId + '_nSol_' + str(nSamples) + '_' + cellLine + '_' + timeStamp + '.csv' so that downstream steps (mannWhitneyUTest) can locate them via the timestamp and nSamples.

## Related tools

- **COBRApy** (Python library through which the optGpSampler algorithm is invoked to sample the feasible flux region of each irreversible cell-relative model.)
- **optGpSampler** (Sampling algorithm executed via COBRApy with thinning of 10 and the GLPK solver to draw steady-state solutions.)
- **GLPK** (Linear programming solver used by COBRApy/optGpSampler during flux sampling; the pipeline was tested with GLPK.) — https://www.gnu.org/software/glpk/
- **MATLAB** (Used (via mat-file conversion of the SBML model) to run the reversible-to-irreversible model conversion whose *_wIrrRxns outputs are the sampler's inputs.)
- **INTEGRATE** (The enclosing pipeline whose randomSampling step (Step 6) implements this skill.) — https://github.com/qLSLab/integrate
- **Python** (Runtime environment in which pipeline/randomSampling.py is executed after the irreversible models are reloaded.)

## Examples

```
python pipeline/randomSampling.py 100000 10
```

## Evaluation signals

- Each of the five expected output files (randomSampling_ENGRO2_nSol_100000_<cellLine>_<timeStamp>.csv) exists for MCF102A, MDAMB231, SKBR3, MCF7 and MDAMB361.
- Each output CSV contains nSamples x nBatches (e.g. 10 batches x 100,000) sampled solutions, i.e. the row count matches the requested total.
- Every sampled solution satisfies the imposed steady-state constraints, including the biomass lower bound epsilon = 1e-4 and the ysi extracellular flux ratios on glucose/glutamine uptake and lactate/glutamate excretion.
- The output file name embeds the correct modelId, nSol, cellLine and timeStamp tokens so mannWhitneyUTest can resolve the files via its timeStampInput and nSamples arguments.
- Sampling was run on the irreversible *_wIrrRxns models rather than the original reversible SBML files, confirming the Step 5 model-splitting prerequisite was completed.

## Limitations

- Sampling depends on prior conversion to irreversible format via MATLAB; reversible models cannot be sampled directly with this pipeline.
- The biomass lower bound is a fixed default epsilon = 1e-4; this constrains the feasible region to non-trivial growth and may not suit models with different biomass scaling.
- The extracellular flux-ratio constraints (ysI) are specific to the glucose/glutamine/lactate/glutamate reactions configured in the pipeline (EX_glc__D_e, EX_gln__L_e, EX_lac__L_e, EX_gluOUT__L_e) and are read from ysi_ratio.csv per cell line and replica.
- Sampling batches are independent files; the total ensemble (nSamples x nBatches) is only aggregated at the downstream Mann-Whitney U test step.

## Evidence

- [readme] Aim: sample the feasible flux region of each cell relative model: "Aim: sample the feasible flux region of each cell relative model"
- [readme] randomSampling usage and nSamples/nBatches parameters: "Usage: `python pipeline/randomSampling.py nSamples nBatches`"
- [readme] MATLAB conversion prerequisite for sampler inputs: "Each model needs to be converted to a mat file in order to exploit the MATLAB function to convert model from the reversible into the irreversible format"
- [readme] output file naming convention: "File named 'randomSampling_' + modelId + '_nSol_' + str(nSamples) + '_' + cellLine + '_' + timeStamp + '.csv' containing the nSamples sampled solutions from each input sample model"
- [readme] ysI constraint rationale on extracellular fluxes: "ysiFileName: the input extracellular flux ratio dataset. This file consists of the following columns: *Ratio* listing the extracellular flux ratios to impose within the model"
- [methods] GLPK solver tested in the pipeline: "The Pipeline was tested using GLPK solver (https://www.gnu.org/software/glpk/)."
