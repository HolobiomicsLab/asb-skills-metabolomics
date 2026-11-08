---
name: rdkit-molecular-graph-manipulation
description: Use when you have derivatized sterol lipid structures (e.g., N-Me tagged
  C═C bonds) and need theoretical fragment-ion m/z and relative-intensity predictions
  for building an MS/MS spectral database.
when_to_use_negative:
- The molecule of interest contains no C═C double bond and was not N-Me derivatized
  — the script's double-bond recognition and N-Me fragmentation patterns are specific
  to unsaturated, derivatized sterols
- You need experimental MS/MS spectra rather than theoretical fragment predictions,
  or you need CCS/RT predictions rather than fragment ions (use the QCC-assisted SVR
  model or GNN RT prediction workflows instead)
edam_topics:
- http://edamontology.org/topic_3332
tools:
- name: RDkit
  role: Provides the built-in molecular graph functions used to parse sterol structures,
    recognize double bond positions, and generate MS/MS based on N-Me fragmentation
    patterns
- name: Python
  role: Language in which the QCCAssisted4DSterol MS/MS calculation scripts are written
  repo: https://github.com/Chen-micslab/QCCAssisted4DSterol
- name: Jupyter Notebook
  role: Environment in which all functions, including the MS/MS calculation, are implemented
  repo: https://github.com/Chen-micslab/QCCAssisted4DSterol
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1002/anie.202507483
    title: Quantum Chemistry Calculation‐Assisted Large‐Scale Collision Cross Section
      Prediction Empowers Derivatization‐Enhanced Multidimensional Metabolomics
  source_content_hash: sha256:2e4649c47a734795cb438f1eb24adfd4100d8f5febaa0fe41c0295ec71faf817
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
  - http://edamontology.org/topic_3332
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# rdkit-molecular-graph-maniplation

## Summary

Use RDkit built-in molecular graph functions to parse N-Me derivatized unsaturated sterol lipid structures, recognize C═C double bond positions, and generate theoretical MS/MS fragment ions (m/z and relative intensity) based on N-Me fragmentation patterns.

## When to use

Use when you have derivatized sterol lipid structures (e.g., N-Me tagged C═C bonds) and need theoretical fragment-ion m/z and relative-intensity predictions for building an MS/MS spectral database.

## When NOT to use

- The molecule of interest contains no C═C double bond and was not N-Me derivatized — the script's double-bond recognition and N-Me fragmentation patterns are specific to unsaturated, derivatized sterols
- You need experimental MS/MS spectra rather than theoretical fragment predictions, or you need CCS/RT predictions rather than fragment ions (use the QCC-assisted SVR model or GNN RT prediction workflows instead)

## Inputs

- N-Me derivatized unsaturated sterol lipid molecular structures (parseable by RDkit, e.g., SMILES/MOL representations)
- Encoded N-Me fragmentation patterns implemented in the Jupyter Notebook script

## Outputs

- Theoretical MS/MS spectra for each derivatized sterol
- Structured table of fragment-ion information: m/z and relative intensity per derivatized sterol

## How to apply

Load each N-Me derivatized unsaturated sterol lipid structure into a Jupyter Notebook and parse it with RDkit. Use RDkit built-in functions operating on the molecular graph to recognize the positions of C═C double bonds in the sterol backbone, since these determine where the N-Me derivatization attaches and how the molecule fragments. Apply the N-Me fragmentation patterns encoded in the script to generate theoretical MS/MS spectra for each derivatized sterol. Record the computed fragment-ion information (m/z and relative intensity) into a structured table per sterol. The approach is theoretically applicable to all molecules containing C=C bonds, though in this work it was validated only on sterol lipids, contributing fragment predictions to a 4D sterol lipid database (n = 4891) that also combines retention time and CCS predictions.

## Related tools

- **RDkit** (Provides the built-in molecular graph functions used to parse sterol structures, recognize double bond positions, and generate MS/MS based on N-Me fragmentation patterns)
- **Python** (Language in which the QCCAssisted4DSterol MS/MS calculation scripts are written) — https://github.com/Chen-micslab/QCCAssisted4DSterol
- **Jupyter Notebook** (Environment in which all functions, including the MS/MS calculation, are implemented) — https://github.com/Chen-micslab/QCCAssisted4DSterol

## Evaluation signals

- The script executes end-to-end on the full sterol lipid set without errors
- Each output record contains both fragment m/z and relative-intensity fields
- Generated fragments are consistent with the N-Me fragmentation patterns encoded in the script
- Recognized double bond positions match the known unsaturation sites of each input sterol structure

## Limitations

- Only tested on sterol lipids; theoretically applicable to all molecules containing C=C bonds but not validated beyond that scope
- Generates theoretical fragments from encoded N-Me patterns, not experimentally measured spectra

## Evidence
