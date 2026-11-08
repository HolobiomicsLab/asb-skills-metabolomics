---
name: roc-auc-evaluation
description: Use when a trained formula-prediction model emits multiple candidate
  formulas per spectrum and you need to quantify how well a confidence score discriminates
  correct from incorrect candidates (ROC AUC) on a held-out MS/MS test split.
when_to_use_negative:
- You have raw MS/MS spectra without generated candidate formulas — the rescore model
  only ranks candidates from the refinement/post-processing step, it does not predict
  formulas itself
- The goal is ablation of the TCN feature extractor (data augmentation / contrastive
  loss) rather than scoring candidate ranking quality — use the ablation workflow
  (experiments_ablation_study.sh) instead
- Input spectra are chimeric (co-eluting) mixtures — FIDDLE's performance, including
  candidate ranking, degrades as spectral mixing increases
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: FIDDLE
  role: Source of the rescore model, the MS/MS feature extractor, candidate formula
    generation, and the ROC AUC evaluation workflow (train_rescore.py, run_fiddle.py)
  repo: https://github.com/JosieHong/FIDDLE
- name: Q-TOF
  role: Instrument domain on which the confidence-score ROC AUC (target ~0.97) is
    evaluated, using the 10 ppm precursor mass tolerance branch
- name: msbuddy
  role: Optional BUDDY candidate-generation backend whose formula candidates can also
    be ranked by the FIDDLE rescore model in file-based workflows
  repo: https://github.com/Philipbear/msbuddy
provenance:
  source_task_ids:
  - task_004
  - task_007
  source_papers:
  - doi: 10.1038/s41467-025-66060-9
    title: 'FIDDLE: a deep learning method for chemical formulas prediction from tandem
      mass spectra'
  source_content_hash: sha256:7cfd2d2808ce1579fe389ec826fa210fe96e9eff0ca575e4192db4f4fbbc829e
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
  - msbuddy
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# roc-auc-evaluation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Evaluate FIDDLE's secondary confidence-scoring (rescore) model, which ranks candidate molecular formulas generated from MS/MS spectra, by measuring the ROC AUC of its confidence scores in separating correct from incorrect candidates.

## When to use

Use when a trained formula-prediction model emits multiple candidate formulas per spectrum and you need to quantify how well a confidence score discriminates correct from incorrect candidates (ROC AUC) on a held-out MS/MS test split.

## When NOT to use

- You have raw MS/MS spectra without generated candidate formulas — the rescore model only ranks candidates from the refinement/post-processing step, it does not predict formulas itself
- The goal is ablation of the TCN feature extractor (data augmentation / contrastive loss) rather than scoring candidate ranking quality — use the ablation workflow (experiments_ablation_study.sh) instead
- Input spectra are chimeric (co-eluting) mixtures — FIDDLE's performance, including candidate ranking, degrades as spectral mixing increases

## Inputs

- Condition-independent MS/MS spectral features learned by the FIDDLE TCN (per-spectrum)
- Candidate formula vectors (top-5 per spectrum from breadth-first refinement + SENIOR-rule post-processing)
- Instrument-specific config files from ./config/ (Q-TOF and Orbitrap)
- MoNA Q-TOF MS/MS spectra in MGF format (title, precursor_mz, precursor_type, collision_energy fields; >=5 peaks, m/z in (0,1500], peaks below 50 m/z removed, 10 ppm precursor tolerance for Q-TOF)

## Outputs

- Trained rescore weights at ./check_point/fiddle_rescore_qtof.pt and ./check_point/fiddle_rescore_orbitrap.pt
- Ranked top-5 candidate formulas per test spectrum with confidence scores
- ROC curve and ROC AUC of confidence scores separating correct from incorrect candidates (target AUC ~0.97 on MoNA Q-TOF)

## How to apply

Build training pairs by concatenating the condition-independent MS/MS spectral features with each candidate formula vector from the training spectra, then train the rescore network — in FIDDLE v1.x a five-layer fully connected architecture with hidden dimensions 416-208-104-26-13 and a Sigmoid output — separately for Orbitrap and Q-TOF spectra using train_rescore.py and the ./config/ files, writing weights to ./check_point/fiddle_rescore_orbitrap.pt and ./check_point/fiddle_rescore_qtof.pt. At evaluation time, score the top-5 candidate formulas per held-out spectrum (test sets split by molecular canonical SMILES to avoid compound leakage), rank candidates by predicted confidence score, and compute the ROC AUC treating each candidate as correct/incorrect. Validate on the MoNA Q-TOF evaluation split: the confidence-score ROC must reach an AUC of ~0.97 (Fig. 4d), with post-processing top-5 candidate coverage of the correct formula exceeding 84%. If using the FIDDLE v2.0.0+ codebase, expect the rescore model to have been redesigned as a Siamese architecture rather than the plain five-layer FC network.

## Related tools

- **FIDDLE** (Source of the rescore model, the MS/MS feature extractor, candidate formula generation, and the ROC AUC evaluation workflow (train_rescore.py, run_fiddle.py)) — https://github.com/JosieHong/FIDDLE
- **Q-TOF** (Instrument domain on which the confidence-score ROC AUC (target ~0.97) is evaluated, using the 10 ppm precursor mass tolerance branch)
- **msbuddy** (Optional BUDDY candidate-generation backend whose formula candidates can also be ranked by the FIDDLE rescore model in file-based workflows) — https://github.com/Philipbear/msbuddy

## Examples

```
python train_rescore.py --config ./config/qtof  # writes ./check_point/fiddle_rescore_qtof.pt
```

## Evaluation signals

- Confidence-score ROC AUC reaches ~0.97 on the MoNA Q-TOF evaluation set (Fig. 4d)
- Post-processing top-5 candidate coverage of the correct formula exceeds 84% of spectra
- Rescore weight files are written to ./check_point/fiddle_rescore_qtof.pt (and fiddle_rescore_orbitrap.pt for Orbitrap)
- Training/test split shows no shared molecular canonical SMILES between train and test compounds
- Ranked candidates respect SENIOR-rule validity and align with the precursor mass within the instrument tolerance (10 ppm Q-TOF, 5 ppm Orbitrap)

## Limitations

- The rescore architecture is version-dependent: FIDDLE v2.0.0 redesigned the rescore model as a Siamese architecture, so reproducing the five-layer 416-208-104-26-13 FC design requires v1.x
- FDR control via confidence-score thresholds is proposed but not implemented, so score cutoffs cannot yet be treated as calibrated error-rate estimates
- FIDDLE's performance (including candidate ranking) decreases on chimeric (co-eluting) spectra as spectral mixing increases, and accuracy varies across experimental conditions, especially those with less training data

## Evidence
