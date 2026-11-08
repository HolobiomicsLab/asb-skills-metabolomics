---
name: shiftx2-benchmark-comparison
description: Use when evaluating or validating a protein chemical shift predictor
  against a published baseline (SHIFTX+), or when reproducing the NMRFx paper's per-atom-type
  RMSD/MAE benchmark on the ShiftX2 test set.
when_to_use_negative:
- You need RNA chemical shift prediction — the paper uses separate FM models trained
  with stochastic gradient descent on a 371 RNA molecule dataset, not the protein
  LARS regression models
- Your goal is NMR data processing or peak assignment (NMRFx Processor workflow),
  not chemical shift predictor benchmarking
- Your target proteins contain paramagnetic centers or non-standard residues — they
  fall outside the 1480-protein training set's scope
edam_operation: http://edamontology.org/operation_0262
edam_topics:
- http://edamontology.org/topic_0593
- http://edamontology.org/topic_0092
tools:
- name: NMRFx
  role: Protein chemical shift prediction engine whose models are trained on the BMRB/PDB
    dataset and evaluated on the ShiftX2 test set
  repo: github.com/nanalysis/nmrfx
- name: SHIFTX+
  role: Established comparator program providing the baseline per-atom-type RMSD/MAE
    and the 61-protein test dataset
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s42004-025-01812-8
    title: NMR data processing, visualization, analysis and structure calculation
      with NMRFx
  source_content_hash: sha256:4cf71bd4f821a4aa4cdca91a60f70c2c48704891e736eebbd692bbaaab6360e3
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
  - SHIFTX+
  edam_operation: http://edamontology.org/operation_0262
  edam_topics:
  - http://edamontology.org/topic_0593
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 15
---

# shiftx2-benchmark-comparison

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmarks NMRFx's protein chemical shift prediction model against the established SHIFTX+ program on the 61-protein ShiftX2 test dataset, computing per-atom-type RMSD and MAE against experimentally assigned BMRB shifts.

## When to use

Use when evaluating or validating a protein chemical shift predictor against a published baseline (SHIFTX+), or when reproducing the NMRFx paper's per-atom-type RMSD/MAE benchmark on the ShiftX2 test set.

## When NOT to use

- You need RNA chemical shift prediction — the paper uses separate FM models trained with stochastic gradient descent on a 371 RNA molecule dataset, not the protein LARS regression models
- Your goal is NMR data processing or peak assignment (NMRFx Processor workflow), not chemical shift predictor benchmarking
- Your target proteins contain paramagnetic centers or non-standard residues — they fall outside the 1480-protein training set's scope

## Inputs

- Experimental chemical shift assignments from BMRB for proteins with matching PDB entries
- 3D protein structures from PDB (non-paramagnetic, standard residues only)
- 61-protein ShiftX2 test dataset (previously curated test set referenced in the paper)

## Outputs

- Per-atom-type RMSD and MAE table (table_S1_rmsd_mae.csv) mirroring Table S1 of the SI
- Trained per-atom-class LARS-regularized linear regression prediction models

## How to apply

Assemble the training corpus by downloading experimental chemical shift assignments from BMRB and 3D structures from PDB for the 1480 non-paramagnetic proteins comprising standard residues with matching BMRB and PDB entries. For each amino-acid and atom-type class (e.g. the atom classes documented for glutamine residues in Table S6 of the SI), fit an LARS-regularized linear regression on the structural and sequence attributes used for chemical shift prediction. Apply the trained NMRFx models to predict shifts for the 61 proteins of the ShiftX2 test dataset, then compare predicted shifts versus experimentally assigned shifts on a per-atom-type basis, computing RMSD and MAE for each backbone atom type (1H, 13C, 15N nuclei classes). Decision point: the paper defers the actual numbers to Table S1 of the SI, so agreement in magnitude with Table S1 is the acceptance check; overall backbone performance similar to SHIFTX+ is the expected outcome.

## Related tools

- **NMRFx** (Protein chemical shift prediction engine whose models are trained on the BMRB/PDB dataset and evaluated on the ShiftX2 test set) — github.com/nanalysis/nmrfx
- **SHIFTX+** (Established comparator program providing the baseline per-atom-type RMSD/MAE and the 61-protein test dataset)

## Evaluation signals

- table_S1_rmsd_mae.csv contains both an RMSD and an MAE entry for every evaluated atom type
- Per-atom-type RMSD/MAE values are consistent in magnitude with the corresponding entries in Table S1 of the SI (the paper's acceptance check)
- All 61 proteins of the ShiftX2 test dataset are covered — no missing predictions or dropped structures
- Prediction models exist for every amino-acid/atom-type class, e.g. the glutamine atom classes listed in Table S6 of the SI
- Overall backbone performance is comparable to SHIFTX+, per the paper's reported finding

## Limitations

- Exact RMSD and MAE values are deferred to Table S1 of the SI and are not reproduced in the main text, so exact-value verification requires the supplementary material
- Chemical shift prediction attribute values are only documented for glutamine atom classes (Table S6), not for all residue types in the main text
- The training set is restricted to non-paramagnetic proteins comprising standard residues with matching BMRB and PDB entries; proteins outside this scope are not covered
- NMRFx is under active development, so exact model behavior may change across versions

## Evidence
