---
name: pytorch-neural-module-implementation
description: Use when implementing a spectrum encoder that must convert variable-length
  peak sets {P_k = (M_k, I_k)} into one token per peak, e.g. to condition a generative
  model via cross-attention.
when_to_use_negative:
- When spectra have already been pooled into a single fixed-length vector and per-peak
  tokens are not needed (the MADGEN design requires individual peak tokens as cross-attention
  keys).
- When the task is scaffold retrieval/ranking rather than spectrum tokenization —
  that stage uses contrastive learning against candidate scaffolds, not the peak-embedding
  module.
edam_operation: http://edamontology.org/operation_3936
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_0154
tools:
- name: PyTorch
  role: Framework used to implement the two MLP peak encoders and the multi-head self-attention
    module over peak embeddings
  repo: https://github.com/HassounLab/MADGEN
- name: MADGEN
  role: 'Downstream consumer: the tokenized spectrum H_X conditions MADGEN''s graph
    generator via cross-attention'
  repo: https://github.com/HassounLab/MADGEN
- name: Python 3.9
  role: Runtime for the encoder module, created via the madgen conda environment
- name: conda
  role: Environment manager used to create the madgen environment (python=3.9, rdkit=2023.09.5)
provenance:
  source_task_ids:
  - task_005
  source_content_hash: sha256:e1df0d8d0206c22e3bdfc2c0d8cae651fda08991f27a515d3ba9881ccd541910
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
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# pytorch-neural-module-implementation

## Summary

Implement a PyTorch neural module that tokenizes MS/MS spectra: two MLP encoders embed each peak's m/z (M_k) and intensity (I_k), concatenated into per-peak embeddings h'_k (Eq. 8), followed by multi-head self-attention over the variable-length peak sequence to yield spectrum tokens H_X for downstream cross-attention conditioning.

## When to use

Use when implementing a spectrum encoder that must convert variable-length peak sets {P_k = (M_k, I_k)} into one token per peak, e.g. to condition a generative model via cross-attention.

## When NOT to use

- When spectra have already been pooled into a single fixed-length vector and per-peak tokens are not needed (the MADGEN design requires individual peak tokens as cross-attention keys).
- When the task is scaffold retrieval/ranking rather than spectrum tokenization — that stage uses contrastive learning against candidate scaffolds, not the peak-embedding module.

## Inputs

- Preprocessed MS/MS spectra with normalized intensities and low-intensity peaks removed (msgym.pkl from MADGEN Zenodo record 15036069)
- Per-spectrum peak sets P_k = (M_k, I_k) with m/z values M_k and intensities I_k

## Outputs

- Per-peak embedding tensor h'_k produced by concatenated MLP encoders of m/z and intensity (Eq. 8)
- Spectrum token representation H_X: a variable-length set of self-attention outputs with one representation per observed peak

## How to apply

Represent each preprocessed spectrum (normalized intensities, low-intensity peaks removed) as a set of peaks P_k = (M_k, I_k) per the paper's tokenization scheme. Implement two separate PyTorch MLP encoders — one for the m/z value M_k and one for the intensity I_k — and concatenate their outputs into the per-peak embedding h'_k following Eq. 8 of the MADGEN paper. Apply a PyTorch multi-head self-attention module over the sequence h'_1..h'_K to produce the spectrum token representation H_X, using masking to support variable-length peak sets. Build the module in the madgen conda environment (Python 3.9, PyTorch) and unit-test it on spectra loaded from the msgym.pkl Zenodo deposit to confirm each observed peak maps to exactly one token and H_X carries one representation per peak, ready for downstream cross-attention in which graph nodes and edges query individual peak tokens.

## Related tools

- **PyTorch** (Framework used to implement the two MLP peak encoders and the multi-head self-attention module over peak embeddings) — https://github.com/HassounLab/MADGEN
- **MADGEN** (Downstream consumer: the tokenized spectrum H_X conditions MADGEN's graph generator via cross-attention) — https://github.com/HassounLab/MADGEN
- **Python 3.9** (Runtime for the encoder module, created via the madgen conda environment)
- **conda** (Environment manager used to create the madgen environment (python=3.9, rdkit=2023.09.5))

## Examples

```
conda create --name madgen python=3.9 rdkit=2023.09.5 -c conda-forge -y; conda activate madgen; wget https://zenodo.org/records/15036069/files/msgym.pkl?download=1 -O ./data/msgym/raw/msgym.pkl
```

## Evaluation signals

- Each peak P_k = (M_k, I_k) maps to exactly one token; token count equals the number of observed peaks in the spectrum.
- H_X is a variable-length token set with one representation per peak, matching the paper's Eq. 8 formulation (MLP embedding followed by self-attention).
- The module runs end-to-end on spectra loaded from the msgym.pkl dataset without shape/masking errors.
- Masking correctly handles batches of spectra with differing numbers of peaks (variable-length peak sets).

## Limitations

- Preprocessing assumptions must hold first: intensities must be normalized and low-intensity peaks below a predefined threshold removed to reduce noise, otherwise the token embeddings carry noise peaks.
- The module is designed for peak-token outputs consumed by cross-attention; it does not by itself handle scaffold retrieval or candidate ranking.
- The NIST23 dataset is commercially available and requires separate licensing, so unit-testing may be limited to msgym.pkl and canopus.pkl.

## Evidence

- [methods] MLP peak embedding + self-attention (Eq. 8): [evidence span withheld — anchor status: not_found]
