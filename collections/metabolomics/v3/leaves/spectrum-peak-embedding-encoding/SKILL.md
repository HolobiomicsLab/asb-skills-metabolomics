---
name: spectrum-peak-embedding-encoding
description: 'Use when preparing MS2 spectra (HDF5 or preprocessed arrays) as model
  input for LipiDetective or a similar transformer-based lipid identifier: converting
  raw peak lists into fixed-length embedded sequences with m/z-precision-aware vocabularies,
  intensity-by-ordering encoding, and label tokenization.'
when_to_use_negative:
- The task is spectrum-to-spectrum similarity or library matching (e.g., cosine scoring
  against GNPS or LipiDex reference entries) rather than learning a fragmentation
  pattern into embeddings — use a spectral library search skill instead.
- Only raw profile spectra are available (no smoothing, baseline correction, or peak
  picking yet); the embedding encoding assumes already-picked, base-peak-normalized
  peaks trimmed to 50–1600 m/z.
- Intensities themselves (not just their ordering) are required as explicit model
  inputs — this encoding deliberately discards raw intensity values and only preserves
  intensity rank via peak ordering.
edam_operation: http://edamontology.org/operation_0336
edam_topics:
- http://edamontology.org/topic_0154
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3474
tools:
- name: LipiDetective
  role: Framework containing the transformer network whose input embedding scheme
    is being encoded
  repo: github.com/LipiTUM/lipidetective
- name: PyTorch
  role: Provides the modified transformer module whose encoder/decoder layers consume
    the embedded peaks and tokens
- name: h5py
  role: Reads the HDF5 training spectra (2xN arrays with lipid_species/adduct/precursor
    attributes) that feed the embedding pipeline
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1101/2024.10.07.617094
    title: LipiDetective a deep learning model for the identification of molecular
      lipid species in tandem mass spectra
  source_content_hash: sha256:5173d713c8099e997741b0c7c15309fe6363492e070760df7f02a20356817782
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - LipiDetective
  - PyTorch
  - h5py
  edam_operation: http://edamontology.org/operation_0336
  edam_topics:
  - http://edamontology.org/topic_0154
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# spectrum-peak-embedding-encoding

## Summary

Encodes MS2 peak m/z values into transformer input embeddings by truncating m/z to fixed decimal precision (1 decimal place, max 1600 m/z, 30 peaks), ordering peaks by descending intensity (which implicitly encodes intensity since raw values are discarded), and adding positional embeddings for the tokenized lipid species label. Core feature-preparation step for training and inference with the LipiDetective transformer.

## When to use

Use when preparing MS2 spectra (HDF5 or preprocessed arrays) as model input for LipiDetective or a similar transformer-based lipid identifier: converting raw peak lists into fixed-length embedded sequences with m/z-precision-aware vocabularies, intensity-by-ordering encoding, and label tokenization.

## When NOT to use

- The task is spectrum-to-spectrum similarity or library matching (e.g., cosine scoring against GNPS or LipiDex reference entries) rather than learning a fragmentation pattern into embeddings — use a spectral library search skill instead.
- Only raw profile spectra are available (no smoothing, baseline correction, or peak picking yet); the embedding encoding assumes already-picked, base-peak-normalized peaks trimmed to 50–1600 m/z.
- Intensities themselves (not just their ordering) are required as explicit model inputs — this encoding deliberately discards raw intensity values and only preserves intensity rank via peak ordering.

## Inputs

- Preprocessed MS2 spectra stored in HDF5 file under /all_datasets/ group (2xN arrays of m/z and intensity, with lipid_species, adduct, precursor attributes)
- Configuration YAML (config_transformer.yaml) with input_embedding settings (n_peaks=30, max_mz=1600, decimal_accuracy=1) and transformer dims (d_model=32, num_heads=4, num_layers=2, output_seq_length=11)
- Lipid species names in shorthand notation (headgroup, fatty acid side chains, adduct) for decoder-side tokenization

## Outputs

- Fixed-length (n_peaks=30) sequence of d_model=32-dimensional embedded input tokens, one per retained peak, m/z truncated to 1 decimal place and ordered by descending intensity
- Tokenized lipid species label sequence (headgroup, fatty acid chains, adduct tokens) for the transformer decoder
- Transformer output token sequence of at most output_seq_length=11 predicted lipid tokens
- Modified PyTorch transformer module (src/lipidetective/models/transformer_network.py)

## How to apply

1. Read the model configuration from config/config_templates/config_transformer.yaml to obtain input_embedding settings (n_peaks 30, max_mz 1600, decimal_accuracy 1) and transformer dimensions (d_model 32, num_heads 4, dropout 0.1, ffn_hidden 256, num_layers 2, output_seq_length 11). 2. Implement the input-peak encoding: truncate each retained peak's m/z to the configured decimal accuracy (1 decimal point) and map it to its corresponding d_model-dimensional embedding vector; keep d_model divisible by num_heads. 3. Implement positional encoding added to each peak embedding — since peaks are ordered by decreasing intensity, position conveys relative intensity rank without explicit intensity values. 4. Tokenize the lipid species label (headgroup, fatty acid side chains, adduct) via regex splitting and map each token to an embedding vector for the decoder. 5. Wire embedded peaks and tokens through PyTorch's encoder/decoder layers, honoring output_seq_length 11; spectra come from the /all_datasets/ group of an HDF5 file (2xN arrays with lipid_species/adduct/precursor attributes). Validate with a forward pass returning a token sequence of at most 11 tokens.

## Related tools

- **LipiDetective** (Framework containing the transformer network whose input embedding scheme is being encoded) — github.com/LipiTUM/lipidetective
- **PyTorch** (Provides the modified transformer module whose encoder/decoder layers consume the embedded peaks and tokens)
- **h5py** (Reads the HDF5 training spectra (2xN arrays with lipid_species/adduct/precursor attributes) that feed the embedding pipeline)

## Examples

```
uv run lipidetective --config config/config_templates/config_transformer.yaml
```

## Evaluation signals

- Configuration values match config_transformer.yaml: n_peaks=30, max_mz=1600, decimal_accuracy=1, d_model=32, num_heads=4, num_layers=2, output_seq_length=11.
- Forward pass on a sample HDF5 spectrum produces a predicted token sequence of at most 11 tokens.
- Module file src/lipidetective/models/transformer_network.py imports without error.
- Input sequence length is exactly 30 (n_peaks) with each token of dimension 32 (d_model), and d_model is divisible by num_heads (4).
- m/z values in the embedded vocabulary are truncated to 1 decimal place and capped at 1600; no raw intensity values appear in the input tokens.

## Limitations

- Raw peak intensity values are never passed to the model; only peak ordering by descending intensity conveys intensity information, so this encoding cannot distinguish two peaks of equal intensity at different positions.
- Fixed m/z decimal accuracy (1 decimal place) and max_mz (1600) constrain the input feature space; peaks outside 50–1600 m/z or requiring finer precision are lost or merged.
- The input vocabulary scales with the number of distinct truncated m/z values; very small d_model (32) may limit expressiveness for complex lipid fragmentation patterns.
- Blank spectra (2002 MS2 spectra from blank measurements) must be included in training so the model learns to predict an end-of-sequence token when no lipid species is present — without them the encoding/prediction step over-predicts on empty spectra.

## Evidence

- [methods] Input embedding of m/z peaks: [evidence span withheld — anchor status: not_found]
- [methods] Config: n_peaks=30, max_mz=1600, decimal_accuracy=1: [evidence span withheld — anchor status: not_found]
- [methods] Positional encoding conveys intensity rank: [evidence span withheld — anchor status: not_found]
- [methods] Label tokenization via regex splitting: [evidence span withheld — anchor status: not_found]
- [methods] HDF5 spectrum format: [evidence span withheld — anchor status: not_found]
- [readme] LipiDetective runs from YAML config: "**LipiDetective** is a deep learning framework designed for the identification of molecular lipid species from tandem mass spectra."
- [abstract] Model generalizes to unseen species: "The model can even correctly predict lipid species for which it has never seen a spectrum before as it is able to generalize the learned lipid fragmentation patterns"
