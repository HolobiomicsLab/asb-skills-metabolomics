---
name: feature-vector-dimension-verification
description: Use when reconstructing or re-implementing GLEAMS' spectrum encoder and
  you must confirm the concatenated precursor, fragment-bin, and reference-similarity
  blocks yield the expected 3010 features before feeding the neural network.
when_to_use_negative:
- You are verifying the neural network output, which is a 32-dimensional embedding,
  not the 3010-dimensional input encoding
- Spectra lack reliable precursor charge or precursor m/z, so the Gray-code and one-hot
  blocks cannot be populated
- You only need `gleams embed` end-to-end embeddings and will not re-implement the
  encoder yourself
edam_operation: http://edamontology.org/operation_2930
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3474
tools:
- name: GLEAMS
  role: Provides the spectrum encoder whose 3010-feature input vector is being verified;
    feature-block settings are configurable in gleams/config.py
  repo: https://github.com/bittremieux/GLEAMS
- name: Python
  role: Used to implement the Gray-code, one-hot, fragment-binning, and dot-product
    similarity encodings and to assert the 3010-feature dimension
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41592-022-01496-1
    title: A learned embedding for efficient joint analysis of millions of mass spectra
  source_content_hash: sha256:01ba62a366846fef84bfed4cffce085a4a717cbc303b9c0b66e9141f4c3b2b63
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bittremieux/GLEAMS
  tools:
  - GLEAMS
  - Python
  edam_operation: http://edamontology.org/operation_2930
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 13
---

# feature-vector-dimension-verification

## Summary

Verifies that GLEAMS' per-spectrum input encoding produces exactly a 3010-dimensional feature vector by summing its fixed-width blocks: two 27-bit Gray codes (precursor mass and m/z), a 7-state one-hot charge, 2449 square-root-transformed fragment-intensity bins (50.5–2500 m/z), and 500 dot-product similarities to reference spectra.

## When to use

Use when reconstructing or re-implementing GLEAMS' spectrum encoder and you must confirm the concatenated precursor, fragment-bin, and reference-similarity blocks yield the expected 3010 features before feeding the neural network.

## When NOT to use

- You are verifying the neural network output, which is a 32-dimensional embedding, not the 3010-dimensional input encoding
- Spectra lack reliable precursor charge or precursor m/z, so the Gray-code and one-hot blocks cannot be populated
- You only need `gleams embed` end-to-end embeddings and will not re-implement the encoder yourself

## Inputs

- Single MS/MS spectrum: precursor mass, precursor m/z, precursor charge, and peak list from a peak file (e.g. mzML)

## Outputs

- 3010-dimensional input feature vector for the GLEAMS neural network

## How to apply

Encode the precursor mass and precursor m/z each as a 27-bit Gray code (2 × 27 = 54 features) and the precursor charge as a one-hot vector over 7 states. Bin fragment ions over the 50.5–2500 m/z range into 2449 bins and apply a square-root transform to the binned intensities. Compute normalized dot-product similarities between the spectrum and an invariant set of 500 reference spectra. Concatenate all blocks (54 + 7 + 2449 + 500) and assert the resulting vector length is exactly 3010; any mismatch indicates a misconfigured bin count, Gray-code width, or reference-spectrum count in gleams/config.py.

## Related tools

- **GLEAMS** (Provides the spectrum encoder whose 3010-feature input vector is being verified; feature-block settings are configurable in gleams/config.py) — https://github.com/bittremieux/GLEAMS
- **Python** (Used to implement the Gray-code, one-hot, fragment-binning, and dot-product similarity encodings and to assert the 3010-feature dimension)

## Examples

```
import numpy as np; assert gray_mass.shape == (27,) and gray_mz.shape == (27,) and charge_oh.shape == (7,) and frag_bins.shape == (2449,) and ref_sim.shape == (500,) and features.shape == (3010,)
```

## Evaluation signals

- len(feature_vector) == 3010 for every encoded spectrum
- Gray-code blocks contribute exactly 27 bits each for precursor mass and precursor m/z (54 total)
- Charge one-hot block sums to 1 and has exactly 7 positions
- Fragment-intensity block has exactly 2449 bins spanning 50.5–2500 m/z and is square-root transformed
- Reference-similarity block contains exactly 500 normalized dot-product values

## Limitations

- The 3010-feature encoding is specific to the published GLEAMS configuration; other fragment-bin widths, m/z ranges, or reference-spectrum counts change the dimension
- Feature-block settings live in gleams/config.py, so silently modified configuration will break the expected dimensionality
- Encoding correctness is only a precondition — the learned model maps these 3010 features to a 32-dimensional embedding space

## Evidence

- [supplementary] GLEAMS encodes each spectrum as a 3010-dimensional feature vector combining precursor attributes, binned fragment intensities, and dot-product similarities with reference spectra: [evidence span withheld — anchor status: not_found]
- [supplementary] precursor mass and m/z use 27-bit Gray codes, charge is one-hot encoded over 7 states, fragments are square-root transformed and binned into 2449 bins (50.5-2500 m/z): [evidence span withheld — anchor status: not_found]
- [readme] GLEAMS encodes mass spectra as vectors of features and feeds them to a neural network to embed them into a 32-dimensional space: "GLEAMS encodes mass spectra as vectors of features and feeds them to a neural network to embed them into a 32-dimensional space in which spectra generated by the same peptide are close together"
- [readme] Full configuration of GLEAMS, including various configurations to train the neural network, can be modified in the gleams/config.py file: "Full configuration of GLEAMS, including various configurations to train the neural network, can be modified in the `gleams/config.py` file"
