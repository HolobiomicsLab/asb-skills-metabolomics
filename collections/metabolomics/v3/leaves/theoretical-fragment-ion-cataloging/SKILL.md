---
name: theoretical-fragment-ion-cataloging
description: Use when building a searchable 4D sterol lipid database that requires
  predicted fragment ions for N-Me derivatized (C=C bond-targeted) unsaturated sterols,
  e.g. to support LC-IM-MS/MS based identification at the isomer level.
when_to_use_negative:
- Input molecules lack C=C bonds (e.g. saturated sterols or non-unsaturated lipids),
  since the N-Me fragmentation logic depends on double bond recognition
- The goal is CCS prediction rather than fragment-ion generation — that task uses
  the separate QCC-assisted scikit-learn (LASSO + SVR) workflow
- The goal is experimental data processing and sterol identification rather than in-silico
  fragment prediction — use the LC-IM-MS/MS data processing part of the pipeline instead
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0092
tools:
- name: RDkit
  role: Provides the built-in functions on which the MS/MS calculation script is written;
    used to parse sterol structures, recognize double bond positions, and generate
    N-Me fragmentation patterns
  repo: https://github.com/Chen-micslab/QCCAssisted4DSterol
- name: Jupyter Notebook
  role: Environment in which all MS/MS calculation functions are implemented and run
  repo: https://github.com/Chen-micslab/QCCAssisted4DSterol
- name: Python
  role: Language of the QCCAssisted4DSterol script collection, including the MS/MS
    calculation module
  repo: https://github.com/Chen-micslab/QCCAssisted4DSterol
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1002/anie.202507483
    title: Quantum Chemistry Calculation‐Assisted Large‐Scale Collision Cross Section
      Prediction Empowers Derivatization‐Enhanced Multidimensional Metabolomics
  source_content_hash: sha256:403b86a0d193ecfbe936d9a3bc52daf6b63a695a256d02533c48eba1cc3befdf
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/Chen-micslab/QCCAssisted4DSterol
  tools:
  - Jupyter Notebook
  - Python
  - RDkit
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# Theoretical Fragment Ion Cataloging

## Summary

Generate theoretical MS/MS fragment-ion information (m/z and relative intensity) for N-Me derivatized unsaturated sterol lipids by recognizing C=C double bond positions in each sterol backbone and applying N-Me fragmentation patterns encoded via RDkit built-in functions.

## When to use

Use when building a searchable 4D sterol lipid database that requires predicted fragment ions for N-Me derivatized (C=C bond-targeted) unsaturated sterols, e.g. to support LC-IM-MS/MS based identification at the isomer level.

## When NOT to use

- Input molecules lack C=C bonds (e.g. saturated sterols or non-unsaturated lipids), since the N-Me fragmentation logic depends on double bond recognition
- The goal is CCS prediction rather than fragment-ion generation — that task uses the separate QCC-assisted scikit-learn (LASSO + SVR) workflow
- The goal is experimental data processing and sterol identification rather than in-silico fragment prediction — use the LC-IM-MS/MS data processing part of the pipeline instead

## Inputs

- N-Me derivatized unsaturated sterol lipid structures (parsed as molecular structures for RDkit)
- Double bond position information in sterol lipid backbones

## Outputs

- Theoretical MS/MS spectra for each derivatized sterol
- Structured table of fragment-ion records with m/z and relative intensity fields per derivatized sterol

## How to apply

Load derivatized unsaturated sterol lipid structures into a Jupyter Notebook environment and parse them with RDkit. Run the RDkit-based MS/MS calculation script, which first recognizes the double bond positions in each sterol lipid backbone and then generates MS/MS spectra based on the N-Me fragmentation patterns implemented with RDkit built-in functions. Record the computed fragment-ion information (m/z and relative intensity) for each derivatized sterol into a structured table so the records can be merged with retention time and CCS predictions into the 4D sterol database. The script is theoretically applicable to all molecules containing a C=C bond, though in this project it was only tested on sterol lipids.

## Related tools

- **RDkit** (Provides the built-in functions on which the MS/MS calculation script is written; used to parse sterol structures, recognize double bond positions, and generate N-Me fragmentation patterns) — https://github.com/Chen-micslab/QCCAssisted4DSterol
- **Jupyter Notebook** (Environment in which all MS/MS calculation functions are implemented and run) — https://github.com/Chen-micslab/QCCAssisted4DSterol
- **Python** (Language of the QCCAssisted4DSterol script collection, including the MS/MS calculation module) — https://github.com/Chen-micslab/QCCAssisted4DSterol

## Evaluation signals

- The MS/MS calculation script executes end-to-end on the full sterol lipid set without errors
- Every output record contains both fragment m/z and relative-intensity fields
- Generated fragment m/z values are consistent with the N-Me fragmentation patterns encoded in the script
- Recognized double bond positions match the known unsaturation sites of the input sterol backbones
- Resulting fragment records correctly complement the retention time and CCS predictions in the 4D sterol lipid database (n = 4891 derivatized sterol lipids)

## Limitations

- Only tested on sterol lipids, although the script is theoretically applicable to all molecules containing a C=C bond
- Fragment generation depends on correct double bond position recognition; isomer-level distinctions rely on the derivatized fragmentation patterns
- No changelog is provided in the source repository

## Evidence
