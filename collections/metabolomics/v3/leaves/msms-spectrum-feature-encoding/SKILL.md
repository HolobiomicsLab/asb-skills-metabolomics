---
name: msms-spectrum-feature-encoding
description: Use when reimplementing, validating, or extending GLEAMS's spectrum encoder,
  or when converting raw MS/MS spectra (precursor mass, charge, peak lists) into the
  exact 3010-feature input format its neural network expects.
when_to_use_negative:
- The goal is a final embedding, not the raw input features — use the `gleams embed`
  command instead, which directly outputs 32-dimensional embeddings.
- The encoding scheme being validated is not GLEAMS's (e.g., a different spectral
  embedding pipeline with different bin ranges or reference sets).
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: GLEAMS
  role: Source implementation of the spectrum encoder whose config (gleams/config.py)
    defines the feature blocks and reference-spectrum set; can be used to cross-check
    a reconstructed encoder.
  repo: https://github.com/bittremieux/GLEAMS
- name: Python 3.8
  role: Runtime for reimplementing the encoding steps (Gray coding, binning, square-root
    transform, dot-product similarities).
- name: ANN-SoLo
  role: Source of the top 500 precursor mass differences from the open modification
    search of GLEAMS cluster centroids used as the reference-spectrum similarity features.
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41592-022-01496-1
    title: A learned embedding for efficient joint analysis of millions of mass spectra
  source_content_hash: sha256:521585b99b6480079fb1165ec41d1c0bded1bc3a03f915fb24cfdb881ad71670
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bittremieux/GLEAMS
  tools:
  - ANN-SoLo
  - GLEAMS
  - Python
  - Python 3.8
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# msms-spectrum-feature-encoding

## Summary

Reconstructs the GLEAMS input feature vector: each MS/MS spectrum is encoded as a 3010-dimensional vector combining Gray-coded precursor mass and m/z, one-hot charge, square-root transformed 2449-bin fragment intensities (50.5–2500 m/z), and normalized dot-product similarities to 500 reference spectra.

## When to use

Use when reimplementing, validating, or extending GLEAMS's spectrum encoder, or when converting raw MS/MS spectra (precursor mass, charge, peak lists) into the exact 3010-feature input format its neural network expects.

## When NOT to use

- The goal is a final embedding, not the raw input features — use the `gleams embed` command instead, which directly outputs 32-dimensional embeddings.
- The encoding scheme being validated is not GLEAMS's (e.g., a different spectral embedding pipeline with different bin ranges or reference sets).

## Inputs

- Single MS/MS spectrum with precursor mass, precursor m/z, precursor charge, and peak list (e.g., from an mzML peak file)
- Invariant reference set of 500 reference spectra for similarity features

## Outputs

- 3010-dimensional feature vector per spectrum (Gray-coded precursor mass + m/z, one-hot charge, 2449 sqrt-transformed fragment intensity bins, 500 reference-spectrum dot-product similarities)
- Feature vector ready to be fed to the GLEAMS neural network to produce a 32-dimensional embedding

## How to apply

Load one MS/MS spectrum (precursor mass, precursor m/z, charge, and peak list) from a peak file such as mzML. Encode the precursor mass and precursor m/z each as a 27-bit Gray code, and the precursor charge as a one-hot vector over 7 states. Bin fragment ion intensities across the 50.5–2500 m/z range into 2449 bins and apply a square-root transform to the binned intensities. Compute normalized dot-product similarities between the spectrum and an invariant set of 500 reference spectra (the top 500 precursor mass differences from the ANN-SoLo open modification search of GLEAMS cluster centroids). Concatenate all feature blocks and verify the resulting vector is exactly 3010 features; this vector is what feeds GLEAMS's neural network to produce the 32-dimensional embedding.

## Related tools

- **GLEAMS** (Source implementation of the spectrum encoder whose config (gleams/config.py) defines the feature blocks and reference-spectrum set; can be used to cross-check a reconstructed encoder.) — https://github.com/bittremieux/GLEAMS
- **Python 3.8** (Runtime for reimplementing the encoding steps (Gray coding, binning, square-root transform, dot-product similarities).)
- **ANN-SoLo** (Source of the top 500 precursor mass differences from the open modification search of GLEAMS cluster centroids used as the reference-spectrum similarity features.)

## Examples

```
gleams embed *.mzML --embed_name GLEAMS_embed
```

## Evaluation signals

- Concatenated feature vector has dimension exactly 3010 for every encoded spectrum.
- Fragment-intensity block has exactly 2449 bins spanning 50.5–2500 m/z with square-root transformed values.
- Precursor mass and precursor m/z encodings are each 27 bits and use Gray codes (adjacent values differ in exactly one bit).
- Charge one-hot vector has exactly 7 states.
- Similarity block contains exactly 500 normalized dot-product values against the reference spectra.
- Downstream check: feeding encoded vectors through the pretrained model (gleams_82c0124b.hdf5) yields 32-dimensional embeddings.

## Limitations

- The feature encoding requires GLEAMS's environment: Python 3.8, a Linux operating system, and a CUDA-enabled GPU if running through the neural network.
- The 500 reference spectra depend on the ANN-SoLo open modification search of GLEAMS cluster centroids; a different reference set changes the similarity features.
- This skill reproduces the network input features, not the trained embedding itself; the 32-dimensional output requires the pretrained weights.

## Evidence

- [readme] GLEAMS encodes each spectrum as a 3010-dimensional feature vector: "GLEAMS encodes mass spectra as vectors of features and feeds them to a neural network to embed them into a 32-dimensional space"
- [supplementary] precursor Gray codes and charge one-hot: [evidence span withheld — anchor status: not_found]
- [supplementary] fragment binning and transform: [evidence span withheld — anchor status: not_found]
- [supplementary] reference spectrum similarities: [evidence span withheld — anchor status: not_found]
- [supplementary] reference spectra provenance: "Top 500 precursor mass differences from the ANN-SoLo open modification search of GLEAMS cluster centroids."
- [readme] embed command output: "command to convert MS/MS spectra in peak files to 32-dimensional embeddings"
