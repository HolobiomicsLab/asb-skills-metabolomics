---
name: spectral-feature-vector-concatenation
description: Use when ranking top-k candidate molecular formulas per MS/MS spectrum
  (Orbitrap or Q-TOF) by predicted confidence, after FIDDLE's refinement step has
  generated candidates.
when_to_use_negative:
- You only need the FIDDLE formula prediction itself, not re-ranking of the refinement
  step's candidates
- You are using FIDDLE v2.0.0+, where the rescore model has been redesigned as a Siamese
  architecture and the five-layer concatenation design no longer matches the released
  code
- Spectra are chimeric (co-eluting), where FIDDLE's performance decreases as spectral
  mixing increases
tools:
- name: FIDDLE
  role: Source of the rescore network code (train_rescore.py, ./config/), spectral
    feature extractor, and candidate formula generator whose outputs are concatenated
    and scored
  repo: github.com/JosieHong/FIDDLE
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-025-66060-9
    title: 'FIDDLE: a deep learning method for chemical formulas prediction from tandem
      mass spectra'
  source_content_hash: sha256:dbd86367469b0d7e70f05f89477054f64a1080b4849fa3b7edd33ebeba311096
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/JosieHong/FIDDLE
  tools:
  - FIDDLE
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# Spectral feature vector concatenation for formula confidence rescoring

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

FIDDLE's secondary rescoring approach: build training pairs by concatenating condition-independent MS/MS spectral features with each candidate formula vector, then train a five-layer fully connected rescore network to predict confidence scores used to rank candidate formulas.

## When to use

Use when ranking top-k candidate molecular formulas per MS/MS spectrum (Orbitrap or Q-TOF) by predicted confidence, after FIDDLE's refinement step has generated candidates.

## When NOT to use

- You only need the FIDDLE formula prediction itself, not re-ranking of the refinement step's candidates
- You are using FIDDLE v2.0.0+, where the rescore model has been redesigned as a Siamese architecture and the five-layer concatenation design no longer matches the released code
- Spectra are chimeric (co-eluting), where FIDDLE's performance decreases as spectral mixing increases

## Inputs

- Condition-independent MS/MS spectral features extracted by the FIDDLE formula identification model
- Top-5 candidate formula vectors from the breadth-first refinement step per test spectrum
- Training spectra (NIST20, NIST23, MoNA, GNPS, Agilent PCDL) with correct/incorrect candidate labels

## Outputs

- Trained rescore weight files ./check_point/fiddle_rescore_orbitrap.pt and ./check_point/fiddle_rescore_qtof.pt
- Predicted confidence score per candidate formula
- Ranked top-5 candidate formula list per spectrum

## How to apply

Implement the rescore network in PyTorch as a five-layer fully connected architecture with hidden dimensions 416-208-104-26-13 and a Sigmoid output layer. Build training pairs by concatenating condition-independent MS/MS spectral features with each candidate formula vector derived from the training spectra. Train the network separately for Orbitrap and Q-TOF spectra using train_rescore.py with the config files in ./config/, writing weights to ./check_point/fiddle_rescore_orbitrap.pt and ./check_point/fiddle_rescore_qtof.pt. At inference, score the top-5 candidate formulas per test spectrum and rank them by predicted confidence; decision point: keep instrument-specific models separate rather than pooling, since Q-TOF data was augmented with Gaussian noise (mean 0, sd 0.1) while Orbitrap training was not.

## Related tools

- **FIDDLE** (Source of the rescore network code (train_rescore.py, ./config/), spectral feature extractor, and candidate formula generator whose outputs are concatenated and scored) — github.com/JosieHong/FIDDLE

## Examples

```
python train_rescore.py --config ./config/rescore_qtof.yaml  # writes ./check_point/fiddle_rescore_qtof.pt
```

## Evaluation signals

- Reconstructed rescore model reproduces the reported ROC AUC of ~0.97 for separating correct from incorrect top-5 candidates on the evaluation set
- Run writes rescore weight files named fiddle_rescore_orbitrap.pt and fiddle_rescore_qtof.pt into ./check_point/
- Rescore output is a single probability in [0, 1] per candidate (Sigmoid output layer)
- Ranking by predicted confidence improves top-1 accuracy toward the reported 88.3% top-1 / 93.6% top-5 evaluation-set accuracy

## Limitations

- FDR control via confidence score thresholds is proposed but not implemented in the current workflow
- As of v2.0.0 the rescore model was redesigned (Siamese architecture), so the five-layer concatenation model corresponds to the v1.0.0 release weights
- Accuracy varies across experimental conditions, especially those with less training data

## Evidence
