---
name: mass-spectrometry-formula-representation
description: Use when a neural model over formula candidates (e.g., the MIST-CF Formula
  Transformer or FastFilter) needs a fixed-dimensional, permutation-ordered numeric
  encoding of integer element counts rather than learned embeddings.
when_to_use_negative:
- The formula contains elements outside MIST-CF's 18-element vocabulary — counts for
  unsupported elements cannot be encoded.
- The representation needed is a learned/embedded formula representation (e.g., fragment
  or subformula features from the Formula Transformer itself) rather than deterministic
  integer-count sinusoidal features.
- A database-driven formula representation (e.g., database formula fingerprints from
  NIST/PubChem lookups) is required instead of the de novo, database-independent encoding.
tools:
- name: MIST-CF
  role: Provides the sinusoidal formula embedding encoder (Equation 5) whose output
    feeds its Chemical Formula Transformer; released as open source under the MIT
    license
  repo: github.com/samgoldman97/mist-cf
- name: SIRIUS
  role: Supplies the formula candidate strings whose element counts are encoded, via
    its dynamic-programming exact-mass decomposition (SIRIUS decomp)
  repo: https://github.com/boecker-lab/sirius
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1021/acs.jcim.3c01082
    title: 'MIST-CF: Chemical Formula Inference from Tandem Mass Spectra'
  source_content_hash: sha256:48950064bbc03abfae69392c9836b3ed5a09c5a4e714786bf368255686e91bf1
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
n_claims: 7
---

# mass-spectrometry-formula-representation

> **License: restricted** — no clear open-source license detected for the underlying tool; verify licensing before commercial use or redistribution. <!-- asb-license-banner -->
## Summary

Featurizes the integer element counts of a chemical formula candidate into a fixed-size, non-negative vector (Enc(F) ∈ R^144) using sinusoidal count embeddings, as defined by MIST-CF's Equation 5, for input to its Chemical Formula Transformer.

## When to use

Use when a neural model over formula candidates (e.g., the MIST-CF Formula Transformer or FastFilter) needs a fixed-dimensional, permutation-ordered numeric encoding of integer element counts rather than learned embeddings.

## When NOT to use

- The formula contains elements outside MIST-CF's 18-element vocabulary — counts for unsupported elements cannot be encoded.
- The representation needed is a learned/embedded formula representation (e.g., fragment or subformula features from the Formula Transformer itself) rather than deterministic integer-count sinusoidal features.
- A database-driven formula representation (e.g., database formula fingerprints from NIST/PubChem lookups) is required instead of the de novo, database-independent encoding.

## Inputs

- Chemical formula candidate string (e.g., precursor formula candidates decomposed by SIRIUS from an exact MS1 mass)
- Per-element integer count vector over the 18 supported elements
- Element vocabulary/order definition (18 elements) shared between training and inference

## Outputs

- 144-dimensional non-negative formula encoding Enc(F) ∈ R^144 (18 elements × 8 sinusoidal dimensions)
- encoder_verification_report.txt recording dimensionality and non-negativity check results

## How to apply

Parse the formula candidate string into per-element integer counts restricted to the 18 elements supported by MIST-CF. For each element count v, compute 8 embedding dimensions (log2(256) = 8) using sinusoidal features at periods 1, 2, 4, ..., 256 (powers of two up to 256). Apply an absolute-value transformation so every embedding component is non-negative, then concatenate the 8-dim blocks across all 18 elements in a fixed element order to yield Enc(F) ∈ R^144. The non-negative sinusoidal scheme is adopted from the SCARF work per the repository README, which lists 'Utilizing sinusoidal formula embeddings' as one of the advances in the MIST-CF chemical formula transformer architecture. Verify the encoder by asserting the output shape is exactly (144,) and that the minimum entry is >= 0 for test formulas including edge cases (count 0, large counts), then feed Enc(F) into the Formula Transformer that scores formula candidates within the energy-based MIST-CF framework.

## Related tools

- **MIST-CF** (Provides the sinusoidal formula embedding encoder (Equation 5) whose output feeds its Chemical Formula Transformer; released as open source under the MIT license) — github.com/samgoldman97/mist-cf
- **SIRIUS** (Supplies the formula candidate strings whose element counts are encoded, via its dynamic-programming exact-mass decomposition (SIRIUS decomp)) — https://github.com/boecker-lab/sirius

## Examples

```
import numpy as np; enc = formula_encoding('C6H12O6'); assert enc.shape == (144,) and enc.min() >= 0
```

## Evaluation signals

- Output shape is exactly (144,) for every test formula (18 elements × 8 sinusoidal dimensions).
- Minimum entry of Enc(F) is >= 0 after the absolute-value transformation, for all test formulas including count 0 and large counts.
- Each 8-dim per-element block reproduces sinusoidal features at periods 1, 2, 4, ..., 256 for the given integer count.
- Same formula string always yields a byte-identical Enc(F) (deterministic featurization, no learned parameters).
- Element ordering is preserved after flattening/concatenation, so identical formulas across candidates align positionally in the Formula Transformer input.

## Limitations

- Only 18 elements are considered; formulas with other elements cannot be represented by this encoder.
- MIST-CF models and their formula encodings cover positive-mode data only; negative-mode support is absent.
- Validation of this encoder in the paper is qualitative (dimensionality and non-negativity checks); no separate quantitative metric is reported for the encoder in isolation.

## Evidence
