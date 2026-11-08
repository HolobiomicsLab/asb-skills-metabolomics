---
name: pytorch-transformer-architecture-implementation
description: Use when building or modifying the PyTorch transformer that maps MS2
  peak lists (m/z vectors ordered by descending intensity) to lipid species token
  sequences, or when reproducing LipiDetective's input embedding and label tokenization.
when_to_use_negative:
- When only mzML prediction files without a trained model or config are available
  — the skill covers network construction, not preprocessing or inference plumbing
- When the task is spectrum matching against databases (e.g. Alex123, GNPS) rather
  than deep-learning-based lipid annotation
- When raw profile spectra are the input — spectra must first undergo smoothing, baseline
  correction, peak picking, base peak normalization, and trimming to 50–1600 m/z
edam_operation: http://edamontology.org/operation_3936
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_3520
tools:
- name: PyTorch
  role: Transformer module modified to implement LipiDetective's encoder/decoder,
    peak embeddings, and label token embeddings
- name: PyTorch Lightning
  role: Multi-GPU training of the implemented transformer network
- name: h5py
  role: Reads HDF5 training spectra (/all_datasets/ group) fed through the network
    for forward-pass validation
- name: LipiDetective
  role: Repository containing the transformer_network.py module, YAML config templates,
    and CLI entry point
  repo: https://github.com/LipiTUM/lipidetective
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1101/2024.10.07.617094
    title: LipiDetective a deep learning model for the identification of molecular
      lipid species in tandem mass spectra
  source_content_hash: sha256:14146390668b8327c4f11771377f4e47c1a3e26db94ccefc6c13ff27a2dd61d6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/LipiTUM/lipidetective
  tools:
  - LipiDetective
  - PyTorch
  - PyTorch Lightning
  - h5py
  edam_operation: http://edamontology.org/operation_3936
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# pytorch-transformer-architecture-implementation

## Summary

Implements LipiDetective's transformer encoder/decoder in PyTorch: m/z values of the 30 most intense peaks are embedded (truncated to 1 decimal place, max m/z 1600) with positional encodings that encode intensity rank, and lipid species names are tokenized for autoregressive decoding into lipid shorthand tokens.

## When to use

Use when building or modifying the PyTorch transformer that maps MS2 peak lists (m/z vectors ordered by descending intensity) to lipid species token sequences, or when reproducing LipiDetective's input embedding and label tokenization.

## When NOT to use

- When only mzML prediction files without a trained model or config are available — the skill covers network construction, not preprocessing or inference plumbing
- When the task is spectrum matching against databases (e.g. Alex123, GNPS) rather than deep-learning-based lipid annotation
- When raw profile spectra are the input — spectra must first undergo smoothing, baseline correction, peak picking, base peak normalization, and trimming to 50–1600 m/z

## Inputs

- YAML configuration file (config/config_templates/config_transformer.yaml) with input_embedding and transformer hyperparameters
- MS2 peak lists from HDF5 training files (/all_datasets/ group, 2xN arrays of m/z and intensity with lipid_species/adduct/precursor attributes)
- Lipid species labels in updated lipid shorthand notation (headgroup, fatty acid side chains, adduct)

## Outputs

- Transformer network module (src/lipidetective/models/transformer_network.py) implementing encoder/decoder with peak and token embeddings
- Predicted lipid species token sequence (≤ output_seq_length = 11 tokens) per input spectrum

## How to apply

Read input_embedding and transformer settings from config/config_templates/config_transformer.yaml: n_peaks 30, max_mz 1600, decimal_accuracy 1, d_model 32 (kept divisible by num_heads=4), dropout 0.1, ffn_hidden 256, num_layers 2, output_seq_length 11. For each retained peak, truncate the m/z value to decimal_accuracy (1 decimal point) and map it to a d_model-dimensional embedding, then add positional encodings — since peaks are sorted by descending intensity and raw intensities are never fed to the model, the positional encoding is the only intensity signal. Tokenize the label side (headgroup, fatty acid side chains, adduct) with regular expressions and embed each token for the decoder. Wire embedded peaks and tokens through PyTorch's transformer encoder/decoder layers so the decoder emits at most output_seq_length (11) tokens; verify with a forward pass on a sample spectrum from the HDF5 dataset (/all_datasets/ group, 2xN arrays with lipid_species/adduct/precursor attributes).

## Related tools

- **PyTorch** (Transformer module modified to implement LipiDetective's encoder/decoder, peak embeddings, and label token embeddings)
- **PyTorch Lightning** (Multi-GPU training of the implemented transformer network)
- **h5py** (Reads HDF5 training spectra (/all_datasets/ group) fed through the network for forward-pass validation)
- **LipiDetective** (Repository containing the transformer_network.py module, YAML config templates, and CLI entry point) — https://github.com/LipiTUM/lipidetective

## Examples

```
uv run lipidetective --config path/to/config.yaml
```

## Evaluation signals

- Module file src/lipidetective/models/transformer_network.py exists and imports without error
- Forward pass on a sample HDF5 spectrum returns a predicted lipid token sequence of at most output_seq_length (11) tokens
- Peak m/z values are truncated to decimal_accuracy 1 and no peak exceeds max_mz 1600; d_model (32) is divisible by num_heads (4)
- Input vector contains only m/z-derived embeddings — intensity values enter only via descending-intensity ordering reflected in positional encodings
- Blank-measurement spectra (2002 added during training) cause the model to emit the end-of-sequence token rather than a lipid species

## Limitations

- The model never receives actual peak intensity values; intensity information is limited to the ordering of m/z values by descending intensity
- Fixed m/z precision (decimal_accuracy 1) constrains the input feature space; peaks above max_mz 1600 are excluded by upstream trimming to 50–1600 m/z
- Some prediction-relevant peaks (e.g. 224.0 m/z) lack annotation in the Alex123 database, complicating interpretation of learned attention
- Training data was relatively small (268,720 spectra), relying on generalization of learned fragmentation patterns to unseen lipid species

## Evidence

- [methods] LipiDetective was implemented by modifying the transformer module provided by PyTorch [26]: "LipiDetective was implemented by modifying the transformer module provided by PyTorch [26]"
- [methods] PyTorch Lightning used for multi-GPU training: "PyTorch Lightning [27] was used for multi-GPU training"
- [methods] Base peak normalization and m/z trimming before model input: "All profile spectra underwent smoothing, baseline correction, and peak picking"
- [readme] Repository README describing configurable input and HDF5 handling: "**Configurable Input:** Accepts YAML configuration file for streamlined and reproducible setup.
- **Data Handling:** Supports training with HDF5 files and prediction using mzML files."
- [abstract] Model can generalize to unseen lipid species: "The model can even correctly predict lipid species for which it has never seen a spectrum before as it is able to generalize the learned lipid fragmentation patterns"
