---
name: cross-validated-rmsd-evaluation
description: Decision support for selecting or validating a chemical shift predictor
  (protein FM, RNA FM, or legacy SVR) when the target atom types are 1H, 13C, 15N
  and BMRB-assigned shifts plus matching PDB structures are available.
when_to_use_negative:
- Predictions on molecules that were part of the training set — in-sample RMSD inflates
  performance and is not the cross-validated metric NMRFx reports
- Atom classes or molecule types outside the evaluated scope (e.g. paramagnetic residues,
  non-standard residues, 31P shifts) — the training sets are defined as 1480 non-paramagnetic
  proteins comprising standard residues and 371 RNA molecules
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0593
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_0736
tools:
- name: NMRFx
  role: Application that implements the FM chemical shift predictors, the cross-validation
    evaluation against assigned BMRB shifts, and the reporting of per-atom-type RMSD
  repo: https://github.com/nanalysis/nmrfx
- name: Tribuo
  role: Java machine-learning library whose stochastic gradient descent implementation
    was used to train the FM shift prediction models that are then evaluated by cross-validated
    RMSD
- name: BMRB
  role: Source of experimentally assigned chemical shifts used as the ground truth
    for RMSD computation in both the protein (1480 proteins) and RNA (371 molecules)
    training datasets
- name: PDB
  role: Source of structural coordinates paired with BMRB assignments; structural
    features from these coordinates feed the predictors whose cross-validated RMSD
    is then computed
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s42004-025-01812-8
    title: NMR data processing, visualization, analysis and structure calculation
      with NMRFx
  source_content_hash: sha256:112b01cfa7ff5ae8e7a6dbf74d40216929d629437d2a36f1c3029bed171478b9
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/nanalysis/nmrfx
  repo_url: https://github.com/nanalysis/nmrfx
  tools:
  - BMRB
  - NMRFx
  - PDB
  - Tribuo
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0593
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_0736
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# cross-validated-rmsd-evaluation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

This skill reproduces the NMRFx cross-validated RMSD evaluation used to benchmark its machine-learning chemical shift predictors (FM models for RNA, prior SVR/NMRViewJ predictors for proteins and RNA) against experimentally assigned BMRB shifts, enabling like-for-like comparison of predictor performance on held-out molecules.

## When to use

Decision support for selecting or validating a chemical shift predictor (protein FM, RNA FM, or legacy SVR) when the target atom types are 1H, 13C, 15N and BMRB-assigned shifts plus matching PDB structures are available.

## When NOT to use

- Predictions on molecules that were part of the training set — in-sample RMSD inflates performance and is not the cross-validated metric NMRFx reports
- Atom classes or molecule types outside the evaluated scope (e.g. paramagnetic residues, non-standard residues, 31P shifts) — the training sets are defined as 1480 non-paramagnetic proteins comprising standard residues and 371 RNA molecules

## Inputs

- BMRB chemical shift assignments for RNA or protein molecules (371 RNA molecules / 1480 non-paramagnetic proteins with standard residues and matching BMRB and PDB entries)
- Matching PDB structures from which per-atom structural features are derived for the FM/SVR predictors
- Out-of-fold shift predictions generated per atom class (1H, 13C, 15N) under cross-validation
- Prior SVR model predictions trained on the same dataset (for FM-vs-SVR comparison)

## Outputs

- Cross-validated RMSD of predicted vs experimentally assigned shifts per atom type (1H, 13C, 15N) — e.g. an SI evaluation table for the RNA FM model
- RMSD and MAE for protein predictors on a held-out test set (e.g. Table S1 of the SI for the 61-protein ShiftX2 dataset)
- Side-by-side RMSD comparison between updated FM models and prior SVR models on the same training data
- Example downstream artifact: rna_fm_crossvalidated_rmsd.csv

## How to apply

Hold out molecules via cross-validation rather than reporting training-set residuals: NMRFx trained its FM shift predictors with stochastic gradient descent as implemented in the Tribuo library, one model per atom class (1H, 13C, 15N), then computed cross-validated RMSD for each atom type against experimentally assigned BMRB shifts. For the RNA predictor the training dataset is 371 RNA molecules with BMRB assignments and associated PDB structures; for the protein predictor the training dataset is 1480 non-paramagnetic proteins with standard residues and matching BMRB and PDB entries. Run cross-validation to produce out-of-fold shift predictions, compute per-atom-type RMSD between predicted and experimentally assigned shifts, and (when comparing model families) train the prior SVR models on the same data and report both RMSD tables side-by-side. A similar evaluation used a 61-protein test dataset previously used for ShiftX2, where both RMSD and MAE are reported. Use the same BMRB/PDB pairing rules across models so RMSD differences reflect the predictor, not the data slice.

## Related tools

- **NMRFx** (Application that implements the FM chemical shift predictors, the cross-validation evaluation against assigned BMRB shifts, and the reporting of per-atom-type RMSD) — https://github.com/nanalysis/nmrfx
- **Tribuo** (Java machine-learning library whose stochastic gradient descent implementation was used to train the FM shift prediction models that are then evaluated by cross-validated RMSD)
- **BMRB** (Source of experimentally assigned chemical shifts used as the ground truth for RMSD computation in both the protein (1480 proteins) and RNA (371 molecules) training datasets)
- **PDB** (Source of structural coordinates paired with BMRB assignments; structural features from these coordinates feed the predictors whose cross-validated RMSD is then computed)

## Evaluation signals

- Per-atom-type (1H, 13C, 15N) cross-validated RMSD values reproduce the SI evaluation table for the RNA FM model
- The FM model's cross-validated RMSD matches or improves on the prior SVR models' RMSD when both are trained on the same 371-RNA-molecule dataset
- Same BMRB/PDB pairing rules used for both FM and SVR models, so RMSD differences are attributable to the predictor, not the data slice
- Protein evaluation on the 61-protein ShiftX2 test set yields both RMSD and MAE (Table S1 of the SI)
- No training-set molecules leak into the evaluation fold (out-of-fold predictions only)

## Limitations

- Cross-validated RMSD tables are reported in the Supplementary Information, not in the main text — Table S1 (protein RMSD/MAE) and the RNA atom-type evaluation table live in the SI
- Only 1H, 13C, 15N atom types are evaluated for the RNA FM model; other nuclei (e.g. 31P) are outside the reported evaluation
- The protein training set is restricted to 1480 non-paramagnetic proteins comprising standard residues with matching BMRB and PDB entries, so the metric does not characterize performance on paramagnetic or non-standard residues
- The article does not state the specific cross-validation scheme (fold count, stratification) used to produce the out-of-fold predictions

## Evidence
