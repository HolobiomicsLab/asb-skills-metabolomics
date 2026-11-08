---
name: sterol-fragmentation-pattern-modeling
description: Use when you need theoretical MS/MS fragment ions for N-Me derivatized
  unsaturated sterol lipids whose C=C double bond positions are known, e.g. to build
  a searchable fragment ion database for isomer-level sterolomics.
when_to_use_negative:
- The target molecules lack C=C bonds, since the script's recognition and fragmentation
  logic is built around C═C bond-targeted N-Me derivatization and was only tested
  on sterol lipids
- You need collision cross section (CCS) prediction or retention time prediction rather
  than fragment ion prediction — use the QCC-assisted SVR/LASSO workflow or the GNN
  RT prediction workflow instead
- The input is raw LC-IM-MS/MS data requiring spectral processing and sterol identification
  rather than in silico fragment generation
edam_operation: http://edamontology.org/operation_3200
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: RDkit
  role: Provides the built-in functions used by the script to recognize double bond
    positions and generate MS/MS spectra based on N-Me fragmentation patterns
- name: Jupyter Notebook
  role: Environment in which all functions of the MS/MS calculation workflow are implemented
- name: Python
  role: Language of the QCCAssisted4DSterol script collection that implements the
    MS/MS calculation
  repo: https://github.com/Chen-micslab/QCCAssisted4DSterol
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1002/anie.202507483
    title: Quantum Chemistry Calculation‐Assisted Large‐Scale Collision Cross Section
      Prediction Empowers Derivatization‐Enhanced Multidimensional Metabolomics
  source_content_hash: sha256:d70dd32292b98be2329db78ef90a2b4092ecf80dc11a033ee8b619cf4334866d
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/Chen-micslab/QCCAssisted4DSterol
  tools:
  - Jupyter Notebook
  - Python
  - RDkit
  edam_operation: http://edamontology.org/operation_3200
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# sterol-fragmentation-pattern-modeling

## Summary

RDkit-based MS/MS calculation that recognizes C=C double bond positions in N-Me derivatized unsaturated sterol lipids and generates theoretical fragment-ion spectra (m/z and relative intensity) according to N-Me fragmentation patterns. It supplies the fragment ion dimension of the 4D sterol lipid database (n = 4891) used for LC-IM-MS/MS-based sterol identification.

## When to use

Use when you need theoretical MS/MS fragment ions for N-Me derivatized unsaturated sterol lipids whose C=C double bond positions are known, e.g. to build a searchable fragment ion database for isomer-level sterolomics.

## When NOT to use

- The target molecules lack C=C bonds, since the script's recognition and fragmentation logic is built around C═C bond-targeted N-Me derivatization and was only tested on sterol lipids
- You need collision cross section (CCS) prediction or retention time prediction rather than fragment ion prediction — use the QCC-assisted SVR/LASSO workflow or the GNN RT prediction workflow instead
- The input is raw LC-IM-MS/MS data requiring spectral processing and sterol identification rather than in silico fragment generation

## Inputs

- N-Me derivatized unsaturated sterol lipid structures (parsed with RDkit) with defined C=C double bond positions
- N-Me fragmentation pattern rules encoded in the Jupyter Notebook script

## Outputs

- Structured table of fragment-ion m/z and relative intensity for each derivatized sterol
- Theoretical MS/MS spectra feeding the fragment ion prediction component of the 4D sterol lipid database (n = 4891)

## How to apply

Load the N-Me derivatized unsaturated sterol lipid structures into a Jupyter Notebook and parse them with RDkit, since the script is written on RDkit's built-in functions. Use the script to recognize the double bond positions in each sterol backbone, then generate theoretical MS/MS fragments according to the N-Me fragmentation patterns encoded for C═C bond-targeted derivatization. Record the computed fragment-ion information (m/z and relative intensity) for each derivatized sterol into a structured table that can be combined with retention time and CCS predictions into the 4D sterol lipid database. The script is theoretically applicable to all molecules containing C=C bonds, but was only tested on sterol lipids, so verify fragment output against known N-Me sterol fragmentation behavior before reusing it on other lipid classes.

## Related tools

- **RDkit** (Provides the built-in functions used by the script to recognize double bond positions and generate MS/MS spectra based on N-Me fragmentation patterns)
- **Jupyter Notebook** (Environment in which all functions of the MS/MS calculation workflow are implemented)
- **Python** (Language of the QCCAssisted4DSterol script collection that implements the MS/MS calculation) — https://github.com/Chen-micslab/QCCAssisted4DSterol

## Evaluation signals

- The script executes end-to-end in the Jupyter Notebook environment on the full unsaturated sterol lipid set without errors
- Each output record contains both fragment m/z and relative-intensity fields consistent with the N-Me fragmentation patterns encoded in the script
- Recognized double bond positions in the RDkit-parsed sterol structures match the annotated C═C bond positions of the derivatized sterols
- Generated fragments cover the expected N-Me fragmentation ions for derivatized sterols and can be matched against LC-IM-MS/MS spectra in the ./Search/tissue identification workflow

## Limitations

- The script is only tested on sterol lipids; applicability to all molecules containing C=C bonds is theoretical
- Only N-Me fragmentation patterns are implemented, so fragment prediction is specific to C═C bond-targeted N-Me derivatized sterols
- No changelog is available for the source repository, limiting traceability of script updates

## Evidence
