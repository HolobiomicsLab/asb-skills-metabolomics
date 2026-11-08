---
name: double-bond-position-recognition
description: Use when you need theoretical MS/MS fragment ions for N-Me derivatized
  (C═C bond-targeted) unsaturated sterol lipids whose double bond positions must be
  recognized from molecular structures before spectra generation.
when_to_use_negative:
- Molecules lacking C=C bonds — the script's double bond recognition and N-Me fragmentation
  patterns target unsaturated sterol lipids
- Non-sterol compound classes — the README states the script is 'only test[ed on]
  sterol lipids' despite being theoretically general
- When experimentally acquired MS/MS spectra are already available for the derivatized
  sterols rather than theoretical fragment predictions
edam_operation: http://edamontology.org/operation_0324
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_2258
tools:
- name: RDkit
  role: Provides the built-in functions on which the MS/MS calculation script is written,
    for parsing sterol structures and recognizing double bond positions
- name: Jupyter Notebook
  role: Environment in which all MS/MS calculation functions are implemented and executed
  repo: https://github.com/Chen-micslab/QCCAssisted4DSterol
- name: Python
  role: Scripting language for the QCCAssisted4DSterol MS/MS calculation scripts
  repo: https://github.com/Chen-micslab/QCCAssisted4DSterol
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1002/anie.202507483
    title: Quantum Chemistry Calculation‐Assisted Large‐Scale Collision Cross Section
      Prediction Empowers Derivatization‐Enhanced Multidimensional Metabolomics
  source_content_hash: sha256:0fe1ab267984f77a83e95adab532db4d9ead96b14ec4f51eeedd2f75180d5a8a
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/Chen-micslab/QCCAssisted4DSterol
  tools:
  - Jupyter Notebook
  - Python
  - RDkit
  edam_operation: http://edamontology.org/operation_0324
  edam_topics:
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_2258
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# double-bond-position-recognition

## Summary

An RDkit-based MS/MS calculation procedure that recognizes C═C double bond positions in N-Me derivatized unsaturated sterol lipids and generates theoretical MS/MS fragment ions (m/z and relative intensity) according to N-Me fragmentation patterns, feeding the fragment-ion layer of the 4D sterol lipid database.

## When to use

Use when you need theoretical MS/MS fragment ions for N-Me derivatized (C═C bond-targeted) unsaturated sterol lipids whose double bond positions must be recognized from molecular structures before spectra generation.

## When NOT to use

- Molecules lacking C=C bonds — the script's double bond recognition and N-Me fragmentation patterns target unsaturated sterol lipids
- Non-sterol compound classes — the README states the script is 'only test[ed on] sterol lipids' despite being theoretically general
- When experimentally acquired MS/MS spectra are already available for the derivatized sterols rather than theoretical fragment predictions

## Inputs

- N-Me derivatized unsaturated sterol lipid structures (C=C bond-targeted derivatization) parsed as RDkit molecule objects
- Sterol lipid structure definitions encoding double bond positions

## Outputs

- Theoretical MS/MS spectra for each N-Me derived sterol based on N-Me fragmentation patterns
- Structured table of fragment-ion information (m/z and relative intensity) per derivatized sterol
- Fragment ion predictions contributing to the 4D sterol lipid database (retention time, CCS, m/z, fragments)

## How to apply

Load the derivatized unsaturated sterol lipid structures into a Jupyter Notebook environment and parse them with RDkit. Apply the RDkit built-in-function-based script to recognize double bond positions in each sterol lipid backbone. Generate theoretical MS/MS fragments according to the N-Me fragmentation patterns encoded in the script, which is theoretically applicable to all molecules containing C=C bonds (validated only on sterol lipids). Record the computed fragment-ion information — m/z and relative intensity — for each derivatized sterol into a structured table that is combined with retention time and CCS predictions into the 4D sterol lipid database (n = 4891). Validate by running the script end-to-end on the sterol lipid set and confirming every output record contains fragment m/z and relative-intensity fields consistent with the encoded N-Me fragmentation patterns.

## Related tools

- **RDkit** (Provides the built-in functions on which the MS/MS calculation script is written, for parsing sterol structures and recognizing double bond positions)
- **Jupyter Notebook** (Environment in which all MS/MS calculation functions are implemented and executed) — https://github.com/Chen-micslab/QCCAssisted4DSterol
- **Python** (Scripting language for the QCCAssisted4DSterol MS/MS calculation scripts) — https://github.com/Chen-micslab/QCCAssisted4DSterol

## Evaluation signals

- Each output record contains both fragment m/z and relative-intensity fields for every derivatized sterol processed
- The script executes end-to-end on the full unsaturated sterol lipid set without errors in the Jupyter Notebook environment
- Generated fragments are consistent with the N-Me fragmentation patterns encoded in the script for the recognized double bond positions
- Predicted fragment ions match observed MS/MS fragments during LC-IM-MS/MS based 4D sterolomics data processing and identification

## Limitations

- The script is theoretically applicable to all molecules including C=C bonds but has only been tested on sterol lipids
- Fragment generation depends on the specific N-Me fragmentation patterns encoded in the script; other derivatization chemistries are not supported
- Recognizes double bond positions only for N-Me derived (C═C bond-targeted) unsaturated sterol lipids

## Evidence

- [readme] The script is written on the basis of RDkit's built-in functions. The script  recognises double bond positions and generates MS/MS based on N-Me fragmentation patterns.: "The script is written on the basis of RDkit's built-in functions. The script  recognises double bond positions and generates MS/MS based on N-Me fragmentation patterns."
