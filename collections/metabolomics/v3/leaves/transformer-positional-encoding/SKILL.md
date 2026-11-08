---
name: transformer-positional-encoding
description: 'Use when reimplementing or extending LipiDetective''s transformer input:
  spectra are stored as 2xN m/z/intensity arrays in HDF5, peaks are already ordered
  by descending intensity, and the model must derive intensity information from position
  rather than values.'
when_to_use_negative:
- Input is raw profile data — smoothing, baseline correction, peak picking, base peak
  normalization, and trimming to 50-1600 m/z must happen first
- The task requires explicit intensity values as model features — LipiDetective's
  design conveys intensity only via descending-intensity ordering
- Predicting on spectra with more than 30 peaks or m/z outside 50-1600 without retraining
  or re-mapping the embedding vocabulary
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: PyTorch
  role: Provides the transformer module whose encoder/decoder layers are modified
    to host the peak and token embeddings plus positional encodings
- name: LipiDetective
  role: 'Reference implementation: YAML-configured deep learning framework identifying
    lipid species in shorthand nomenclature from HDF5 training data and mzML prediction
    files'
  repo: https://github.com/LipiTUM/lipidetective
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1101/2024.10.07.617094
    title: LipiDetective a deep learning model for the identification of molecular
      lipid species in tandem mass spectra
  source_content_hash: sha256:7239566ba8ec021ea427103f9a05c575bc6feeb598a025d416cedcf965d37dfd
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/LipiTUM/lipidetective
  tools:
  - LipiDetective
  - PyTorch
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# transformer-positional-encoding

## Summary

Encodes MS2 peak m/z values as discrete embeddings (m/z truncated to fixed decimal precision) and adds positional encodings whose positions carry the descending-intensity rank, since LipiDetective feeds no explicit intensity values to the transformer.

## When to use

Use when reimplementing or extending LipiDetective's transformer input: spectra are stored as 2xN m/z/intensity arrays in HDF5, peaks are already ordered by descending intensity, and the model must derive intensity information from position rather than values.

## When NOT to use

- Input is raw profile data — smoothing, baseline correction, peak picking, base peak normalization, and trimming to 50-1600 m/z must happen first
- The task requires explicit intensity values as model features — LipiDetective's design conveys intensity only via descending-intensity ordering
- Predicting on spectra with more than 30 peaks or m/z outside 50-1600 without retraining or re-mapping the embedding vocabulary

## Inputs

- HDF5 file with /all_datasets/ group containing 2xN arrays of m/z/intensity pairs and lipid_species/adduct/precursor attributes
- YAML configuration file (config/config_templates/config_transformer.yaml template) with input_embedding and transformer settings
- Lipid species names in shorthand nomenclature for decoder-side tokenization

## Outputs

- Peak embedding matrix of up to 30 d_model-dimensional vectors with positional encodings added
- Tokenized lipid label sequence fed to the transformer decoder
- Predicted lipid token sequence of at most output_seq_length 11 tokens per spectrum

## How to apply

Read input_embedding settings from config/config_templates/config_transformer.yaml (n_peaks 30, max_mz 1600, decimal_accuracy 1) and transformer dimensions (d_model 32, num_heads 4, ffn_hidden 256, num_layers 2, output_seq_length 11). Truncate each retained peak's m/z to 1 decimal place and map the truncated value to an embedding vector of dimension d_model, kept divisible by num_heads; keep only the top 30 peaks and treat max_mz 1600 as the vocabulary ceiling (spectra are pre-trimmed to 50-1600 m/z and base-peak normalized upstream). Add a positional encoding to each peak embedding: because peaks are sorted by descending intensity, position encodes relative intensity rank, so the model never receives actual intensity values. On the label side, tokenize lipid species names (headgroup, fatty acid side chains, adduct) with regular expressions and embed each token for the decoder. Wire embedded peaks and tokens through PyTorch's transformer module (the basis LipiDetective was built on) and verify with a forward pass on a sample HDF5 spectrum from the /all_datasets/ group.

## Related tools

- **PyTorch** (Provides the transformer module whose encoder/decoder layers are modified to host the peak and token embeddings plus positional encodings)
- **LipiDetective** (Reference implementation: YAML-configured deep learning framework identifying lipid species in shorthand nomenclature from HDF5 training data and mzML prediction files) — https://github.com/LipiTUM/lipidetective

## Examples

```
uv run lipidetective --config config/config_templates/config_transformer.yaml
```

## Evaluation signals

- src/lipidetective/models/transformer_network.py imports without error
- A forward pass on a sample HDF5 spectrum from /all_datasets/ returns a predicted lipid token sequence of at most 11 tokens (output_seq_length)
- Every input m/z is truncated to decimal_accuracy 1 before embedding lookup; no raw intensity value is passed into the model
- d_model remains divisible by num_heads (32/4) after any reconfiguration
- Peaks in the input vector are ordered by descending intensity so positional encoding corresponds to intensity rank

## Limitations

- Intensity information is lossy: only relative rank is preserved via position ordering, never absolute or normalized intensity values
- The m/z embedding vocabulary is bounded by max_mz 1600 and 1-decimal precision, so m/z values outside the trimmed 50-1600 range or with finer precision cannot be encoded
- Only the top 30 peaks (n_peaks 30) are represented; relevant low-intensity peaks beyond this cutoff are dropped
- Peaks relevant for prediction, such as 224.0 m/z, lack annotation in the Alex123 database, complicating interpretability checks

## Evidence

- [methods] The model never receives actual peak intensity values; the only indication of intensities is the ordering of the input vector of m/z values by descending intensity.: [evidence span withheld — anchor status: not_found]
- [methods] LipiDetective was implemented by modifying the transformer module provided by PyTorch [26]: "LipiDetective was implemented by modifying the transformer module provided by PyTorch [26]"
- [methods] all MS2 spectra were trimmed to an m/z range of 50 - 1600 m/z: "all MS2 spectra were trimmed to an m/z range of 50 - 1600 m/z"
- [methods] base peak normalization was performed for all spectra: "base peak normalization was performed for all spectra"
- [readme] Utilizes the transformer architecture to identify lipid species in shorthand nomenclature format.: [evidence span withheld — anchor status: external_doc]
