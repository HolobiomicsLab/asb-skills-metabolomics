---
name: rna-chemical-shift-prediction
description: Use when assigned RNA chemical shifts and a 3D RNA structure (PDB coordinates)
  are available and per-atom-type predicted shifts for 1H, 13C, or 15N are needed,
  e.g. for assignment validation or NMR structure assessment.
when_to_use_negative:
- Input is a protein chemical shift prediction task — NMRFx's protein shift predictor
  is a separate LARS-regularized linear-regression model trained on 1480 proteins
  and evaluated against SHIFTX+ on a 61-protein test set.
- No PDB structure is available for the RNA — the FM model derives structural features
  from PDB coordinates, so it cannot run on sequence-only input.
edam_operation: http://edamontology.org/operation_0570
edam_topics:
- http://edamontology.org/topic_0593
tools:
- name: NMRFx
  role: Runs the RNA chemical shift prediction models and integrates BMRB/PDB data
    handling; the FM RNA shift predictor replaces the prior NMRViewJ SVR predictors
  repo: github.com/nanalysis/nmrfx
- name: Tribuo
  role: Library used to train the FM shift prediction models with stochastic gradient
    descent
provenance:
  source_task_ids:
  - task_004
  - task_001
  source_papers:
  - doi: 10.1038/s42004-025-01812-8
    title: NMR data processing, visualization, analysis and structure calculation
      with NMRFx
  source_content_hash: sha256:54b51af1bc83f164547b294e476339e0899f051fd702a985af80dae90a2eb2cb
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/nanalysis/nmrfx
  tools:
  - NMRFx
  - PDB
  - Tribuo
  edam_operation: http://edamontology.org/operation_0570
  edam_topics:
  - http://edamontology.org/topic_0593
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6428571428571429
n_claims: 14
---

# rna-chemical-shift-prediction

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Predict RNA 1H, 13C, and 15N chemical shifts from PDB-derived structural features using NMRFx Factorization Machine (FM) models trained on 371 BMRB-assigned RNA molecules, replacing the older NMRViewJ SVR predictors.

## When to use

Use when assigned RNA chemical shifts and a 3D RNA structure (PDB coordinates) are available and per-atom-type predicted shifts for 1H, 13C, or 15N are needed, e.g. for assignment validation or NMR structure assessment.

## When NOT to use

- Input is a protein chemical shift prediction task — NMRFx's protein shift predictor is a separate LARS-regularized linear-regression model trained on 1480 proteins and evaluated against SHIFTX+ on a 61-protein test set.
- No PDB structure is available for the RNA — the FM model derives structural features from PDB coordinates, so it cannot run on sequence-only input.

## Inputs

- RNA chemical shift assignments from BMRB for 371 RNA molecules
- Associated PDB 3D structures (coordinates) for those 371 RNA molecules

## Outputs

- Predicted RNA chemical shifts per atom type (1H, 13C, 15N)
- Cross-validated RMSD per atom type (1H, 13C, 15N), reported in an SI evaluation table

## How to apply

Retrieve the BMRB chemical shift assignments and associated PDB structures for the 371 RNA molecules that constitute the training dataset. Pair each assigned RNA chemical shift with structural features computed from the corresponding PDB coordinates, then train factorization-machine (FM) shift prediction models with stochastic gradient descent as implemented in the Tribuo library, one model per atom class (1H, 13C, 15N). Validate by running cross-validation over the 371-molecule dataset to obtain out-of-fold shift predictions per atom type, computing the cross-validated RMSD between predicted and experimentally assigned shifts for each atom class, and comparing against the prior SVR models trained on the same data. The paper reports that 'The updated FM model robustly handles a significant increase in training data points while maintaining high performance.'

## Related tools

- **NMRFx** (Runs the RNA chemical shift prediction models and integrates BMRB/PDB data handling; the FM RNA shift predictor replaces the prior NMRViewJ SVR predictors) — github.com/nanalysis/nmrfx
- **Tribuo** (Library used to train the FM shift prediction models with stochastic gradient descent)

## Evaluation signals

- Per-atom-type cross-validated RMSD (1H, 13C, 15N) should match the SI evaluation table values.
- FM models trained with stochastic gradient descent (Tribuo) should handle the increased 371-molecule training set while maintaining performance comparable to the prior SVR model.
- Dataset used for training must be the 371 BMRB RNA molecules with associated PDB structures.
- Output includes RMSD for all three atom classes: 1H, 13C, and 15N.

## Limitations

- Cross-validated RMSD evaluation table is only provided in the SI, not in the main text.
- Dataset limited to non-paramagnetic standard residues; excludes RNA molecules lacking PDB structures.
- No DOI or changelog provided for the article in this context.

## Evidence
