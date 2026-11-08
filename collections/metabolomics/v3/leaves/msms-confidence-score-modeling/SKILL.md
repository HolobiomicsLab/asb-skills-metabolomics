---
name: msms-confidence-score-modeling
description: Use when ranking top-k candidate molecular formulas generated from tandem
  mass spectra requires a learned confidence score, e.g. training a rescore model
  on Orbitrap or Q-TOF MS/MS features.
when_to_use_negative:
- The task is de novo structure elucidation rather than ranking candidate molecular
  formulas — FIDDLE only predicts formulas, not structures
- A calibrated FDR-controlled identification at a fixed score threshold is required
  — confidence-threshold FDR control is proposed but not implemented in FIDDLE
- Precursor adduct type is unknown and must be predicted — adduct type prediction
  is not part of the current workflow
edam_operation: http://edamontology.org/operation_3802
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3474
tools:
- name: FIDDLE
  role: Provides the rescore network architecture, training script (train_rescore.py),
    configs, and MS/MS spectral features used to score candidate formulas
  repo: https://github.com/JosieHong/FIDDLE
- name: BUDDY (msbuddy)
  role: Optional source of candidate formulas from a reference library that can be
    rescored alongside FIDDLE candidates
  repo: https://github.com/Philipbear/msbuddy
- name: msfiddle
  role: PyPI package exposing pre-trained FIDDLE models for prediction and incorporating
    BUDDY and SIRIUS candidate outputs in file-based workflows
  repo: https://github.com/josiehong/msfiddle
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-025-66060-9
    title: 'FIDDLE: a deep learning method for chemical formulas prediction from tandem
      mass spectra'
  source_content_hash: sha256:e2f7805c1e42a60290cdbb3ea3e176491d13c53a6655d62811a2ace48b622578
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
  - BUDDY (msbuddy)
  - FIDDLE
  - msfiddle
  edam_operation: http://edamontology.org/operation_3802
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7857142857142857
n_claims: 14
---

# msms-confidence-score-modeling

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Train and apply FIDDLE's auxiliary rescore network that estimates confidence scores for candidate chemical formulas by combining condition-independent MS/MS spectral features with each candidate formula vector, and rank candidates by predicted confidence.

## When to use

Use when ranking top-k candidate molecular formulas generated from tandem mass spectra requires a learned confidence score, e.g. training a rescore model on Orbitrap or Q-TOF MS/MS features.

## When NOT to use

- The task is de novo structure elucidation rather than ranking candidate molecular formulas — FIDDLE only predicts formulas, not structures
- A calibrated FDR-controlled identification at a fixed score threshold is required — confidence-threshold FDR control is proposed but not implemented in FIDDLE
- Precursor adduct type is unknown and must be predicted — adduct type prediction is not part of the current workflow

## Inputs

- Condition-independent MS/MS spectral features from the FIDDLE formula identification model
- Candidate formula vectors from the breadth-first refinement step (top-5 candidates per spectrum)
- Training spectra with known correct formulas (NIST20, NIST23, MoNA, GNPS, Agilent PCDL compendium)
- Instrument type label (Orbitrap vs Q-TOF) to select the instrument-specific model
- Config files in ./config/ for train_rescore.py

## Outputs

- Trained rescore weights ./check_point/fiddle_rescore_orbitrap.pt
- Trained rescore weights ./check_point/fiddle_rescore_qtof.pt
- Confidence scores for each top-5 candidate formula per test spectrum
- Ranked candidate formula lists per spectrum

## How to apply

Implement the rescore network as a five-layer fully connected architecture with hidden dimensions 416-208-104-26-13 and a Sigmoid output layer in PyTorch. Build training pairs by concatenating condition-independent MS/MS spectral features learned during formula identification with each candidate formula vector derived from the training spectra (NIST20, NIST23, MoNA, GNPS, Agilent PCDL sources). Train the network separately for Orbitrap and Q-TOF spectra using train_rescore.py with config files in ./config/, writing weights to ./check_point/fiddle_rescore_orbitrap.pt and ./check_point/fiddle_rescore_qtof.pt. At inference, score the top-5 candidate formulas per test spectrum, enforce the SENIOR rules via candidate generation, and rank candidates by predicted confidence. Note that FIDDLE v2.0.0 redesigned the rescore model as a Siamese architecture, so verify the intended version before reproducing the 416-208-104-26-13 MLP.

## Related tools

- **FIDDLE** (Provides the rescore network architecture, training script (train_rescore.py), configs, and MS/MS spectral features used to score candidate formulas) — https://github.com/JosieHong/FIDDLE
- **BUDDY (msbuddy)** (Optional source of candidate formulas from a reference library that can be rescored alongside FIDDLE candidates) — https://github.com/Philipbear/msbuddy
- **msfiddle** (PyPI package exposing pre-trained FIDDLE models for prediction and incorporating BUDDY and SIRIUS candidate outputs in file-based workflows) — https://github.com/josiehong/msfiddle

## Examples

```
python train_rescore.py --config ./config/rescore_orbitrap.yaml  # writes ./check_point/fiddle_rescore_orbitrap.pt
```

## Evaluation signals

- ROC AUC of the confidence scores for separating correct from incorrect top-5 candidates reaches ~0.97 on the evaluation set
- A successful training run writes ./check_point/fiddle_rescore_orbitrap.pt and ./check_point/fiddle_rescore_qtof.pt
- Ranked top-1 candidate matches the true formula for ~88.3% of the evaluation set and top-5 for ~93.6%
- Network architecture matches five fully connected layers with hidden dimensions 416-208-104-26-13 ending in a Sigmoid output
- Separate models are trained for Orbitrap and Q-TOF rather than a single pooled model

## Limitations

- The rescore model was redesigned in FIDDLE v2.0.0 as a Siamese architecture (breaking change), so the 416-208-104-26-13 MLP describes the original version
- Confidence score thresholds for FDR control are proposed but not implemented
- Performance degrades on chimeric (co-eluting) spectra as spectral mixing increases
- Accuracy varies across experimental conditions, especially those with less training data

## Evidence
