---
name: self-attention-over-variable-length-peak-sequences
description: Use when MS/MS peak lists {P_k = (M_k, I_k)} must be encoded as one token
  per peak for downstream cross-attention conditioning of a scaffold-conditioned molecular
  graph generator, rather than pooled into a fixed-length spectrum vector.
when_to_use_negative:
- When the downstream model requires a single fixed-length spectrum embedding (e.g.
  contrastive scaffold retrieval alignment) rather than per-peak tokens queried individually
  by graph nodes and edges
- When spectra have not been preprocessed — intensities not normalized and low-intensity
  peaks not removed below the predefined threshold — since the tokenization assumes
  MADGEN's noise-reduced peak sets
- When the target generator conditions via concatenation or FiLM rather than cross-attention
  over individual peak tokens
edam_operation: http://edamontology.org/operation_3936
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: PyTorch
  role: Provides the two MLP encoders for m/z and intensity and the multi-head self-attention
    module (with masking) used to build H_X from peak embeddings
- name: MADGEN
  role: 'Downstream consumer of H_X: its graph generator''s cross-attention lets molecular
    graph nodes and edges query individual peak tokens'
  repo: https://github.com/HassounLab/MADGEN
- name: Python 3.9
  role: Runtime for the madgen conda environment in which the tokenizer/encoder module
    is implemented and unit-tested
  repo: https://github.com/HassounLab/MADGEN
- name: conda
  role: Creates the madgen environment (python=3.9, rdkit=2023.09.5) in which the
    peak-encoding module is developed and tested
provenance:
  source_task_ids:
  - task_005
  source_content_hash: sha256:d53493b5f53216168b26e5088fd8729e2d374ca0691709dcfd838147200a3dcc
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
  edam_operation: http://edamontology.org/operation_3936
  edam_topics:
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# self-attention-over-variable-length-peak-sequences

## Summary

Tokenizes an MS/MS spectrum into per-peak embeddings — separate MLP encoders for m/z (M_k) and intensity (I_k) concatenated per peak (Eq. 8) — then applies multi-head self-attention over the variable-length peak sequence to produce the spectrum token set H_X used by MADGEN's cross-attention-conditioned graph generator.

## When to use

Use when MS/MS peak lists {P_k = (M_k, I_k)} must be encoded as one token per peak for downstream cross-attention conditioning of a scaffold-conditioned molecular graph generator, rather than pooled into a fixed-length spectrum vector.

## When NOT to use

- When the downstream model requires a single fixed-length spectrum embedding (e.g. contrastive scaffold retrieval alignment) rather than per-peak tokens queried individually by graph nodes and edges
- When spectra have not been preprocessed — intensities not normalized and low-intensity peaks not removed below the predefined threshold — since the tokenization assumes MADGEN's noise-reduced peak sets
- When the target generator conditions via concatenation or FiLM rather than cross-attention over individual peak tokens

## Inputs

- Preprocessed MS/MS spectra from the MADGEN Zenodo deposit (msgym.pkl), with normalized intensities and low-intensity peaks removed
- Peak sets {P_1,...,P_K} with P_k = (M_k, I_k) per the paper's tokenization scheme

## Outputs

- Per-peak embedding sequence h'_1..h'_K (concatenated m/z-MLP and intensity-MLP outputs, Eq. 8)
- Spectrum token representation H_X: a variable-length token set with one representation per observed peak, ready for cross-attention conditioning

## How to apply

Load preprocessed spectra (normalized intensities, low-intensity peaks below the predefined threshold already removed) from the MADGEN Zenodo deposit msgym.pkl in the madgen conda environment (Python 3.9). Represent each spectrum as a peak set with P_k = (M_k, I_k) per the paper's tokenization scheme. Implement two separate PyTorch MLP encoders — one for the m/z value M_k, one for the intensity I_k — and concatenate their outputs into the per-peak embedding h'_k following Eq. 8. Apply a PyTorch multi-head self-attention module over the sequence h'_1..h'_K with masking so peak sets of arbitrary length K are supported, yielding H_X with one representation per observed peak. H_X then serves as the key/value memory that molecular graph nodes and edges query in MADGEN's cross-attention conditioning of the Markov-bridge generator.

## Related tools

- **PyTorch** (Provides the two MLP encoders for m/z and intensity and the multi-head self-attention module (with masking) used to build H_X from peak embeddings)
- **MADGEN** (Downstream consumer of H_X: its graph generator's cross-attention lets molecular graph nodes and edges query individual peak tokens) — https://github.com/HassounLab/MADGEN
- **Python 3.9** (Runtime for the madgen conda environment in which the tokenizer/encoder module is implemented and unit-tested) — https://github.com/HassounLab/MADGEN
- **conda** (Creates the madgen environment (python=3.9, rdkit=2023.09.5) in which the peak-encoding module is developed and tested)

## Examples

```
attn = torch.nn.MultiheadAttention(embed_dim=d_model, num_heads=4, batch_first=True); H_X = attn(h_prime, h_prime, h_prime, key_padding_mask=~peak_mask)[0]
```

## Evaluation signals

- Each peak P_k = (M_k, I_k) maps to exactly one token — token count K equals the number of observed peaks after low-intensity filtering
- H_X is a variable-length token set (not a pooled vector), with one representation per observed peak, matching the Eq. 8 formulation (MLP peak embedding followed by self-attention)
- Masking correctly excludes padding positions so spectra of different lengths K produce valid outputs without attending to padded peaks
- The module runs end-to-end on spectra loaded from the MADGEN Zenodo msgym.pkl deposit in the madgen environment
- Downstream, graph nodes/edges in the generator can cross-attend to H_X tokens without shape mismatches

## Limitations

- Requires preprocessed spectra: intensities must be normalized and low-intensity peaks below the predefined threshold removed beforehand to reduce noise
- Spectra longer than the attention module's maximum sequence budget require truncation or adjusted masking; variable-length handling depends on correct key-padding masks
- Designed for MADGEN's cross-attention conditioning; reusing H_X with non-attention generators would need an additional pooling/aggregation step not described in the paper

## Evidence

- [results] MADGEN conditions its graph generator on the spectrum through a cross-attention mechanism in which molecular graph nodes and edges directly query the individual peak tokens of the tokenized spectrum: [evidence span withheld — anchor status: not_found]
