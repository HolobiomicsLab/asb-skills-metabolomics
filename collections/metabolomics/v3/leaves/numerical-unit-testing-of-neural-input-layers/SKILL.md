---
name: numerical-unit-testing-of-neural-input-layers
description: 'Use when reimplementing or validating the input featurization layer
  of a neural formula scorer: element counts must be converted to fixed-length sinusoidal
  embeddings before a Formula Transformer can consume them.'
when_to_use_negative:
- Input is already a fixed-length feature vector or a learned embedding — this skill
  only builds the deterministic sinusoidal count encoder
- Negative-ion-mode scoring is required — MIST-CF models were trained only for positive-mode
  data
- The goal is scoring a spectrum against a formula rather than featurizing the formula
  itself — use the full MIST-CF prediction pipeline instead
tools:
- name: MIST-CF
  role: The neural model whose sinusoidal formula embedding input layer this encoder
    feeds; implemented in the mist-cf repository
  repo: github.com/samgoldman97/mist-cf
- name: SIRIUS
  role: Supplies the candidate formulas (via SIRIUS decomp) whose element counts are
    encoded
  repo: https://github.com/boecker-lab/sirius
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1021/acs.jcim.3c01082
    title: 'MIST-CF: Chemical Formula Inference from Tandem Mass Spectra'
  source_content_hash: sha256:ec45a308563c1c37d918e0b98f0dbbae7104bc09208be92598136ba3348a61d8
schema_version: 0.2.0
metadata:
  license_tier: restricted
  tools:
  - MIST-CF
  - SIRIUS
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# Numerical Unit Testing of Neural Input Layers

> **License: restricted** — no clear open-source license detected for the underlying tool; verify licensing before commercial use or redistribution. <!-- asb-license-banner -->
## Summary

Implements and unit-checks the sinusoidal counts-based formula embedding encoder (Equation 5) used by MIST-CF, verifying by script that formula encodings Enc(F) have exactly 144 dimensions (8 dims × 18 elements) and contain only non-negative entries.

## When to use

Use when reimplementing or validating the input featurization layer of a neural formula scorer: element counts must be converted to fixed-length sinusoidal embeddings before a Formula Transformer can consume them.

## When NOT to use

- Input is already a fixed-length feature vector or a learned embedding — this skill only builds the deterministic sinusoidal count encoder
- Negative-ion-mode scoring is required — MIST-CF models were trained only for positive-mode data
- The goal is scoring a spectrum against a formula rather than featurizing the formula itself — use the full MIST-CF prediction pipeline instead

## Inputs

- Chemical formula candidate string (e.g., precursor formula candidate from SIRIUS decomp)
- Per-element integer count vector over the 18 supported elements

## Outputs

- 144-dimensional formula encoding Enc(F) ∈ R^144+ (8 sinusoidal dims per element × 18 elements, all entries >= 0)
- encoder_verification_report.txt with dimensionality and non-negativity check results

## How to apply

Parse a chemical formula candidate string (e.g., a precursor formula candidate produced by SIRIUS decomp) into per-element integer counts covering the 18 supported elements. For each element count v, compute 8 embedding dimensions using sinusoidal features at periods 1, 2, 4, ..., 256 (powers of two up to 256, i.e., log2(256) = 8 dims), then apply an absolute-value (non-negative) transformation so every component is >= 0. Concatenate the 8-dim embeddings across all 18 elements to assemble the 144-dimensional vector Enc(F). Validate with a small Python script using NumPy: assert the output shape is (144,) and the minimum entry is >= 0, including edge cases such as count 0 and large counts. This embedding scheme is the sinusoidal formula embedding introduced in the SCARF work and adopted by MIST-CF ahead of its Formula Transformer scoring the agreement between a candidate formula and an unknown MS/MS spectrum.

## Related tools

- **MIST-CF** (The neural model whose sinusoidal formula embedding input layer this encoder feeds; implemented in the mist-cf repository) — github.com/samgoldman97/mist-cf
- **SIRIUS** (Supplies the candidate formulas (via SIRIUS decomp) whose element counts are encoded) — https://github.com/boecker-lab/sirius

## Examples

```
import numpy as np; enc = sinusoidal_formula_encoding('C6H12O6'); assert enc.shape == (144,) and enc.min() >= 0
```

## Evaluation signals

- Output vector Enc(F) for any test formula has shape exactly (144,)
- Minimum entry of Enc(F) is >= 0 (non-negativity after the absolute-value transformation) for all test formulas
- Edge cases pass: count 0 and large counts produce valid, bounded sinusoidal features within the period-1 to period-256 range
- Number of encoded elements equals 18 and each contributes exactly 8 dimensions (log2(256))
- Script exits successfully and writes dimensionality and non-negativity results to encoder_verification_report.txt

## Limitations

- Validation is qualitative only — the paper reports no separate quantitative metric for this encoder; correctness is judged by shape and non-negativity invariants
- Only the 18 considered elements are supported; formulas with elements outside this set will fail parsing
- Encoder verified in isolation; the sinusoidal scheme originates from SCARF, and downstream MIST-CF performance (10% top-1 accuracy gain) is not evidence of encoder correctness on its own

## Evidence
