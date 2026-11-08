---
name: mlp-peak-embedding-encoding
description: Use when MS/MS peak lists (m/z, intensity pairs) must be tokenized into
  variable-length per-peak embeddings so that a downstream generator can attend to
  individual peaks via cross-attention rather than to a pooled spectrum vector.
when_to_use_negative:
- Input is a single pooled/fixed-length spectrum vector — this skill's purpose is
  to preserve one token per peak for node-level cross-attention, not to produce a
  summarized embedding
- The task is scaffold retrieval (contrastive ranking of candidate scaffolds) rather
  than spectra-conditioned molecular generation, where per-peak token conditioning
  is not the required interface
- Spectra have not been preprocessed (intensities not normalized, low-intensity noise
  peaks not removed), which would degrade the token set quality
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3474
tools:
- name: PyTorch
  role: Implement the two MLP encoders (for m/z M_k and intensity I_k) and the multi-head
    self-attention module over the peak embeddings
- name: MADGEN
  role: Target framework whose cross-attention generator consumes the resulting spectrum
    token representation H_X
  repo: https://github.com/HassounLab/MADGEN
- name: Python 3.9
  role: Runtime environment (madgen conda env) in which the tokenization module is
    implemented and unit-tested
provenance:
  source_task_ids:
  - task_005
  source_content_hash: sha256:fab5f8856a3fa81c3a504eba59fc4d9fab14ea1cd9ba2d35bdd3f65395a8453c
  source_papers:
  - doi: 10.48550/arxiv.2501.01950
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/HassounLab/MADGEN
  tools:
  - MADGEN
  - PyTorch
  - Python
  - Python 3.9
  - conda
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5294117647058824
n_claims: 17
---

# mlp-peak-embedding-encoding

## Summary

Encodes each MS/MS spectrum as a set of per-peak tokens by passing m/z values and intensities through separate MLP encoders, concatenating the outputs into per-peak embeddings, and applying multi-head self-attention over the peak sequence to yield the spectrum token representation H_X used for cross-attention conditioning in MADGEN.

## When to use

Use when MS/MS peak lists (m/z, intensity pairs) must be tokenized into variable-length per-peak embeddings so that a downstream generator can attend to individual peaks via cross-attention rather than to a pooled spectrum vector.

## When NOT to use

- Input is a single pooled/fixed-length spectrum vector — this skill's purpose is to preserve one token per peak for node-level cross-attention, not to produce a summarized embedding
- The task is scaffold retrieval (contrastive ranking of candidate scaffolds) rather than spectra-conditioned molecular generation, where per-peak token conditioning is not the required interface
- Spectra have not been preprocessed (intensities not normalized, low-intensity noise peaks not removed), which would degrade the token set quality

## Inputs

- Preprocessed MS/MS spectra (normalized intensities, low-intensity peaks removed) from the MADGEN Zenodo deposit msgym.pkl
- Per-spectrum peak sets P_k = (M_k, I_k) of m/z values and intensities

## Outputs

- Per-peak embedding h'_k for each observed peak (concatenated MLP outputs for M_k and I_k)
- Spectrum token representation H_X: a variable-length set of tokens with one representation per peak, ready for downstream cross-attention conditioning

## How to apply

Load preprocessed spectra from the MADGEN Zenodo deposit (msgym.pkl), where intensities are normalized and low-intensity peaks below a predefined threshold have been removed. Represent each spectrum as a peak set {P_1,...,P_K} with P_k = (M_k, I_k). Implement two separate PyTorch MLP encoders — one for the m/z value M_k and one for intensity I_k — and concatenate their outputs into the per-peak embedding h'_k following Eq. 8 of the paper. Apply a PyTorch multi-head self-attention module over the sequence h'_1..h'_K to produce H_X, using masking to support variable-length peak sets. The rationale is that MADGEN's graph generator queries individual peak tokens via cross-attention, so each observed peak must map to exactly one token.

## Related tools

- **PyTorch** (Implement the two MLP encoders (for m/z M_k and intensity I_k) and the multi-head self-attention module over the peak embeddings)
- **MADGEN** (Target framework whose cross-attention generator consumes the resulting spectrum token representation H_X) — https://github.com/HassounLab/MADGEN
- **Python 3.9** (Runtime environment (madgen conda env) in which the tokenization module is implemented and unit-tested)

## Examples

```
wget https://zenodo.org/records/15036069/files/msgym.pkl?download=1 -O ./data/msgym/raw/msgym.pkl
```

## Evaluation signals

- Each observed peak in a loaded spectrum maps to exactly one token in H_X (no pooling, no dropped peaks)
- H_X is a variable-length token set whose length equals the number of peaks in the input spectrum, confirmed by running end-to-end on spectra from msgym.pkl
- Masking correctly handles variable-length peak sets in mini-batches (no attention leakage across padded positions)
- The per-peak embedding h'_k matches the paper's Eq. 8 formulation: separate MLP encodings of M_k and I_k concatenated
- The module runs end-to-end inside the madgen conda environment (python=3.9, rdkit=2023.09.5) without shape errors on real spectra

## Limitations

- Low-intensity peaks must be removed and intensities normalized during preprocessing, otherwise noise peaks become spurious tokens
- The encoder is designed for variable-length peak sets, so padding/masking must be handled carefully in batched training
- The token representation H_X is an intermediate for MADGEN's cross-attention conditioning; it is not directly comparable to pooled spectral embeddings used by retrieval baselines like Spec2Mol

## Evidence
