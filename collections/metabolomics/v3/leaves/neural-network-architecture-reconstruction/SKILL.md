---
name: neural-network-architecture-reconstruction
description: Use when a paper describes a network's architecture concretely (layer
  count, hidden dimensions, output activation) and the goal is to reimplement, train,
  and validate it from the described spec and released codebase.
when_to_use_negative:
- Input is raw centroided spectra needing the main formula prediction model — this
  skill only reconstructs the auxiliary rescore/ranking network, not the dilated-convolution
  formula identifier.
- Working with FIDDLE v2.0.0, where the rescore model was redesigned as a Siamese
  architecture and the five-layer 416-208-104-26-13 spec no longer applies.
- Data lacking instrument labels or candidate formula lists, since the model is trained
  separately for Orbitrap vs. Q-TOF and requires per-candidate feature vectors.
edam_operation: http://edamontology.org/operation_3937
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_3520
tools:
- name: FIDDLE
  role: Hosts the rescore model implementation, train_rescore.py script, config files,
    and pre-trained checkpoints to reproduce the five-layer confidence-scoring network
  repo: https://github.com/JosieHong/FIDDLE
- name: BUDDY (msbuddy)
  role: Optional dependency for candidate formula generation feeding the rescore ranking
    step
  repo: https://github.com/Philipbear/msbuddy
- name: SIRIUS
  role: Optional external baseline whose candidate outputs can be incorporated into
    file-based FIDDLE workflows
  repo: https://github.com/sirius-ms/sirius
- name: msfiddle
  role: PyPI package providing pre-trained FIDDLE checkpoints and CLI/Python API for
    running predictions with the rescore ranking
  repo: https://github.com/josiehong/msfiddle
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-025-66060-9
    title: 'FIDDLE: a deep learning method for chemical formulas prediction from tandem
      mass spectra'
  source_content_hash: sha256:ba3c83580356b15b7f8242126bc31e388bc24c8320e38c930e489629584e570b
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
  - SIRIUS
  - msfiddle
  edam_operation: http://edamontology.org/operation_3937
  edam_topics:
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# neural-network-architecture-reconstruction

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Reconstruct a published deep learning module from its paper-specified architecture and training protocol — here, FIDDLE's rescore network that estimates confidence scores for candidate formulas. It converts textual architecture descriptions (layer stack, hidden dimensions, activation, output) into working PyTorch code plus a reproducible training/scoring pipeline.

## When to use

Use when a paper describes a network's architecture concretely (layer count, hidden dimensions, output activation) and the goal is to reimplement, train, and validate it from the described spec and released codebase.

## When NOT to use

- Input is raw centroided spectra needing the main formula prediction model — this skill only reconstructs the auxiliary rescore/ranking network, not the dilated-convolution formula identifier.
- Working with FIDDLE v2.0.0, where the rescore model was redesigned as a Siamese architecture and the five-layer 416-208-104-26-13 spec no longer applies.
- Data lacking instrument labels or candidate formula lists, since the model is trained separately for Orbitrap vs. Q-TOF and requires per-candidate feature vectors.

## Inputs

- Training MS/MS spectra with labeled correct/incorrect candidate formulas (compiled from NIST20, NIST23, MoNA, GNPS, Agilent PCDL)
- Condition-independent MS/MS spectral feature vectors
- Candidate formula vectors (top-5 candidates per spectrum, SENIOR-rule-compliant)
- Instrument-specific config files in ./config/
- Train script train_rescore.py

## Outputs

- Trained rescore weights ./check_point/fiddle_rescore_orbitrap.pt
- Trained rescore weights ./check_point/fiddle_rescore_qtof.pt
- Confidence scores and rankings for top-5 candidate formulas per test spectrum
- ROC AUC evaluation of score discrimination between correct and incorrect candidates

## How to apply

Transcribe the published architecture exactly: for FIDDLE's rescore model this is a five-layer fully connected network with hidden dimensions 416-208-104-26-13 and a Sigmoid output layer in PyTorch. Build training pairs by concatenating condition-independent MS/MS spectral features with each candidate formula vector derived from training spectra, and train separate models for Orbitrap and Q-TOF instruments using the provided train script (train_rescore.py) with config files in ./config/. Write weights to ./check_point/fiddle_rescore_orbitrap.pt and ./check_point/fiddle_rescore_qtof.pt. At inference, score the top-5 candidate formulas per test spectrum, rank them by predicted confidence, and evaluate discrimination with ROC AUC. Note the repo caveat that v2.0.0 redesigned the rescore model as a Siamese architecture, so pin the v1.0.0 release weights when reproducing the paper's five-layer design.

## Related tools

- **FIDDLE** (Hosts the rescore model implementation, train_rescore.py script, config files, and pre-trained checkpoints to reproduce the five-layer confidence-scoring network) — https://github.com/JosieHong/FIDDLE
- **BUDDY (msbuddy)** (Optional dependency for candidate formula generation feeding the rescore ranking step) — https://github.com/Philipbear/msbuddy
- **SIRIUS** (Optional external baseline whose candidate outputs can be incorporated into file-based FIDDLE workflows) — https://github.com/sirius-ms/sirius
- **msfiddle** (PyPI package providing pre-trained FIDDLE checkpoints and CLI/Python API for running predictions with the rescore ranking) — https://github.com/josiehong/msfiddle

## Examples

```
python train_rescore.py  # writes ./check_point/fiddle_rescore_orbitrap.pt and ./check_point/fiddle_rescore_qtof.pt
```

## Evaluation signals

- ROC AUC of the predicted confidence scores for separating correct from incorrect candidates reaches ~0.97 on the evaluation set.
- Successful training run writes both weight files (fiddle_rescore_orbitrap.pt and fiddle_rescore_qtof.pt) into ./check_point/.
- Architecture check: the reconstructed model has exactly five fully connected layers with hidden dimensions 416-208-104-26-13 and a Sigmoid output.
- Separate models train per instrument type (Orbitrap and Q-TOF) rather than a single pooled model.
- Ranked top-1 candidates from the rescored predictions reproduce the reported 88.3% top-1 / 93.6% top-5 accuracy regime when plugged into the full pipeline.

## Limitations

- FIDDLE v2.0.0 redesigned the rescore model into a Siamese architecture, so the five-layer reconstruction only matches the v1.0.0 release.
- Confidence-score thresholds for FDR control are proposed in the paper but not implemented, so ranking cannot yet be calibrated to a target FDR.
- Performance varies across experimental conditions with less training data, and degrades on chimeric (co-eluting) spectra as spectral mixing increases.

## Evidence

- [methods] five-layer rescore architecture: [evidence span withheld — anchor status: not_found]
- [results] validation AUC: [evidence span withheld — anchor status: not_found]
