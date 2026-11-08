---
name: n-methyl-derivatized-lipid-msms-simulation
description: Use when generating theoretical fragment-ion information (m/z and relative
  intensity) for N-Me derivatized unsaturated sterol lipids to build a fragment ion
  prediction component of a 4D sterol lipid database.
when_to_use_negative:
- The molecules of interest lack C=C double bonds (the script recognizes double bond
  positions and generates MS/MS based on N-Me fragmentation patterns, so it does not
  apply)
- You need experimentally measured MS/MS spectra rather than theoretical fragment-ion
  predictions, or you need CCS prediction — that is handled by the separate QCC-Assisted
  dataset CCS prediction workflow (scikit-learn LASSO + SVR)
edam_operation: http://edamontology.org/operation_3639
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3172
tools:
- name: RDkit
  role: Provides the built-in functions on which the MS/MS calculation script is written,
    for recognizing double bond positions and generating MS/MS based on N-Me fragmentation
    patterns
  repo: https://github.com/Chen-micslab/QCCAssisted4DSterol
- name: Jupyter Notebook
  role: Environment in which all functions of the MS/MS calculation module are implemented
  repo: https://github.com/Chen-micslab/QCCAssisted4DSterol
- name: Python
  role: Language of the QCCAssisted4DSterol script collection that contains the MS/MS
    calculation module
  repo: https://github.com/Chen-micslab/QCCAssisted4DSterol
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1002/anie.202507483
    title: Quantum Chemistry Calculation‐Assisted Large‐Scale Collision Cross Section
      Prediction Empowers Derivatization‐Enhanced Multidimensional Metabolomics
  source_content_hash: sha256:0084ddcba4fcd3db133c5131e12333023e83b62fd545c21fc8c58d111bcd79f1
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/Chen-micslab/QCCAssisted4DSterol
  tools:
  - Jupyter Notebook
  - Python
  - RDkit
  edam_operation: http://edamontology.org/operation_3639
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# n-methyl-derivatized-lipid-msms-simulation

## Summary

A Python/RDkit-based MS/MS calculation module from the QCCAssisted4DSterol repository that recognizes C=C double bond positions in N-methylpyridinium (N-Me) derivatized unsaturated sterol lipids and generates theoretical fragment-ion spectra according to N-Me fragmentation patterns.

## When to use

Use when generating theoretical fragment-ion information (m/z and relative intensity) for N-Me derivatized unsaturated sterol lipids to build a fragment ion prediction component of a 4D sterol lipid database.

## When NOT to use

- The molecules of interest lack C=C double bonds (the script recognizes double bond positions and generates MS/MS based on N-Me fragmentation patterns, so it does not apply)
- You need experimentally measured MS/MS spectra rather than theoretical fragment-ion predictions, or you need CCS prediction — that is handled by the separate QCC-Assisted dataset CCS prediction workflow (scikit-learn LASSO + SVR)

## Inputs

- N-Me derivatized unsaturated sterol lipid structures (parsed with RDkit, containing C=C bond positions)
- N-Me fragmentation patterns encoded in the script

## Outputs

- Theoretical MS/MS fragment-ion records per derivatized sterol (fragment m/z and relative intensity)
- Fragment ion predictions contributing to the large-scale 4D information database of derivatized sterol lipids (n = 4891)

## How to apply

Run the MS/MS calculation script (implemented in Jupyter Notebook on the basis of RDkit's built-in functions) from the QCCAssisted4DSterol repository: load derivatized unsaturated sterol lipid structures, parse them with RDkit, and let the script recognize double bond positions in each sterol backbone. It then generates MS/MS spectra according to N-Me fragmentation patterns, producing computed fragment-ion m/z and relative-intensity values for each derivatized sterol. These predicted fragments feed into the construction of the 4D sterol lipid database (n = 4891 derivatized sterols) that also combines retention time and CCS predictions for LC-IM-MS/MS based sterol identification. Although theoretically applicable to all molecules containing C=C bonds, the script was only tested on sterol lipids.

## Related tools

- **RDkit** (Provides the built-in functions on which the MS/MS calculation script is written, for recognizing double bond positions and generating MS/MS based on N-Me fragmentation patterns) — https://github.com/Chen-micslab/QCCAssisted4DSterol
- **Jupyter Notebook** (Environment in which all functions of the MS/MS calculation module are implemented) — https://github.com/Chen-micslab/QCCAssisted4DSterol
- **Python** (Language of the QCCAssisted4DSterol script collection that contains the MS/MS calculation module) — https://github.com/Chen-micslab/QCCAssisted4DSterol

## Examples

```
jupyter notebook execute QCCAssisted4DSterol MS/MS calculation script on N-Me derivatized sterol structures (run within the repository's Jupyter Notebook environment; see github.com/Chen-micslab/QCCAssisted4DSterol)
```

## Evaluation signals

- The script executes end-to-end on the derivatized sterol lipid set in the Jupyter Notebook environment
- Each output record contains fragment m/z and relative-intensity fields consistent with the N-Me fragmentation patterns encoded in the script
- Predicted fragment ions correctly reflect the double bond positions recognized by RDkit for each sterol backbone
- The resulting fragment ion predictions combine coherently with RT and CCS predictions in the 4D database of derivatized sterol lipids (n = 4891)

## Limitations

- The script is 'theoretically applicable to all molecules including C=C bond (only test sterol lipids)' — non-sterol molecules are untested
- Fragmentation is limited to N-Me derivatized lipids; other derivatization chemistries would require different fragmentation patterns

## Evidence
