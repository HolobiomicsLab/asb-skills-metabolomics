---
name: breast-cell-line-metabolic-model-simulation
description: Use when cell-relative metabolic models (e.g. ENGRO2_MCF102A, ENGRO2_MCF7)
  have been generated from RAS integration and you need to characterize their feasible
  flux distributions.
when_to_use_negative:
- Before the cell-relative models exist — if rasIntegration has not yet produced ENGRO2_<cellLine>.xml
  outputs, there is no model to sample
- When the goal is differential reaction expression or RAS scoring rather than flux-space
  characterization — Steps 1-3 of the pipeline (getGPRsFromModel, getRASscore, getNormalizedRAS)
  handle transcriptomics-based regulation instead
- When extracellular flux ratio or medium constraints are not available/applicable
  — the sampler relies on ysi_ratio.csv and medium.csv to constrain the feasible region
edam_operation: http://edamontology.org/operation_3961
edam_topics:
- http://edamontology.org/topic_3168
- http://edamontology.org/topic_0605
tools:
- name: INTEGRATE
  role: Pipeline containing the randomSampling step; coordinates input/output naming
    and constraints
  repo: https://github.com/qLSLab/integrate
- name: Python
  role: Runtime for the randomSampling script
- name: MATLAB
  role: Converts reversible SBML/mat models into irreversible format needed by the
    sampler
- name: GLPK
  role: LP solver used by COBRApy during sampling
  repo: https://www.gnu.org/software/glpk/
- name: COBRApy
  role: Python library through which optGpSampler samples the feasible flux region
- name: optGpSampler
  role: Sampling algorithm executed via COBRApy with thinning=10
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1371/journal.pcbi.1009337
    title: 'INTEGRATE: Model-based multi-omics data integration to characterize multi-level
      metabolic regulation'
  source_content_hash: sha256:d5bb91b13d895edc8d8913df71789246c55c76dbe9f4e17f432410c27dbfd0ca
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
  edam_operation: http://edamontology.org/operation_3961
  edam_topics:
  - http://edamontology.org/topic_3168
  - http://edamontology.org/topic_0605
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# breast-cell-line-metabolic-model-simulation

## Summary

Sample the feasible flux region of cell-relative breast cell line metabolic models (ENGRO2) using random flux sampling to probe flux space.

## When to use

Use when cell-relative metabolic models (e.g. ENGRO2_MCF102A, ENGRO2_MCF7) have been generated from RAS integration and you need to characterize their feasible flux distributions.

## When NOT to use

- Before the cell-relative models exist — if rasIntegration has not yet produced ENGRO2_<cellLine>.xml outputs, there is no model to sample
- When the goal is differential reaction expression or RAS scoring rather than flux-space characterization — Steps 1-3 of the pipeline (getGPRsFromModel, getRASscore, getNormalizedRAS) handle transcriptomics-based regulation instead
- When extracellular flux ratio or medium constraints are not available/applicable — the sampler relies on ysi_ratio.csv and medium.csv to constrain the feasible region

## Inputs

- Cell-relative SBML metabolic model per breast cell line (e.g. ENGRO2_MCF102A.xml, ENGRO2_MCF7.xml, ENGRO2_SKBR3.xml, ENGRO2_MDAMB231.xml, ENGRO2_MDAMB361.xml)
- MAT file format of each model for reversible-to-irreversible conversion
- ysi_ratio.csv (extracellular flux ratio constraints per cell line replica)
- medium.csv (nutrient availability lower/upper bounds per cell line)
- Biomass reaction name (default 'Biomass') and epsilon lower bound (default 1e-4)

## Outputs

- randomSampling_<modelId>_nSol_<nSamples>_<cellLine>_<timeStamp>.csv — one CSV per cell line containing the 1,000,000 sampled solutions (100,000 per batch × 10 batches)
- Irreversible models (*_wIrrRxns) required by the sampler
- Feasible flux distribution (FFD) dataset for each of the five cell lines

## How to apply

Following the INTEGRATE pipeline's Step 5-6, convert each cell-relative SBML model (e.g. ENGRO2_MCF102A.xml, ENGRO2_MCF7.xml) to a mat file, then convert from reversible to irreversible format via a MATLAB function, producing *_wIrrRxns models. Run `python pipeline/randomSampling.py nSamples nBatches` with nSamples=100000 and nBatches=10, using optGpSampler via COBRApy with GLPK solver, thinning=10, and epsilon=1e-4 lower bound on the Biomass reaction. The sampling imposes extracellular flux ratio constraints (glucose/glutamine uptake, lactate/glutamate excretion) from ysi_ratio.csv and nutrient availability from medium.csv. Output is 'randomSampling_' + modelId + '_nSol_' + str(nSamples) + '_' + cellLine + '_' + timeStamp + '.csv' per cell line.

## Related tools

- **INTEGRATE** (Pipeline containing the randomSampling step; coordinates input/output naming and constraints) — https://github.com/qLSLab/integrate
- **Python** (Runtime for the randomSampling script)
- **MATLAB** (Converts reversible SBML/mat models into irreversible format needed by the sampler)
- **GLPK** (LP solver used by COBRApy during sampling) — https://www.gnu.org/software/glpk/
- **COBRApy** (Python library through which optGpSampler samples the feasible flux region)
- **optGpSampler** (Sampling algorithm executed via COBRApy with thinning=10)

## Examples

```
python pipeline/randomSampling.py 100000 10
```

## Evaluation signals

- Each of the five randomSampling output CSVs exists and contains the requested number of sampled solutions (10 batches × 100,000 = 1,000,000)
- The sampled solutions respect the imposed extracellular flux ratios from ysi_ratio.csv and the biomass lower bound epsilon = 1e-4
- The irreversible models (*_wIrrRxns) were successfully generated from the mat files before sampling
- Output file naming matches 'randomSampling_' + modelId + '_nSol_' + str(nSamples) + '_' + cellLine + '_' + timeStamp + '.csv'
- The sampled flux distributions reflect the cell-specific RAS and medium constraints applied during rasIntegration

## Limitations

- Sampling is restricted to models already constrained by transcriptomics-derived RAS and extracellular metabolomics (ysi/medium) constraints from rasIntegration; it does not re-fit constraints independently
- Models must be converted from reversible to irreversible format before sampling — the sampler does not accept the original reversible SBML directly
- The sampling requires a specific batch size (100,000) and batch count (10) to reproduce the published FFD dataset; changing these changes the output granularity
- Thinning and solver choice (GLPK) are fixed defaults; alternative solvers or thinning values are not documented in the README

## Evidence

- [readme] Aim: sample the feasible flux region of each cell relative model: "Step 6: randomSampling * Aim: sample the feasible flux region of each cell relative model * Usage: `python pipeline/randomSampling.py nSamples nBatches`"
- [readme] Output file naming for sampled solutions: "File named 'randomSampling_' + modelId + '_nSol_' + str(nSamples) + '_' + cellLine + '_' + timeStamp + '.csv' containing the nSamples sampled solutions from each input sample model"
- [readme] Input requirements for sampling (ysi and medium constraints): "ysiFileName: the input extracellular flux ratio dataset. This file consists of the following columns: *Ratio* listing the extracellular flux ratios to impose within the model; n columns for each cell"
- [methods] Model conversion requirement before sampling: "Each model needs to be converted to a mat file in order to exploit the MATLAB function to convert model from the reversible into the irreversible format"
- [methods] Pipeline solver used for sampling: "The Pipeline was tested using GLPK solver (https://www.gnu.org/software/glpk/)."
- [readme] RAS integration produces cell relative models: "Step 4: rasIntegration * Aim: integrate RAS scores within the input generic models to generate cell relative models"
- [readme] Epsilon lower bound on biomass synthesis: "epsilon: lower bound imposed on the biomass synthesis reaction. Default value: 1e-4."
