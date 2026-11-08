---
name: lars-regularized-linear-regression-training
description: Use when building or retraining a protein chemical shift predictor from
  a large set of BMRB/PDB-matched proteins, or when reproducing/validating per-atom-type
  predicted-vs-experimental shift RMSD and MAE benchmarks against SHIFTX+.
when_to_use_negative:
- 'The target shifts are for RNA: NMRFx trains its RNA chemical shift predictor with
  FM models via stochastic gradient descent (Tribuo library), not LARS-regularized
  linear regression.'
- 'The proteins contain non-standard residues or paramagnetic centers: these were
  explicitly excluded from the 1480-protein training set, so the fitted models are
  out of scope.'
- 'You need an out-of-the-box shift prediction without retraining: use the existing
  NMRFx chemical shift prediction feature directly rather than refitting the regression
  models.'
edam_operation: http://edamontology.org/operation_3658
edam_topics:
- http://edamontology.org/topic_0593
tools:
- name: NMRFx
  role: Protein chemical shift prediction framework whose per-atom-class LARS-regularized
    linear regression models are trained on the 1480-protein BMRB/PDB dataset and
    benchmarked on the 61-protein ShiftX2 test set
  repo: github.com/nanalysis/nmrfx
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s42004-025-01812-8
    title: NMR data processing, visualization, analysis and structure calculation
      with NMRFx
  source_content_hash: sha256:e5170aa81374fa744a50323d5881c7597c64c35381c26ef2b41709270baa73cf
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
  edam_operation: http://edamontology.org/operation_3658
  edam_topics:
  - http://edamontology.org/topic_0593
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# LARS-regularized linear regression training

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Trains per-atom-class LARS-regularized linear regression models that predict protein NMR chemical shifts from structural and sequence attributes, as used in NMRFx's protein chemical shift predictor, which was benchmarked against SHIFTX+ on the 61-protein ShiftX2 test dataset.

## When to use

Use when building or retraining a protein chemical shift predictor from a large set of BMRB/PDB-matched proteins, or when reproducing/validating per-atom-type predicted-vs-experimental shift RMSD and MAE benchmarks against SHIFTX+.

## When NOT to use

- The target shifts are for RNA: NMRFx trains its RNA chemical shift predictor with FM models via stochastic gradient descent (Tribuo library), not LARS-regularized linear regression.
- The proteins contain non-standard residues or paramagnetic centers: these were explicitly excluded from the 1480-protein training set, so the fitted models are out of scope.
- You need an out-of-the-box shift prediction without retraining: use the existing NMRFx chemical shift prediction feature directly rather than refitting the regression models.

## Inputs

- Experimental chemical shift assignments from BMRB for 1480 non-paramagnetic, standard-residue proteins
- 3D structures from PDB for the same BMRB/PDB-matched proteins (training features)
- Structural and sequence attribute values per atom class (documented per residue type, e.g. glutamine in Table S6 of the SI)
- 3D structures for the 61 proteins of the ShiftX2 test dataset

## Outputs

- One fitted LARS-regularized linear regression model per amino-acid/atom-type class
- Predicted chemical shifts for the 61-protein test dataset
- table_S1_rmsd_mae.csv with per-atom-type RMSD and MAE values mirroring Table S1

## How to apply

Assemble the training set from the 1480 non-paramagnetic proteins comprising standard residues with matching BMRB (shifts) and PDB (3D structures) entries. Partition shifts into amino-acid and atom-type classes (e.g., the glutamine atom classes whose attribute values are documented in Table S6 of the SI) and fit a separate LARS-regularized linear regression per class, using the structural and sequence attributes used for chemical shift prediction. Apply the fitted models to the 61 proteins of the ShiftX2 test dataset to generate predicted shifts. Compare predictions against experimentally assigned shifts on a per-atom-type basis, computing RMSD and MAE for each backbone atom type, and write results to table_S1_rmsd_mae.csv mirroring Table S1's layout. Decision point: because the paper defers exact numbers to the SI, accept the result only if each evaluated atom type has an RMSD/MAE entry of magnitude consistent with Table S1.

## Related tools

- **NMRFx** (Protein chemical shift prediction framework whose per-atom-class LARS-regularized linear regression models are trained on the 1480-protein BMRB/PDB dataset and benchmarked on the 61-protein ShiftX2 test set) — github.com/nanalysis/nmrfx

## Evaluation signals

- table_S1_rmsd_mae.csv contains both an RMSD and an MAE entry for every evaluated backbone atom type
- Per-atom-type RMSD/MAE values are consistent in magnitude with the corresponding entries in Table S1 of the SI (exact-value matching is the acceptance check since the paper defers numbers to the SI)
- Models were fit per amino-acid and atom-type class using the attribute set documented for each class (cf. glutamine atom classes in Table S6)
- Overall backbone performance is comparable to SHIFTX+ on the same 61-protein test dataset
- Training set is restricted to the 1480 non-paramagnetic, standard-residue proteins with matching BMRB and PDB entries — no paramagnetic or non-standard-residue proteins leak in

## Limitations

- RMSD and MAE values for the 61-protein test dataset are deferred to Table S1 of the SI, so exact benchmark numbers must be recovered from the SI to validate results.
- The attribute values used for prediction per atom class are only enumerated in Table S6 of the SI (e.g. for glutamine residues); the main text does not spell them out.
- Model applicability is limited to standard-residue, non-paramagnetic proteins, matching the composition of the training data.

## Evidence
