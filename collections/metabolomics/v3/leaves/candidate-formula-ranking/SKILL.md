---
name: candidate-formula-ranking
description: Use when a formula-identification pipeline produces multiple candidate
  formulas per MS/MS spectrum (e.g. after SENIOR-rule refinement against precursor
  mass) and you need to order them by predicted correctness using learned MS/MS spectral
  features.
when_to_use_negative:
- The input is a single known formula or already a ranked candidate list with calibrated
  scores — retraining a rescore model is unnecessary
- The MS/MS spectra fall outside the retained scope (fewer than five peaks, peaks
  below 50 m/z, m/z above 1500, or molecules with atoms beyond C, H, O, N, F, S, Cl,
  P, B, I, Br, Na, K or more than 300 atoms)
- You need a formula-free predictor — the rescore step only ranks candidates already
  generated within the precursor mass tolerance by the refinement step
tools:
- name: FIDDLE
  role: Provides the rescore network (five-layer fully connected, Sigmoid output)
    that scores and ranks candidate formulas; pre-trained weights fiddle_rescore_orbitrap.pt
    and fiddle_rescore_qtof.pt can be downloaded from the release page
  repo: https://github.com/JosieHong/FIDDLE
- name: msfiddle
  role: PyPI CLI/Python API exposing pre-trained FIDDLE models (including rescore
    checkpoints downloaded via msfiddle-download-models) for ranking candidates in
    prediction workflows
  repo: https://github.com/josiehong/msfiddle
provenance:
  source_task_ids:
  - task_007
  - task_003
  source_papers:
  - doi: 10.1038/s41467-025-66060-9
    title: 'FIDDLE: a deep learning method for chemical formulas prediction from tandem
      mass spectra'
  source_content_hash: sha256:a3e84fce4ab2bc28e82d729dce0eba73b909ddad6b59ce836177c98a2c3fedeb
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
  - msfiddle
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 15
---

# candidate-formula-ranking

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

FIDDLE's secondary deep learning rescore model estimates confidence scores for the candidate formulas produced by breadth-first refinement and ranks them so the top-5 list reflects the most likely molecular formula for a tandem mass spectrum.

## When to use

Use when a formula-identification pipeline produces multiple candidate formulas per MS/MS spectrum (e.g. after SENIOR-rule refinement against precursor mass) and you need to order them by predicted correctness using learned MS/MS spectral features.

## When NOT to use

- The input is a single known formula or already a ranked candidate list with calibrated scores — retraining a rescore model is unnecessary
- The MS/MS spectra fall outside the retained scope (fewer than five peaks, peaks below 50 m/z, m/z above 1500, or molecules with atoms beyond C, H, O, N, F, S, Cl, P, B, I, Br, Na, K or more than 300 atoms)
- You need a formula-free predictor — the rescore step only ranks candidates already generated within the precursor mass tolerance by the refinement step

## Inputs

- Condition-independent MS/MS spectral feature vectors from the FIDDLE formula-identification model (Q-TOF or Orbitrap)
- Candidate formula vectors from the breadth-first refinement step (SENIOR-rule compliant, within precursor mass tolerance)
- Training spectra with known correct formulas (e.g. from NIST20/NIST23, MoNA, GNPS, Agilent PCDL compilations)
- Instrument-specific config yml files in ./config/

## Outputs

- Trained rescore weight files ./check_point/fiddle_rescore_orbitrap.pt and ./check_point/fiddle_rescore_qtof.pt
- Confidence scores (Sigmoid outputs) for the top-5 candidate formulas per test spectrum
- Ranked top-5 candidate formula lists and ROC AUC metrics for correct vs. incorrect candidate separation

## How to apply

Implement the rescore network as a five-layer fully connected architecture with hidden dimensions 416-208-104-26-13 and a Sigmoid output layer in PyTorch. Build training pairs by concatenating condition-independent MS/MS spectral features learned during formula identification with each candidate formula vector derived from the training spectra. Train the network separately for Orbitrap and Q-TOF spectra (train_rescore.py with the config yml in ./config/), writing weights as ./check_point/fiddle_rescore_orbitrap.pt and ./check_point/fiddle_rescore_qtof.pt. At inference, score the top-5 refined candidate formulas per test spectrum, apply precursor mass tolerances of 10 ppm (Q-TOF) or 5 ppm (Orbitrap), and rank candidates by predicted confidence score; evaluate discrimination between correct and incorrect candidates via ROC AUC, expecting about 0.97 on the evaluation set. Note that as of FIDDLE v2.0.0 the rescore model was redesigned to a Siamese architecture, so verify which version's architecture you are reconstructing.

## Related tools

- **FIDDLE** (Provides the rescore network (five-layer fully connected, Sigmoid output) that scores and ranks candidate formulas; pre-trained weights fiddle_rescore_orbitrap.pt and fiddle_rescore_qtof.pt can be downloaded from the release page) — https://github.com/JosieHong/FIDDLE
- **msfiddle** (PyPI CLI/Python API exposing pre-trained FIDDLE models (including rescore checkpoints downloaded via msfiddle-download-models) for ranking candidates in prediction workflows) — https://github.com/josiehong/msfiddle

## Examples

```
python train_rescore.py --config ./config/  # writes ./check_point/fiddle_rescore_orbitrap.pt and ./check_point/fiddle_rescore_qtof.pt
```

## Evaluation signals

- ROC AUC of the predicted confidence scores for separating correct from incorrect top-5 candidates reaches approximately 0.97 on the evaluation set
- The training run writes both weight files (fiddle_rescore_orbitrap.pt and fiddle_rescore_qtof.pt) into ./check_point/
- Top-5 ranked candidates satisfy the SENIOR rules and lie within the instrument-specific precursor mass tolerance (10 ppm Q-TOF, 5 ppm Orbitrap)
- End-to-end top-5 accuracy matches reported values (e.g. 93.6% on the evaluation set; 95.0% Q-TOF and 93.3% Orbitrap at the highest noise level)

## Limitations

- Breaking change in FIDDLE v2.0.0: the rescore model was redesigned to a Siamese architecture, so reproducing the original five-layer 416-208-104-26-13 network requires the v1.x codebase and v1.0.0 release weights
- FDR control via confidence score thresholds is proposed but not implemented in the current workflow
- Chimeric (co-eluting) spectra degrade FIDDLE performance as spectral mixing increases, which also affects rescore reliability

## Evidence

- [intro] secondary confidence-scoring model: "we train a secondary deep learning model to estimate confidence scores for candidate formulas and rank them based on MS/MS features"
- [discussion] ranking refines formulas via SENIOR rules: "ranking them by predicted confidence scores, aim to refine the predicted formulas using the SENIOR rules"
- [readme] rescore architecture redesigned in v2: "The rescore model has been redesigned (Siamese architecture), see details in [CHANGELOG.md](./CHANGELOG.md)."
- [methods] pre-trained weights from release page: "download the weights from the [release page](https://github.com/JosieHong/FIDDLE/releases/tag/v1.0.0)"
