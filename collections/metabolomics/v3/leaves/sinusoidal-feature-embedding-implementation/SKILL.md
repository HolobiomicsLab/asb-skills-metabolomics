---
name: sinusoidal-feature-embedding-implementation
description: Use when featurizing integer element counts of a chemical formula candidate
  into a fixed-length, non-negative vector for a neural scorer — e.g., before the
  Formula Transformer in MIST-CF or when replicating SCARF-style sinusoidal formula
  embeddings.
when_to_use_negative:
- Input is already a learned or precomputed feature vector rather than raw element
  counts — sinusoidal encoding would discard existing structure
- Formulas contain elements outside the 18-element set supported by MIST-CF's counts-based
  encoder
- The target model expects a different encoder (e.g., learned embeddings); MIST-CF
  models were trained only for positive-mode data, so this encoder also presumes positive-mode
  formula candidates
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3473
- http://edamontology.org/topic_3272
tools:
- name: MIST-CF
  role: Model whose counts-based formula encoder (Equation 5) this skill reimplements;
    the 144-dim Enc(F) output is passed into its Formula Transformer
  repo: https://github.com/samgoldman97/mist-cf
- name: SIRIUS
  role: Supplies the formula candidate set whose element counts are encoded; SIRIUS
    decomp decomposes exact masses into formula candidates upstream
  repo: https://github.com/boecker-lab/sirius
- name: Chemical Formula Transformer
  role: Downstream architecture that consumes Enc(F); MIST-CF extends this architecture
    with sinusoidal formula embeddings from SCARF
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1021/acs.jcim.3c01082
    title: 'MIST-CF: Chemical Formula Inference from Tandem Mass Spectra'
  source_content_hash: sha256:d9e3ea949ab52834261c7adeb64729e08e09fb6c9101e6869456e6c847237f4e
schema_version: 0.2.0
metadata:
  license_tier: restricted
  repo_url: https://github.com/samgoldman97/mist-cf
  tools:
  - Chemical Formula Transformer
  - MIST-CF
  - SIRIUS
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3473
  - http://edamontology.org/topic_3272
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9285714285714286
n_claims: 14
---

# sinusoidal-feature-embedding-implementation

> **License: restricted** — no clear open-source license detected for the underlying tool; verify licensing before commercial use or redistribution. <!-- asb-license-banner -->
## Summary

Implements MIST-CF's counts-based formula encoder (Equation 5): integer element counts are converted via sinusoidal features at powers-of-two periods (up to 256) into 8-dimensional non-negative embeddings per element, concatenated across 18 elements into a 144-dimensional vector Enc(F) fed to the Formula Transformer.

## When to use

Use when featurizing integer element counts of a chemical formula candidate into a fixed-length, non-negative vector for a neural scorer — e.g., before the Formula Transformer in MIST-CF or when replicating SCARF-style sinusoidal formula embeddings.

## When NOT to use

- Input is already a learned or precomputed feature vector rather than raw element counts — sinusoidal encoding would discard existing structure
- Formulas contain elements outside the 18-element set supported by MIST-CF's counts-based encoder
- The target model expects a different encoder (e.g., learned embeddings); MIST-CF models were trained only for positive-mode data, so this encoder also presumes positive-mode formula candidates

## Inputs

- Chemical formula candidate strings (e.g., precursor formula candidates decomposed by SIRIUS from exact masses)
- Per-element integer counts over the 18 supported elements (parsed from formula strings)

## Outputs

- 144-dimensional non-negative formula encoding Enc(F) ∈ R^144+ (18 elements × 8 dims)
- encoder_verification_report.txt with dimensionality and non-negativity check results

## How to apply

Parse each input formula candidate string into per-element integer counts using a formula parser covering the 18 supported elements. For each count v, compute an 8-dimensional embedding (8 = log2(256)) using sinusoidal features at periods 1, 2, 4, ..., 256, then apply an absolute-value (non-negative) transformation so every component is >= 0 — this abs trick lets raw count magnitudes be encoded without signed features. Concatenate the 18 per-element 8-dim embeddings to assemble the 144-dimensional vector Enc(F). Validate with a small NumPy script asserting output shape (144,) and min entry >= 0 for test formulas, including edge cases of count 0 and large counts, and write the dimensionality/non-negativity check results to encoder_verification_report.txt. Acceptance is qualitative — exactly 144 dimensions and only non-negative entries; the paper reports no separate quantitative metric for this encoder alone.

## Related tools

- **MIST-CF** (Model whose counts-based formula encoder (Equation 5) this skill reimplements; the 144-dim Enc(F) output is passed into its Formula Transformer) — https://github.com/samgoldman97/mist-cf
- **SIRIUS** (Supplies the formula candidate set whose element counts are encoded; SIRIUS decomp decomposes exact masses into formula candidates upstream) — https://github.com/boecker-lab/sirius
- **Chemical Formula Transformer** (Downstream architecture that consumes Enc(F); MIST-CF extends this architecture with sinusoidal formula embeddings from SCARF)

## Examples

```
import numpy as np; P=[1,2,4,8,16,32,64,128,256]; enc=np.abs(np.concatenate([np.concatenate([np.sin(2*np.pi*max(v,0.5)/p) for p in P[:8]]) for v in counts])); assert enc.shape==(144,) and enc.min()>=0
```

## Evaluation signals

- Output vector Enc(F) has exactly 144 dimensions (18 elements × 8 dims per element)
- Minimum entry of Enc(F) is >= 0 for every test formula (absolute-value transformation applied)
- Edge cases handled: count 0 and large counts produce valid non-negative 8-dim embeddings
- Verification results recorded in encoder_verification_report.txt confirm shape (144,) and non-negativity
- Downstream consistency: Enc(F) concatenated from 8-dim embeddings matches the Formula Transformer's expected input dimensionality

## Limitations

- Acceptance is qualitative only — the paper reports no separate quantitative evaluation metric for this encoder
- MIST-CF models (and hence this encoder in context) were trained only for positive-mode data; negative-mode support is absent
- Encoder dimensionality depends on exactly 18 considered elements and 8 = log2(256) dimensions; changing the element set or max period changes Enc(F)'s shape
- MIST-CF still relies on SIRIUS's algorithmic decomposition of exact masses into formula candidates upstream of this encoding

## Evidence
