---
name: qtof-spectrum-formula-prediction
description: Use when input is Q-TOF MS/MS spectra (MGF with title, precursor_mz,
  precursor_type, collision_energy) and the goal is top-k chemical formula identification,
  optionally ablation of augmentation/contrastive loss/post-processing.
when_to_use_negative:
- Spectra come from Orbitrap instruments — no data augmentation was applied there
  (noise addition lowered similarity), and the 5 ppm tolerance applies instead of
  10 ppm
- Spectra have fewer than five peaks or m/z above 1500 — these are excluded as unreliable
  or rare by the pipeline filters
- The task is structural elucidation rather than formula prediction — FIDDLE only
  provides the initial formula constraint step
edam_operation: http://edamontology.org/operation_3802
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: FIDDLE
  role: Deep learning TCN model predicting formulas from Q-TOF MS/MS, with breadth-first
    refinement and rescore-model ranking
  repo: https://github.com/JosieHong/FIDDLE
- name: msbuddy
  role: Optional bottom-up candidate formula generator (BUDDY) whose candidates can
    be incorporated in file-based workflows
  repo: https://github.com/Philipbear/msbuddy
- name: msfiddle
  role: PyPI CLI/Python API for running pre-trained FIDDLE predictions from MGF files
  repo: https://github.com/josiehong/msfiddle
- name: SIRIUS
  role: Optional external top-down tool whose candidate outputs can be combined with
    FIDDLE
  repo: https://github.com/sirius-ms/sirius
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-025-66060-9
    title: 'FIDDLE: a deep learning method for chemical formulas prediction from tandem
      mass spectra'
  source_content_hash: sha256:61528b348681c23a686f8133496ef10f38129f8091f70663f547f047d840504a
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/JosieHong/FIDDLE
  repo_url: https://github.com/JosieHong/FIDDLE
  tools:
  - FIDDLE
  - Q-TOF
  - SIRIUS
  - msbuddy
  - msfiddle
  edam_operation: http://edamontology.org/operation_3802
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6470588235294118
n_claims: 17
---

# qtof-spectrum-formula-prediction

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Predict molecular formulas from Q-TOF tandem mass spectra using FIDDLE, a deep learning pipeline that encodes MS/MS with dilated convolutions, refines predictions via breadth-first search against the precursor mass with SENIOR-rule filtering, and ranks candidates with a secondary confidence-scoring model.

## When to use

Use when input is Q-TOF MS/MS spectra (MGF with title, precursor_mz, precursor_type, collision_energy) and the goal is top-k chemical formula identification, optionally ablation of augmentation/contrastive loss/post-processing.

## When NOT to use

- Spectra come from Orbitrap instruments — no data augmentation was applied there (noise addition lowered similarity), and the 5 ppm tolerance applies instead of 10 ppm
- Spectra have fewer than five peaks or m/z above 1500 — these are excluded as unreliable or rare by the pipeline filters
- The task is structural elucidation rather than formula prediction — FIDDLE only provides the initial formula constraint step

## Inputs

- MoNA Q-TOF MS/MS spectra in MGF format with title, precursor_mz, precursor_type, and collision_energy fields
- Pre-trained Q-TOF checkpoints (fiddle_tcn_qtof.pt, fiddle_rescore_qtof.pt)
- Q-TOF config files from ./config/

## Outputs

- Top-1 and top-5 formula predictions per spectrum
- Missed-atom histograms of predictions before and after post-processing
- ROC curve and AUC of confidence scores distinguishing correct from incorrect formulas (~0.97 on MoNA Q-TOF)
- Top-5 candidate coverage of correct formulas (>84%)

## How to apply

Create the conda environment from environment.yml and fetch the Q-TOF weights (fiddle_tcn_qtof.pt, fiddle_rescore_qtof.pt) via running_scripts/download_models.sh. Convert MoNA Q-TOF spectra to MGF format with title, precursor_mz, precursor_type, and collision_energy fields, applying the standard filters (>=5 peaks, m/z in (0, 1500], peaks below 50 m/z removed, 10 ppm precursor mass tolerance). Run run_fiddle.py so the TCN predicts formulas, breadth-first refinement adjusts atomic compositions to align with the precursor mass under SENIOR rules, and the rescore model (train_rescore.py) ranks candidates by predicted confidence scores. For ablations, train variants with train_tcn_gpus.py and train_tcn_gpus_cl.py: 'Pred w/o DA' (no Gaussian noise, mean 0 sd 0.1, on intensities) and 'Pred w/o CL' (no contrastive loss). Evaluate on the held-out test split split by molecular canonical SMILES to avoid compound leakage.

## Related tools

- **FIDDLE** (Deep learning TCN model predicting formulas from Q-TOF MS/MS, with breadth-first refinement and rescore-model ranking) — https://github.com/JosieHong/FIDDLE
- **msbuddy** (Optional bottom-up candidate formula generator (BUDDY) whose candidates can be incorporated in file-based workflows) — https://github.com/Philipbear/msbuddy
- **msfiddle** (PyPI CLI/Python API for running pre-trained FIDDLE predictions from MGF files) — https://github.com/josiehong/msfiddle
- **SIRIUS** (Optional external top-down tool whose candidate outputs can be combined with FIDDLE) — https://github.com/sirius-ms/sirius

## Examples

```
conda env create -f environment.yml; bash running_scripts/download_models.sh; python run_fiddle.py --config ./config/qtof.yaml --input spectra.mgf
```

## Evaluation signals

- Post-processing top-5 candidate coverage of correct formulas exceeds 84% on the MoNA Q-TOF evaluation
- Confidence-score ROC AUC reaches ~0.97 for distinguishing correct from incorrect formulas
- With data augmentation, correctly predicted top-1 formulas rise from 0% to 9% (all atoms) and 18% to 26% (heavy atoms)
- Missed-atom histograms show the post-processing mass-alignment effect relative to raw TCN predictions
- Training/test split by molecular canonical SMILES confirms no compound leakage between train and test sets

## Limitations

- Accuracy varies across experimental conditions, especially those with less training data
- Performance degrades on chimeric (co-eluting) spectra as spectral mixing increases
- FDR control via confidence-score thresholds is proposed but not implemented; adduct type prediction is left for future work
- The v2.0.0 release redesigned the rescore model (Siamese architecture), which may change reproduction behavior

## Evidence
