---
name: mass-spectra-conditioning-representation-learning
description: Use when MS/MS spectra (peak lists with m/z and intensity pairs) must
  be encoded as per-peak tokens to condition a graph-based or attention-based molecular
  generative model.
when_to_use_negative:
- When spectra are not yet preprocessed — intensities must first be normalized and
  low-intensity peaks below the predefined threshold removed before tokenization
- When the downstream model consumes whole-spectrum fixed-length fingerprints rather
  than per-peak tokens queried individually by graph nodes and edges
tools:
- name: PyTorch
  role: Implements the two MLP peak encoders and the multi-head self-attention module
    over peak embeddings
- name: Python 3.9
  role: Runtime environment for the tokenization and self-attention modules
- name: conda
  role: Creates the madgen environment (python=3.9 rdkit=2023.09.5) in which the encoder
    modules run and are unit-tested
- name: MADGEN
  role: Source of the tokenization scheme (Eq. 8), the preprocessed msgym.pkl dataset,
    and the cross-attention conditioning architecture that consumes H_X
  repo: https://github.com/HassounLab/MADGEN
provenance:
  source_task_ids:
  - task_005
  source_content_hash: sha256:7916edd6974c383e1cad1f79f997f5a8242b9d33f1ac8641b48340d2c07327a5
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
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6
n_claims: 15
---

# mass-spectra-conditioning-representation-learning

## Summary

Tokenize preprocessed MS/MS spectra into per-peak embeddings by encoding m/z values and intensities with separate MLPs, then applying multi-head self-attention over peak embeddings to yield a variable-length spectrum token set (H_X) usable for cross-attention conditioning of a scaffold-conditioned molecular generator.

## When to use

Use when MS/MS spectra (peak lists with m/z and intensity pairs) must be encoded as per-peak tokens to condition a graph-based or attention-based molecular generative model.

## When NOT to use

- When spectra are not yet preprocessed — intensities must first be normalized and low-intensity peaks below the predefined threshold removed before tokenization
- When the downstream model consumes whole-spectrum fixed-length fingerprints rather than per-peak tokens queried individually by graph nodes and edges

## Inputs

- Preprocessed MS/MS spectra as peak sets {P_k = (M_k, I_k)} with normalized intensities and low-intensity peaks removed (msgym.pkl from Zenodo record 15036069)
- Peak count K and per-peak masks for variable-length spectra

## Outputs

- Per-peak embedding sequence h'_1..h'_K produced by concatenated m/z and intensity MLPs (Eq. 8)
- Spectrum token representation H_X: a variable-length set with one representation per observed peak, ready for downstream cross-attention conditioning

## How to apply

Load preprocessed spectra (normalized intensities, low-intensity peaks removed below a predefined threshold) from the MADGEN Zenodo deposit msgym.pkl in the madgen conda environment (Python 3.9). Represent each spectrum as a set of peaks {P_1,...,P_K} with P_k = (M_k, I_k) following the paper's tokenization scheme. Implement two separate PyTorch MLP encoders — one for the m/z value M_k and one for the intensity I_k — and concatenate their outputs into the per-peak embedding h'_k per Eq. 8 of the paper. Apply a PyTorch multi-head self-attention module over the sequence h'_1..h'_K to produce the spectrum token representation H_X, using masking to support variable-length peak sets so each observed peak maps to exactly one token.

## Related tools

- **PyTorch** (Implements the two MLP peak encoders and the multi-head self-attention module over peak embeddings)
- **Python 3.9** (Runtime environment for the tokenization and self-attention modules)
- **conda** (Creates the madgen environment (python=3.9 rdkit=2023.09.5) in which the encoder modules run and are unit-tested)
- **MADGEN** (Source of the tokenization scheme (Eq. 8), the preprocessed msgym.pkl dataset, and the cross-attention conditioning architecture that consumes H_X) — https://github.com/HassounLab/MADGEN

## Examples

```
wget https://zenodo.org/records/15036069/files/msgym.pkl?download=1 -O ./data/msgym/raw/msgym.pkl
```

## Evaluation signals

- Each peak P_k = (M_k, I_k) maps to exactly one token in H_X (one representation per observed peak)
- H_X is a variable-length token set across spectra, with masking correctly handling different peak counts K
- The module runs end-to-end on spectra loaded from msgym.pkl, matching the Eq. 8 formulation (MLP peak embedding followed by self-attention)
- Downstream cross-attention accepts H_X such that molecular graph nodes and edges can query individual peak tokens

## Limitations

- Requires preprocessed spectra: normalized intensities and low-intensity peaks removed below a predefined threshold; raw spectra will inject noise into peak tokens
- The NIST23 dataset is commercially available and requires separate licensing, limiting the freely reproducible training data to msgym.pkl and canopus.pkl

## Evidence
