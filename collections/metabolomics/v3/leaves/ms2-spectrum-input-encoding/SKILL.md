---
name: ms2-spectrum-input-encoding
description: Use when raw or profile MS2 spectra (mzML for prediction, HDF5 for training)
  must be converted into peak-embedding plus positional-encoding input sequences for
  a transformer that predicts lipid species tokens.
when_to_use_negative:
- The task is decoding transformer output into lipid species strings rather than preparing
  model input
- Identification should rely on database/spectrum matching (e.g. LipiDex, Alex123,
  Compound Discoverer 3.1) instead of neural encoding
- Spectra are already smoothed, peak-picked, normalized and stored as HDF5 model-ready
  sequences
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0092
tools:
- name: LipiDetective
  role: Transformer model whose encoder consumes the peak-embedding + positional-encoding
    input sequences built from preprocessed MS2 spectra
  repo: https://github.com/LipiTUM/lipidetective
- name: pyOpenMS
  role: Library used for preprocessing profile spectra (smoothing, baseline correction,
    peak picking) before encoding
- name: PyTorch
  role: Provides the transformer module that LipiDetective modifies; peak embeddings
    and positional encodings are PyTorch tensors
- name: h5py
  role: Used to store the preprocessed spectra in HDF5 files for training
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1101/2024.10.07.617094
    title: LipiDetective a deep learning model for the identification of molecular
      lipid species in tandem mass spectra
  source_content_hash: sha256:5cdf2ed1b79a882e7928246766f751786bbf10712bac334373f844023eeb1261
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/LipiTUM/lipidetective
  tools:
  - LipiDetective
  - PyTorch
  - h5py
  - pyOpenMS
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# ms2-spectrum-input-encoding

## Summary

Encodes preprocessed tandem MS2 lipid spectra into transformer-ready token sequences for LipiDetective: peaks are picked, base-peak normalized, trimmed to 50–1600 m/z, restricted to the top n_peaks m/z values ordered by decreasing intensity, and embedded with positional encodings before being passed to the transformer.

## When to use

Use when raw or profile MS2 spectra (mzML for prediction, HDF5 for training) must be converted into peak-embedding plus positional-encoding input sequences for a transformer that predicts lipid species tokens.

## When NOT to use

- The task is decoding transformer output into lipid species strings rather than preparing model input
- Identification should rely on database/spectrum matching (e.g. LipiDex, Alex123, Compound Discoverer 3.1) instead of neural encoding
- Spectra are already smoothed, peak-picked, normalized and stored as HDF5 model-ready sequences

## Inputs

- Profile MS2 spectra from lipid standards or biological samples (mzML files for prediction)
- HDF5 files containing preprocessed spectra for training
- YAML configuration file specifying n_peaks, d_model, num_heads and file paths
- Pre-trained LipiDetective model weights (.pth) matching the encoding config

## Outputs

- Peak-embedding plus positional-encoding input sequences (top n_peaks m/z values by decreasing intensity)
- HDF5 files of preprocessed, base-peak normalized spectra (50–1600 m/z range)

## How to apply

First preprocess the profile spectra: smoothing, baseline correction, and peak picking (via pyOpenMS), then base peak normalization so intensities are relative to the base peak, and trim all MS2 spectra to an m/z range of 50–1600 m/z. Select the top n_peaks m/z values ordered by decreasing intensity as the model input sequence — this ordering and cutoff is the fixed configuration tied to the .pth checkpoint (d_model, num_heads, n_peaks must match training). Embed each selected peak and add positional encodings to form the encoder input sequence; spectra where no lipid is present are encoded so the model learns to emit the end-of-sequence token (blank measurements were used for this). Save intermediate spectra in HDF5 format for training and read mzML files for prediction. Verify by checking that every encoded sequence respects the m/z bounds, the intensity ordering, and the n_peaks length limit before feeding the transformer.

## Related tools

- **LipiDetective** (Transformer model whose encoder consumes the peak-embedding + positional-encoding input sequences built from preprocessed MS2 spectra) — https://github.com/LipiTUM/lipidetective
- **pyOpenMS** (Library used for preprocessing profile spectra (smoothing, baseline correction, peak picking) before encoding)
- **PyTorch** (Provides the transformer module that LipiDetective modifies; peak embeddings and positional encodings are PyTorch tensors)
- **h5py** (Used to store the preprocessed spectra in HDF5 files for training)

## Examples

```
uv run lipidetective --config path/to/config.yaml
```

## Evaluation signals

- All MS2 spectra are trimmed to the m/z range 50–1600 before encoding
- Encoded peak intensities are base-peak normalized and peaks are ordered by decreasing intensity
- Input sequence length does not exceed n_peaks and matches the config of the loaded .pth checkpoint (d_model, num_heads, n_peaks)
- Training spectra are stored in a valid HDF5 file readable by h5py
- Blank-measurement spectra encode to sequences whose model output is the end-of-sequence token (no lipid present)

## Limitations

- The encoding config (n_peaks, m/z range 50–1600, base peak normalization) is tied to the training setup; mismatched checkpoint/config pairs will invalidate predictions
- Peaks relevant for prediction (e.g. 224.0 m/z) may lack annotation in databases like Alex123, though the encoder still passes them to the model
- Noisy spectra must be excluded beforehand — spectra above a 0.01 base-peak normalized median intensity score were filtered out during training data creation

## Evidence

- [methods] All profile spectra underwent smoothing, baseline correction, and peak picking: "All profile spectra underwent smoothing, baseline correction, and peak picking"
- [methods] all MS2 spectra were trimmed to an m/z range of 50 - 1600 m/z: "all MS2 spectra were trimmed to an m/z range of 50 - 1600 m/z"
- [methods] base peak normalization was performed for all spectra: "base peak normalization was performed for all spectra"
- [methods] all spectra were saved in a Hierarchical Data Format 5 (HDF5) file: "all spectra were saved in a Hierarchical Data Format 5 (HDF5) file"
- [methods] LipiDetective was implemented by modifying the transformer module provided by PyTorch: "LipiDetective was implemented by modifying the transformer module provided by PyTorch [26]"
- [readme] Supports training with HDF5 files and prediction using mzML files: [evidence span withheld — anchor status: external_doc]
