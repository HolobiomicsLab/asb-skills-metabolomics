---
name: msms-fragment-annotation
description: Use when you need theoretical MS/MS fragment ions for derivatized unsaturated
  sterols (or other C=C-containing molecules) to support matching of LC-IM-MS/MS features
  against a reference lipid database.
when_to_use_negative:
- Molecules lacking C=C double bonds, since the script recognizes double bond positions
  and only sterol lipids have been tested
- Spectra matching against an already-built fragment database — use the downstream
  4D data-processing/identification workflow instead
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0154
tools:
- name: RDkit
  role: 'Basis of the MS/MS calculation script: built-in functions recognize double
    bond positions and generate fragments'
- name: QCCAssisted4DSterol
  role: Collection of Python/Jupyter scripts implementing the MS/MS calculation part
  repo: https://github.com/Chen-micslab/QCCAssisted4DSterol
- name: Python
  role: Language in which QCCAssisted4DSterol scripts are written
  repo: https://github.com/Chen-micslab/QCCAssisted4DSterol
- name: Jupyter Notebook
  role: Environment in which all MS/MS calculation functions are implemented
  repo: https://github.com/Chen-micslab/QCCAssisted4DSterol
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1002/anie.202507483
    title: Quantum Chemistry Calculation‐Assisted Large‐Scale Collision Cross Section
      Prediction Empowers Derivatization‐Enhanced Multidimensional Metabolomics
  source_content_hash: sha256:e4e976c5d40d3d2516e755dde6c51426414c5f8695344bf8531732cb403f250a
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/Chen-micslab/QCCAssisted4DSterol
  tools:
  - Jupyter Notebook
  - Python
  - QCCAssisted4DSterol
  - RDkit
  edam_operation: http://edamontology.org/operation_3646
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# msms-fragment-annotation

## Summary

Generates theoretical MS/MS fragment patterns for N-Me derivatized unsaturated sterol lipids by recognizing C=C double bond positions and applying N-Me fragmentation rules, providing the fragment-ion dimension of the 4D sterol lipid database used for isomer-level identification.

## When to use

Use when you need theoretical MS/MS fragment ions for derivatized unsaturated sterols (or other C=C-containing molecules) to support matching of LC-IM-MS/MS features against a reference lipid database.

## When NOT to use

- Molecules lacking C=C double bonds, since the script recognizes double bond positions and only sterol lipids have been tested
- Spectra matching against an already-built fragment database — use the downstream 4D data-processing/identification workflow instead

## Inputs

- Sterol lipid molecular structures with specified C=C double bond positions
- N-Me derivatization scheme definitions for unsaturated sterols

## Outputs

- Theoretical MS/MS fragment ion lists (N-Me fragmentation patterns) per sterol lipid
- Fragment-ion entries for the 4D sterol lipid database (combined with RT and CCS predictions)

## How to apply

Run the MS/MS calculation scripts from QCCAssisted4DSterol (implemented in Jupyter notebooks on top of RDkit built-in functions). Input sterol lipid structures with defined double bond positions; the script recognizes double bond positions and generates MS/MS spectra based on N-Me fragmentation patterns specific to the C=C bond-targeted N-Me derivatization. The resulting fragment ion predictions feed the 4D database (with m/z, retention time, CCS, and N-Me fragment ions, n = 4891 derivatized sterols) against which per-sample features are matched in the ./Search/tissue data-processing workflow. Although theoretically applicable to all molecules containing C=C bonds, only sterol lipids have been tested.

## Related tools

- **RDkit** (Basis of the MS/MS calculation script: built-in functions recognize double bond positions and generate fragments)
- **QCCAssisted4DSterol** (Collection of Python/Jupyter scripts implementing the MS/MS calculation part) — https://github.com/Chen-micslab/QCCAssisted4DSterol
- **Python** (Language in which QCCAssisted4DSterol scripts are written) — https://github.com/Chen-micslab/QCCAssisted4DSterol
- **Jupyter Notebook** (Environment in which all MS/MS calculation functions are implemented) — https://github.com/Chen-micslab/QCCAssisted4DSterol

## Examples

```
jupyter notebook MSMS_calculation.ipynb  # run in QCCAssisted4DSterol repo; RDkit-based script generates N-Me MS/MS for sterol structures
```

## Evaluation signals

- Generated fragments follow known N-Me fragmentation patterns for the derivatized sterol class
- Fragment predictions correctly reflect the input molecule's C=C double bond positions (isomer-level discrimination)
- Fragment ions, once combined with m/z, RT, and CCS in the 4D database, yield isomer-level sterol identifications with fragment-ion evidence fields in per-sample result tables
- The workflow produces one identification result table per sample folder under ./Search/tissue, each containing matched sterol identifications with fragment-ion evidence

## Limitations

- Only tested on sterol lipids, although theoretically applicable to all molecules containing C=C bonds
- Fragment predictions depend on the N-Me derivatization scheme; spectra of underivatized sterols are out of scope

## Evidence
