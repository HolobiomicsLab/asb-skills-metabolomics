---
name: metabolic-model-reversible-to-irreversible-conversion
description: Use when a cell-relative SBML model produced by rasIntegration must be
  split so that reversible reactions are represented as separate forward and reverse
  reactions before sampling or RAS t-tests.
when_to_use_negative:
- When the model is already irreversible and only needs sampling or RAS scoring directly.
- When the goal is to inspect or edit GPR rules rather than to prepare for flux sampling
  — getGPRsFromModel operates on the original reversible SBML model.
- When converting a model that has not yet been constrained by rasIntegration, since
  the conversion is part of the cell-relative pipeline and expects the integrated
  SBML.
edam_operation: http://edamontology.org/operation_0334
edam_topics:
- http://edamontology.org/topic_0153
tools:
- name: INTEGRATE
  role: The pipeline whose Step 5 requires converting each cell-relative model from
    reversible to irreversible format via a MATLAB function before randomSampling
    and RAS t-test.
  repo: https://github.com/qLSLab/integrate
- name: MATLAB
  role: Hosts the helper function that converts the model from the reversible into
    the irreversible format; each model must first be exported as a mat file to exploit
    it.
- name: GLPK
  role: The solver with which the pipeline (including the converted irreversible models)
    was tested; used to confirm the converted model remains solvable.
  repo: https://www.gnu.org/software/glpk/
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1371/journal.pcbi.1009337
    title: 'INTEGRATE: Model-based multi-omics data integration to characterize multi-level
      metabolic regulation'
  source_content_hash: sha256:dc15ccb59be0155bee82e4ccbb1210346d3b32beb016299e7e5834e2831d4e7d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/qLSLab/integrate
  tools:
  - GLPK
  - INTEGRATE
  - MATLAB
  - Python
  edam_operation: http://edamontology.org/operation_0334
  edam_topics:
  - http://edamontology.org/topic_0153
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# metabolic-model-reversible-to-irreversible-conversion

## Summary

Convert a constraint-based metabolic model from its reversible into an irreversible format via a MATLAB helper, exporting cell-relative SBML models as .mat files first. This irreversible split is required because downstream INTEGRATE steps (RAS t-test, randomSampling) operate on reactions separated by direction.

## When to use

Use when a cell-relative SBML model produced by rasIntegration must be split so that reversible reactions are represented as separate forward and reverse reactions before sampling or RAS t-tests.

## When NOT to use

- When the model is already irreversible and only needs sampling or RAS scoring directly.
- When the goal is to inspect or edit GPR rules rather than to prepare for flux sampling — getGPRsFromModel operates on the original reversible SBML model.
- When converting a model that has not yet been constrained by rasIntegration, since the conversion is part of the cell-relative pipeline and expects the integrated SBML.

## Inputs

- Cell-relative SBML models produced by rasIntegration (one .xml per cell line, e.g. ENGRO2_MCF102A.xml)
- MATLAB environment with the model-conversion helper function

## Outputs

- Irreversible-format model in mat-file form for each input cell line (e.g. ENGRO2_MCF102A_wIrrRxns), with reversible reactions split into direction-specific reactions
- Split RAS score file derived from the irreversible model (output of rasTtest), named rasScoreFile + '_splt.csv'

## How to apply

After rasIntegration emits one SBML model per cell line (e.g. ENGRO2_MCF102A.xml), load each model in MATLAB and apply the reversible-to-irreversible conversion function to obtain a mat file whose reactions are directionally split. The conversion is required because Step 8 (rasTtest) expects an irreversible model (e.g. 'ENGRO2_MCF102A_wIrrRxns') to split reversible reactions in the RAS score file, and randomSampling samples the feasible flux region of the irreversible models. The pipeline was tested with the GLPK solver, so verify the converted model still solves with GLPK before proceeding. Typical inputs are the five breast cell-line models (MCF102A, MDAMB231, SKBR3, MCF7, MDAMB361) generated with ENGRO2, and the output is one irreversible mat file per cell line feeding Steps 6 and 8.

## Related tools

- **INTEGRATE** (The pipeline whose Step 5 requires converting each cell-relative model from reversible to irreversible format via a MATLAB function before randomSampling and RAS t-test.) — https://github.com/qLSLab/integrate
- **MATLAB** (Hosts the helper function that converts the model from the reversible into the irreversible format; each model must first be exported as a mat file to exploit it.)
- **GLPK** (The solver with which the pipeline (including the converted irreversible models) was tested; used to confirm the converted model remains solvable.) — https://www.gnu.org/software/glpk/

## Examples

```
MATLAB: load ENGRO2_MCF102A.xml; convertToIrreversible(ENGRO2_MCF102A); save ENGRO2_MCF102A_wIrrRxns.mat ENGRO2_MCF102A_wIrrRxns
```

## Evaluation signals

- The irreversible mat file for each cell line must preserve the stoichiometric structure but list reversible reactions twice (forward and reverse direction), matching the naming convention used in Step 8 (e.g. 'ENGRO2_MCF102A_wIrrRxns').
- randomSampling on the converted model should produce the same number of sampled solutions as specified (nSamples per batch), confirming the model still solves after conversion.
- rasTtest output file 'rasScoreFile_splt.csv' must contain the RAS scores with reversible reactions split, consistent with the irreversible model's reaction set.
- The model retains the imposed constraints from rasIntegration (YSI ratios, medium bounds, RAS constraints) after conversion.
- Solving the converted model with GLPK should succeed before randomSampling is invoked.

## Limitations

- The conversion is a prerequisite step that must be executed in MATLAB; the pipeline does not offer a pure-Python alternative for this step, so users without MATLAB cannot proceed from SBML to irreversible models within the pipeline as documented.
- The README indicates models must be exported to mat file first, adding an intermediate serialization step that can introduce format-compatibility pitfalls if SBML and MATLAB versions mismatch.
- The conversion applies to the input cell-relative models generated by rasIntegration; applying it to a generic unconstrained model is out of scope for the documented workflow.

## Evidence

- [readme] Each model needs to be converted to a mat file in order to exploit the MATLAB function to convert model from the reversible into the irreversible format: "Each model needs to be converted to a mat file in order to exploit the MATLAB function to convert model from the reversible into the irreversible format"
- [readme] The input rasScoreFile converted so that reversible reactions in the input modelIrreversible file are splitted: "The input rasScoreFile converted so that reversible reactions in the input modelIrreversible file are splitted. The resulting file is named rasScoreFile + '_splt.csv'"
- [readme] modelIrreversible: one of the irreversible models generated during Step 5. Default value: 'ENGRO2_MCF102A_wIrrRxns': "modelIrreversible: one of the irreversible models generated during Step 5. Default value: 'ENGRO2_MCF102A_wIrrRxns'"
- [readme] The Pipeline was tested using GLPK solver: "The Pipeline was tested using GLPK solver (https://www.gnu.org/software/glpk/)."
