---
name: spectral-peak-tokenization
description: Use when raw MS/MS peak lists must be converted into per-peak token embeddings
  to condition a generative or cross-attention model on spectral information, as in
  MADGEN-style spectra-guided molecular generation.
when_to_use_negative:
- Input spectra are not yet preprocessed (unnormalized intensities, noise peaks not
  thresholded) — preprocess first per the MADGEN workflow
- The downstream model needs a single fixed-length global spectrum vector rather than
  per-peak tokens queried by graph nodes and edges
- Working on scaffold retrieval/ranking (contrastive learning stage) rather than the
  spectra-conditioned generation stage of MADGEN
edam_operation: http://edamontology.org/operation_2928
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: PyTorch
  role: Implements the two MLP peak encoders (m/z and intensity) and the multi-head
    self-attention module over peak embeddings
  repo: https://github.com/HassounLab/MADGEN
- name: Python 3.9
  role: Runtime environment for loading preprocessed spectra and running the tokenization
    module
- name: conda
  role: Creates the madgen environment (python=3.9, rdkit=2023.09.5) in which the
    tokenization module is developed and unit-tested
- name: MADGEN
  role: 'Downstream consumer of H_X: its graph generator cross-attends molecular graph
    nodes and edges to the individual peak tokens'
  repo: https://github.com/HassounLab/MADGEN
provenance:
  source_task_ids:
  - task_005
  source_content_hash: sha256:366a0c26958adca5ebdf099898a1cdf6339444f054cc9dd515815eb03dd834a8
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
  edam_operation: http://edamontology.org/operation_2928
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6923076923076923
n_claims: 13
---

# spectral-peak-tokenization

## Summary

Encodes a preprocessed MS/MS spectrum as a variable-length set of peak tokens by embedding each (m/z, intensity) pair with paired MLP encoders and applying multi-head self-attention, yielding per-peak representations (H_X) for downstream conditioning of MADGEN's scaffold-conditioned generator.

## When to use

Use when raw MS/MS peak lists must be converted into per-peak token embeddings to condition a generative or cross-attention model on spectral information, as in MADGEN-style spectra-guided molecular generation.

## When NOT to use

- Input spectra are not yet preprocessed (unnormalized intensities, noise peaks not thresholded) — preprocess first per the MADGEN workflow
- The downstream model needs a single fixed-length global spectrum vector rather than per-peak tokens queried by graph nodes and edges
- Working on scaffold retrieval/ranking (contrastive learning stage) rather than the spectra-conditioned generation stage of MADGEN

## Inputs

- Preprocessed MS/MS spectra with normalized intensities and low-intensity peaks removed (e.g. MADGEN msgym.pkl from Zenodo record 15036069)
- Peak list per spectrum: set of peaks {P_1,...,P_K} with P_k = (m/z M_k, intensity I_k)

## Outputs

- Per-peak embedding h'_k formed by concatenating the m/z MLP and intensity MLP outputs (Eq. 8 formulation)
- Variable-length spectrum token representation H_X with one token per observed peak, ready for downstream cross-attention conditioning

## How to apply

Start from preprocessed spectra in which intensities are normalized and low-intensity peaks below a predefined threshold have been removed (as in the MADGEN Zenodo msgym.pkl deposit). Represent each spectrum as a set of peaks {P_1,...,P_K} with P_k = (M_k, I_k) per the paper's tokenization scheme. Implement two separate PyTorch MLP encoders, one for the m/z value M_k and one for the intensity I_k, and concatenate their outputs into the per-peak embedding h'_k following Eq. 8 of the paper. Apply a PyTorch multi-head self-attention module over the sequence of peak embeddings h'_1..h'_K to yield the spectrum token representation H_X, using masking to support variable-length peak sets. The output H_X carries one representation per observed peak and is consumed by MADGEN's graph generator, whose molecular graph nodes and edges cross-attend to individual peak tokens.

## Related tools

- **PyTorch** (Implements the two MLP peak encoders (m/z and intensity) and the multi-head self-attention module over peak embeddings) — https://github.com/HassounLab/MADGEN
- **Python 3.9** (Runtime environment for loading preprocessed spectra and running the tokenization module)
- **conda** (Creates the madgen environment (python=3.9, rdkit=2023.09.5) in which the tokenization module is developed and unit-tested)
- **MADGEN** (Downstream consumer of H_X: its graph generator cross-attends molecular graph nodes and edges to the individual peak tokens) — https://github.com/HassounLab/MADGEN

## Examples

```
conda activate madgen && python -c "import pickle; spectra = pickle.load(open('./data/msgym/raw/msgym.pkl','rb'))"
```

## Evaluation signals

- Each spectrum peak maps to exactly one token (one-to-one peak-to-token correspondence)
- H_X is a variable-length token set whose length equals the number of observed peaks after thresholding, matching the paper's Eq. 8 formulation
- The module runs end-to-end on spectra loaded from the MADGEN Zenodo deposit within the madgen conda environment
- Downstream cross-attention can consume H_X directly: molecular graph nodes and edges query individual peak tokens without shape errors
- Variable-length batch inputs are handled correctly via masking (no leakage across padded positions)

## Limitations

- Low-intensity peaks are discarded below a predefined threshold, so tokens only cover retained peaks
- Designed for the MADGEN pipeline's spectra-conditioned generation; its utility outside cross-attention conditioning is not established here
- Requires the madgen conda environment (Python 3.9, RDKit 2023.09.5) as specified by the repository

## Evidence
