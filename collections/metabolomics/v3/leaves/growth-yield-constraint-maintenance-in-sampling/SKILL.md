---
name: growth-yield-constraint-maintenance-in-sampling
description: Use when sampling the feasible flux region of a constraint-based model
  where biomass output must be non-zero, so sampled FFD solutions reflect proliferating
  cells rather than zero-growth artifacts.
when_to_use_negative:
- The model is a static FBA optimization (single optimal solution) rather than random
  sampling of the feasible flux region
- The cell line of interest is non-proliferating or the analysis targets zero-growth
  states, so a non-zero biomass lower bound would bias the sampled space
- Extracellular flux-ratio or medium constraints are unavailable, so the sampled space
  is unconstrained and the growth-yield bound alone may not reflect measured physiology
edam_operation: http://edamontology.org/operation_3660
edam_topics:
- http://edamontology.org/topic_3315
- http://edamontology.org/topic_0605
tools:
- name: optGpSampler
  role: Sampler used via COBRApy to draw random steady-state flux solutions subject
    to the biomass lower bound
  repo: github.com/qLSLab/integrate
- name: COBRApy
  role: Python framework through which optGpSampler is invoked for constraint-based
    random sampling
  repo: github.com/qLSLab/integrate
- name: GLPK
  role: LP solver used by the sampling pipeline
  repo: https://www.gnu.org/software/glpk/
- name: randomSampling.py
  role: INTEGRATE pipeline script that applies the biomass epsilon bound during sampling
  repo: github.com/qLSLab/integrate
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1371/journal.pcbi.1009337
    title: 'INTEGRATE: Model-based multi-omics data integration to characterize multi-level
      metabolic regulation'
  source_content_hash: sha256:0383f94dfada3e5059d6e2af012db7bdd4d8e6a64b48684e3561a4a4b07a615e
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
  - randomSampling.py
  edam_operation: http://edamontology.org/operation_3660
  edam_topics:
  - http://edamontology.org/topic_3315
  - http://edamontology.org/topic_0605
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 18
---

# growth-yield-constraint-maintenance-in-sampling

## Summary

Maintain the growth-yield constraint during random flux sampling of cell-relative metabolic models by imposing a lower bound of 1e-4 on the biomass reaction while sampling a million steady-state solutions with optGpSampler in COBRApy.

## When to use

Use when sampling the feasible flux region of a constraint-based model where biomass output must be non-zero, so sampled FFD solutions reflect proliferating cells rather than zero-growth artifacts.

## When NOT to use

- The model is a static FBA optimization (single optimal solution) rather than random sampling of the feasible flux region
- The cell line of interest is non-proliferating or the analysis targets zero-growth states, so a non-zero biomass lower bound would bias the sampled space
- Extracellular flux-ratio or medium constraints are unavailable, so the sampled space is unconstrained and the growth-yield bound alone may not reflect measured physiology

## Inputs

- Cell-relative SBML models (e.g. ENGRO2_MCF102A.xml) converted to irreversible *_wIrrRxns format
- ysi_ratio.csv extracellular flux-ratio dataset
- medium.csv nutrients availability dataset
- nSamples per batch (100,000) and nBatches (10) parameters
- Biomass reaction name (default 'Biomass') and epsilon lower bound (default 1e-4)

## Outputs

- randomSampling_' + modelId + '_nSol_' + str(nSamples) + '_' + cellLine + '_' + timeStamp + '.csv' files containing sampled FFD solutions per cell line, each satisfying the growth-yield and flux-ratio constraints

## How to apply

When running randomSampling on each cell-relative ENGRO2 model (e.g. ENGRO2_MCF102A.xml, ENGRO2_MCF7.xml), impose a lower bound epsilon = 1e-4 on the biomass reaction (default value 'Biomass') so every sampled solution retains a minimum growth yield. Run `python pipeline/randomSampling.py nSamples nBatches` with 100,000 samples per batch over ten batches, using optGpSampler via COBRApy with thinning of 10 and the GLPK solver on the irreversible *_wIrrRxns models. Keep the extracellular flux-ratio constraints (ysI) from ysi_ratio.csv on glucose/glutamine uptake and lactate/glutamate excretion alongside the biomass bound, so the sampled flux space is simultaneously constrained by growth yield and observed exchange ratios. The rationale is that without a non-zero biomass bound the sampler can return solutions with no growth, which would not represent the proliferating phenotype being compared across cell lines. Save each cell line's samples as 'randomSampling_' + modelId + '_nSol_' + str(nSamples) + '_' + cellLine + '_' + timeStamp + '.csv'.

## Related tools

- **optGpSampler** (Sampler used via COBRApy to draw random steady-state flux solutions subject to the biomass lower bound) — github.com/qLSLab/integrate
- **COBRApy** (Python framework through which optGpSampler is invoked for constraint-based random sampling) — github.com/qLSLab/integrate
- **GLPK** (LP solver used by the sampling pipeline) — https://www.gnu.org/software/glpk/
- **randomSampling.py** (INTEGRATE pipeline script that applies the biomass epsilon bound during sampling) — github.com/qLSLab/integrate

## Examples

```
python pipeline/randomSampling.py 100000 10
```

## Evaluation signals

- Verify each randomSampling output CSV exists for every input cell line and contains the expected number of solutions (10 batches × 100,000).
- Check that every sampled solution has a biomass flux at or above the imposed epsilon (1e-4), confirming the growth-yield constraint was retained.
- Confirm that extracellular flux ratios (glucose/glutamine uptake, lactate/glutamate excretion) in the sampled solutions respect the values in ysi_ratio.csv for each cell line replica.
- Validate that sampled solutions are steady-state feasible in the irreversible *_wIrrRxns model (all fluxes satisfy S·v = 0).
- Check that the output filename matches the pattern 'randomSampling_' + modelId + '_nSol_' + str(nSamples) + '_' + cellLine + '_' + timeStamp + '.csv'.

## Limitations

- The growth-yield constraint is imposed as a fixed lower bound (epsilon = 1e-4) on the biomass reaction, which may exclude physiologically relevant low-growth or quiescent states if the analysis includes them.
- Sampling requires the model to first be converted to the irreversible format (Step 5, MATLAB conversion to *_wIrrRxns mat files) — reversible SBML models cannot be sampled directly.
- Sampling is performed in batches (100,000 per batch × 10 batches) with thinning of 10, so results depend on batch structure and the assumed thinning interval.
- The constraint relies on the specific biomass reaction name ('Biomass') and exchange reaction IDs (EX_lac__L_e, EX_glc__D_e, EX_gln__L_e, EX_gluOUT__L_e) hard-coded in the pipeline defaults; models with different IDs need explicit overrides.

## Evidence

- [other] Retain the growth-yield constraint by imposing a lower bound epsilon = 1e-4 on the biomass reaction: [evidence span withheld — anchor status: not_found]
- [other] Sampling method and solver: [evidence span withheld — anchor status: not_found]
- [readme] Output file naming convention: "File named 'randomSampling_' + modelId + '_nSol_' + str(nSamples) + '_' + cellLine + '_' + timeStamp + '.csv' containing the nSamples sampled solutions from each input sample model"
- [readme] Default epsilon parameter: "epsilon: lower bound imposed on the biomass synthesis reaction. Default value: 1e-4."
- [readme] Sampling purpose: "sample the feasible flux region of each cell relative model"
- [readme] Model format requirement: "Each model needs to be converted to a mat file in order to exploit the MATLAB function to convert model from the reversible into the irreversible format"
