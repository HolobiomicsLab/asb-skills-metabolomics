---
name: factorization-machine-sgd-training
description: Use when building or retraining an RNA or protein chemical shift prediction
  model from BMRB assignments paired with PDB structures, and you need an FM (rather
  than SVR) predictor evaluated by cross-validated RMSD per atom type.
when_to_use_negative:
- When the goal is peak picking, assignment, or spectrum processing rather than chemical
  shift prediction — use the corresponding NMRFx processing/assignment workflows instead
- When no paired BMRB chemical shifts and PDB structures are available for the molecules
  of interest, since training features cannot be assembled
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_0593
tools:
- name: NMRFx
  role: Provides the chemical shift prediction functionality in which the FM models
    are trained and evaluated
  repo: github.com/nanalysis/nmrfx
- name: Tribuo
  role: Java ML library implementing the stochastic gradient descent used to train
    the FM models
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s42004-025-01812-8
    title: NMR data processing, visualization, analysis and structure calculation
      with NMRFx
  source_content_hash: sha256:3a60d73a0d708699e54c1721a5a773503691376019fafda9032e58f69213f268
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
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_0593
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.875
n_claims: 16
---

# factorization-machine-sgd-training

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Train Factorization Machine (FM) chemical-shift predictors with stochastic gradient descent in the Tribuo library, using paired BMRB chemical shifts and PDB-derived structural features from 371 RNA molecules (or 1480 proteins), and validate via cross-validated per-atom-type RMSD against experimentally assigned shifts.

## When to use

Use when building or retraining an RNA or protein chemical shift prediction model from BMRB assignments paired with PDB structures, and you need an FM (rather than SVR) predictor evaluated by cross-validated RMSD per atom type.

## When NOT to use

- When the goal is peak picking, assignment, or spectrum processing rather than chemical shift prediction — use the corresponding NMRFx processing/assignment workflows instead
- When no paired BMRB chemical shifts and PDB structures are available for the molecules of interest, since training features cannot be assembled

## Inputs

- BMRB chemical shift assignments for 371 RNA molecules (RNA case) or 1480 non-paramagnetic proteins with standard residues and matching BMRB/PDB entries (protein case)
- Associated PDB structures from which structural features are computed

## Outputs

- Trained FM shift prediction models, one per atom class (1H, 13C, 15N)
- Cross-validated per-atom-type RMSD values comparing predicted vs. experimentally assigned shifts
- SVR-vs-FM comparison table (as reported in the SI evaluation table)

## How to apply

Retrieve the BMRB chemical shift assignments and their associated PDB structures for the training set — for RNA the current dataset consists of 371 molecules, and for proteins a dataset of 1480 non-paramagnetic proteins with standard residues and matching BMRB/PDB entries is used. Pair each assigned shift with structural features computed from the corresponding PDB coordinates. Train one FM model per atom class (1H, 13C, 15N) with stochastic gradient descent as implemented in the Tribuo library. Run cross-validation over the full dataset to obtain out-of-fold shift predictions for each atom type. Compute cross-validated RMSD between predicted and experimentally assigned shifts, and optionally train the prior SVR models on the same data to reproduce the FM-vs-SVR comparison reported in the SI table.

## Related tools

- **NMRFx** (Provides the chemical shift prediction functionality in which the FM models are trained and evaluated) — github.com/nanalysis/nmrfx
- **Tribuo** (Java ML library implementing the stochastic gradient descent used to train the FM models)

## Evaluation signals

- Cross-validated RMSD per atom type (1H, 13C, 15N) should match the values in the SI evaluation table
- FM model should maintain high performance while handling the increased training data points, supporting the reported finding over the prior SVR model
- All three atom classes (1H, 13C, 15N) must produce a trained FM model and a cross-validated RMSD value
- Protein training set must contain 1480 non-paramagnetic proteins with standard residues and matching BMRB/PDB entries; RNA training set must contain 371 molecules

## Limitations

- Chemical shift prediction attribute values are only listed in Table S6 of the SI, not in the main text
- The per-atom-type cross-validated RMSD evaluation table is only provided in the SI, not in the article body
- Dataset size and composition are fixed to the described curated sets (371 RNA molecules; 1480 non-paramagnetic proteins) — other datasets are not covered here

## Evidence
