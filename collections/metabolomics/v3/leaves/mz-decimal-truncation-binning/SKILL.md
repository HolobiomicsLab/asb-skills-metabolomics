---
name: mz-decimal-truncation-binning
description: 'Use when building or reproducing LipiDetective''s transformer input
  encoding: MS2 peak lists (m/z + intensity) from base-peak-normalized spectra must
  be converted to embedding tokens with m/z precision fixed by config decimal_accuracy.'
when_to_use_negative:
- When the downstream task requires actual fragment intensities as numerical features
  — LipiDetective's encoder only receives intensity information implicitly via peak
  ordering by descending intensity.
- When spectra have not been preprocessed (smoothing, baseline correction, peak picking,
  base-peak normalization, trimming to 50-1600 m/z) — raw profile spectra would violate
  the encoding assumptions.
- When the target model needs sub-0.1 m/z mass accuracy for fragment matching — the
  fixed decimal_accuracy 1 truncation discards that resolution by design.
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3474
tools:
- name: LipiDetective
  role: Framework whose transformer input-embedding implements m/z decimal truncation
    and ordering-based intensity encoding
  repo: https://github.com/LipiTUM/lipidetective
- name: PyTorch
  role: Transformer module modified by LipiDetective; peak embeddings are fed through
    its encoder layers
- name: h5py
  role: Used to load the HDF5 training/prediction files whose /all_datasets/ group
    holds the 2xN peak arrays being encoded
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1101/2024.10.07.617094
    title: LipiDetective a deep learning model for the identification of molecular
      lipid species in tandem mass spectra
  source_content_hash: sha256:f7f06240e06f8dc8bda29e5266d89e908fded85ad9a9089cddb42545f8005dd8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/LipiTUM/lipidetective
  tools:
  - LipiDetective
  - PyTorch
  - h5py
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# mz-decimal-truncation-binning

## Summary

Encodes MS2 fragment peaks for LipiDetective's transformer by truncating each peak's m/z to a fixed decimal precision (1 decimal place) and mapping the discretized value to an embedding vector, with peak order (not intensity) conveying relative abundance. This bounds the input feature space to discrete m/z bins up to max_mz 1600.

## When to use

Use when building or reproducing LipiDetective's transformer input encoding: MS2 peak lists (m/z + intensity) from base-peak-normalized spectra must be converted to embedding tokens with m/z precision fixed by config decimal_accuracy.

## When NOT to use

- When the downstream task requires actual fragment intensities as numerical features — LipiDetective's encoder only receives intensity information implicitly via peak ordering by descending intensity.
- When spectra have not been preprocessed (smoothing, baseline correction, peak picking, base-peak normalization, trimming to 50-1600 m/z) — raw profile spectra would violate the encoding assumptions.
- When the target model needs sub-0.1 m/z mass accuracy for fragment matching — the fixed decimal_accuracy 1 truncation discards that resolution by design.

## Inputs

- Base-peak-normalized, peak-picked MS2 spectra trimmed to the 50-1600 m/z range (2xN peak arrays from the /all_datasets/ group of the HDF5 training file)
- Model configuration YAML (config/config_templates/config_transformer.yaml) with input_embedding settings (n_peaks 30, max_mz 1600, decimal_accuracy 1) and transformer dimensions
- PyTorch transformer module at src/lipidetective/models/transformer_network.py

## Outputs

- Per-spectrum tensor of up to 30 d_model-dimensional (d_model=32) peak embeddings with positional encodings added, ordered by descending intensity
- Discrete m/z token sequence (truncated to 1 decimal place) suitable for the transformer encoder
- Predicted lipid token sequence of at most output_seq_length (11) tokens from the decoder

## How to apply

Read the model configuration from config/config_templates/config_transformer.yaml to obtain input_embedding settings: n_peaks 30, max_mz 1600, decimal_accuracy 1, and transformer dimensions (d_model 32, num_heads 4, dropout 0.1, ffn_hidden 256, num_layers 2, output_seq_length 11). Retain at most the 30 most intense peaks from spectra already trimmed to the 50-1600 m/z range and base-peak normalized. Truncate each retained peak's m/z value to the user-specified decimal point (1 decimal place) and map it to its corresponding embedding vector of dimension d_model (kept divisible by num_heads); do NOT feed intensity values into the model — peaks are simply ordered by descending intensity. Add a positional encoding to each peak embedding so the transformer receives the relative intensity rank of each peak. Because intensity is dropped, the fixed decimal truncation is what limits the input feature space: each m/z maps to one discrete token, so precision loss at 1 decimal place is a deliberate vocabulary-binning trade-off. Wire the embedded peaks through the encoder of the PyTorch transformer module (src/lipidetective/models/transformer_network.py) alongside decoder-side lipid-name tokens produced by the label tokenization.

## Related tools

- **LipiDetective** (Framework whose transformer input-embedding implements m/z decimal truncation and ordering-based intensity encoding) — https://github.com/LipiTUM/lipidetective
- **PyTorch** (Transformer module modified by LipiDetective; peak embeddings are fed through its encoder layers)
- **h5py** (Used to load the HDF5 training/prediction files whose /all_datasets/ group holds the 2xN peak arrays being encoded)

## Examples

```
uv run lipidetective --config config/config_templates/config_transformer.yaml
```

## Evaluation signals

- Config check: config_transformer.yaml input_embedding must report n_peaks 30, max_mz 1600, decimal_accuracy 1; d_model must remain divisible by num_heads (32 / 4).
- Every encoded m/z token, after truncation to 1 decimal place, must lie within the trimmed 50-1600 m/z window.
- The encoded peak sequence must contain at most n_peaks = 30 tokens, sorted by descending intensity.
- A forward pass through src/lipidetective/models/transformer_network.py on a sample HDF5 spectrum must return a predicted lipid token sequence of at most output_seq_length (11) tokens.
- The module imports without error and produces output for a sample spectrum from the /all_datasets/ group (2xN array with lipid_species/adduct/precursor attributes).

## Limitations

- The model never receives actual peak intensity values; the only intensity signal is the descending-intensity ordering of the input m/z vector plus positional encoding.
- Fixed decimal truncation (decimal_accuracy 1) constrains the input feature space to discrete m/z bins, discarding sub-0.1 m/z resolution.
- Peaks relevant for prediction (e.g. 224.0 m/z) may lack annotation in the Alex123 database, so encoded tokens are not always interpretable against known fragments.

## Evidence

- [methods] The model never receives actual peak intensity values; the only indication of intensities is the ordering of the input vector of m/z values by descending intensity.: [evidence span withheld — anchor status: not_found]
- [methods] all MS2 spectra were trimmed to an m/z range of 50 - 1600 m/z: "all MS2 spectra were trimmed to an m/z range of 50 - 1600 m/z"
- [methods] LipiDetective was implemented by modifying the transformer module provided by PyTorch [26]: "LipiDetective was implemented by modifying the transformer module provided by PyTorch"
- [methods] all spectra were saved in a Hierarchical Data Format 5 (HDF5) file: "all spectra were saved in a Hierarchical Data Format 5 (HDF5) file"
- [readme] Accepts YAML configuration file for streamlined and reproducible setup.: [evidence span withheld — anchor status: external_doc]
