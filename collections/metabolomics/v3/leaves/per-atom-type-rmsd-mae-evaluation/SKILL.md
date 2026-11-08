---
name: per-atom-type-rmsd-mae-evaluation
description: Use when a predicted-vs-experimental chemical shift comparison must be
  quantified per atom class (e.g. backbone 1H, 13C, 15N) to benchmark a prediction
  model like NMRFx's against an established predictor.
when_to_use_negative:
- The evaluation target is RNA chemical shift prediction, whose model is assessed
  by cross-validated RMSD per atom type in a separate SI table, not this protein test-set
  protocol
- Input is a single protein's assignment quality check rather than a model benchmark
  over a fixed multi-protein test dataset
- The prediction model has not yet been trained on the 1480-protein BMRB/PDB training
  set — benchmarking before training yields meaningless RMSD/MAE
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0593
- http://edamontology.org/topic_3474
tools:
- name: NMRFx
  role: Provides the trained protein chemical shift prediction model whose predicted
    shifts are evaluated against BMRB experimental assignments via per-atom-type RMSD/MAE
  repo: github.com/nanalysis/nmrfx
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s42004-025-01812-8
    title: NMR data processing, visualization, analysis and structure calculation
      with NMRFx
  source_content_hash: sha256:b68affd3f551aa6c61247dee5e5737032831ac2e5b7e03817696fb8e6d27f498
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
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0593
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# per-atom-type-rmsd-mae-evaluation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmark a chemical shift prediction model against experimentally assigned shifts by computing per-atom-type RMSD and MAE on a fixed test set, as NMRFx does against SHIFTX+ on the 61-protein ShiftX2 test dataset, with results reported in a Table-S1-style CSV.

## When to use

Use when a predicted-vs-experimental chemical shift comparison must be quantified per atom class (e.g. backbone 1H, 13C, 15N) to benchmark a prediction model like NMRFx's against an established predictor.

## When NOT to use

- The evaluation target is RNA chemical shift prediction, whose model is assessed by cross-validated RMSD per atom type in a separate SI table, not this protein test-set protocol
- Input is a single protein's assignment quality check rather than a model benchmark over a fixed multi-protein test dataset
- The prediction model has not yet been trained on the 1480-protein BMRB/PDB training set — benchmarking before training yields meaningless RMSD/MAE

## Inputs

- Predicted chemical shifts for the 61-protein ShiftX2 test dataset (output of the trained NMRFx protein shift prediction model)
- Experimentally assigned chemical shifts from BMRB for the same 61 proteins
- Atom-class definitions (e.g. per-amino-acid, per-atom-type classes as in Table S6 of the SI)

## Outputs

- table_S1_rmsd_mae.csv with per-atom-type RMSD and MAE entries mirroring Table S1 of the SI
- A qualitative comparison statement of NMRFx vs SHIFTX+ performance across backbone atom types

## How to apply

First assemble a test set of proteins with both experimental BMRB shift assignments and PDB structures — the article uses the 61 proteins previously used for ShiftX2. Run the trained prediction model (NMRFx's protein chemical shift model, trained by LARS-regularized linear regression on 1480 non-paramagnetic, standard-residue proteins with matching BMRB/PDB entries) on each test protein's structure. For each atom type (atom classes such as those documented for glutamine in Table S6 of the SI), compare predicted shifts against experimentally assigned shifts and compute two error metrics: RMSD and MAE. Write one RMSD and one MAE entry per evaluated atom type to a CSV mirroring the layout of Table S1, since the paper defers the numbers to the SI. Validation is a schema/magnitude check: the CSV must contain an RMSD and MAE entry for every evaluated atom type, and values must be consistent in magnitude with the corresponding Table S1 entries; overall, NMRFx's performance should be similar to SHIFTX+ across backbone atom types.

## Related tools

- **NMRFx** (Provides the trained protein chemical shift prediction model whose predicted shifts are evaluated against BMRB experimental assignments via per-atom-type RMSD/MAE) — github.com/nanalysis/nmrfx

## Evaluation signals

- table_S1_rmsd_mae.csv contains an RMSD and a MAE entry for each evaluated atom type (schema completeness check)
- Per-atom-type RMSD/MAE values are consistent in magnitude with the corresponding entries in Table S1 of the SI (exact-value matching is the acceptance check)
- Overall performance across backbone atom types is similar to SHIFTX+ on the same 61-protein test dataset
- Test set is exactly the 61 proteins previously used for ShiftX2, not a resampled subset

## Limitations

- Exact RMSD and MAE values for the 61-protein test dataset are deferred to Table S1 of the SI and are not reported in the main text
- Training and test data are restricted to non-paramagnetic proteins comprising standard residues with matching BMRB and PDB entries
- The attribute values used for shift prediction per atom class are only documented in Table S6 of the SI

## Evidence
