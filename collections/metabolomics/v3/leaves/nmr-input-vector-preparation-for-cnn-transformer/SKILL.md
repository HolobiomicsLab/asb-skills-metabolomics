---
name: nmr-input-vector-preparation-for-cnn-transformer
description: Use when raw 1H NMR spectra (ppm, intensity pairs) and/or 13C chemical-shift
  lists must be turned into the fixed-length concatenated input vectors expected by
  the NMR2Struct spectrum-to-structure model before training or single-spectrum inference.
when_to_use_negative:
- The spectra are already preprocessed hdf5 'spectra' vectors (28000-point 1H / 80-bin
  13C) ready for nmr_train or nmr_infer — re-preprocessing would double-transform
  the data
- The goal is peak picking, phasing, or baseline correction beyond the paper's deliberately
  minimal preprocessing — this skill only interpolates, normalizes, and bins
- Input is 2D NMR data or spectra for molecules outside the model's scope (C, N, O
  heavy atoms, up to 19 heavy atoms), which this pipeline was not built to handle
edam_topics:
- http://edamontology.org/topic_0593
- http://edamontology.org/topic_3474
tools:
- name: NMR2Struct
  role: Provides the preprocessing configuration, the nmr_infer_single_spectrum entry
    point with the --normalize flag, and the hdf5 spectra file format into which the
    concatenated vectors are written
  repo: https://github.com/MarklandGroup/NMR2Struct
- name: Anaconda
  role: Recommended package manager for installing the NMR2Struct environment (environment.yml,
    NMR_env) in which the preprocessing and inference code runs
  repo: https://www.anaconda.com/download
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1021/acscentsci.4c01132
    title: Accurate and Efficient Structure Elucidation from Routine One-Dimensional
      NMR Spectra Using Multitask Machine Learning
  source_content_hash: sha256:b9e3aae4a3309d122d864e93a24b2557c9ea1f2eb4a3480c4460622eb309c198
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/MarklandGroup/NMR2Struct
  tools:
  - Anaconda
  - NMR2Struct
  - transformer
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0593
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# nmr-input-vector-preparation-for-cnn-transformer

## Summary

Converts raw 1D 1H and/or 13C NMR spectra into the fixed-length numerical vectors consumed by the NMR2Struct multitask CNN/transformer: 1H spectra are interpolated onto a 28000-point grid (−2 to 12 ppm, 0.0005 ppm resolution) and max-normalized, 13C shifts are binned into 80 bins spanning 3.42–231.3 ppm, and the vectors are concatenated into the single input representation.

## When to use

Use when raw 1H NMR spectra (ppm, intensity pairs) and/or 13C chemical-shift lists must be turned into the fixed-length concatenated input vectors expected by the NMR2Struct spectrum-to-structure model before training or single-spectrum inference.

## When NOT to use

- The spectra are already preprocessed hdf5 'spectra' vectors (28000-point 1H / 80-bin 13C) ready for nmr_train or nmr_infer — re-preprocessing would double-transform the data
- The goal is peak picking, phasing, or baseline correction beyond the paper's deliberately minimal preprocessing — this skill only interpolates, normalizes, and bins
- Input is 2D NMR data or spectra for molecules outside the model's scope (C, N, O heavy atoms, up to 19 heavy atoms), which this pipeline was not built to handle

## Inputs

- Raw 1H NMR spectrum as a text/TSV file of (ppm, intensity) pairs, one per row, space separated
- Raw 13C NMR chemical shifts as a text file with a comma-separated list of carbon ppm values
- Shift-grid pickle file (1D numpy array) for single-spectrum inference
- NMR2Struct YAML configuration file with at least global_args, model, and inference sections

## Outputs

- Preprocessed 1H vector: 28000 values on the −2 to 12 ppm grid at 0.0005 ppm resolution, intensities normalized to 0–1
- Preprocessed 13C vector: 80-bin binary vector spanning 3.42 to 231.3 ppm
- Single concatenated 1H+13C input vector (source_size 28045) per molecule, written into an hdf5 'spectra' dataset for training/inference

## How to apply

Load the raw 1H spectrum as space-separated (ppm, intensity) rows and/or the 13C spectrum as a comma-separated list of carbon ppm shifts (pass 'None' for either if absent). Linearly interpolate the 32768-value 1H spectrum onto a grid of 28000 values covering −2 to 12 ppm at 0.0005 ppm resolution — this retains splittings and rapidly varying features rather than heavy downsampling — then normalize by dividing by the highest intensity (the '--normalize' flag toggles this). Extract the 13C chemical shifts and bin them into 80 bins spanning 3.42 to 231.3 ppm. Concatenate the processed 1H and 13C vectors into the single input representation; for training over full datasets, the spectra hdf5 file must contain a 'spectra' dataset whose rows are these concatenated vectors, matching model.model_args.structure_model_args.source_size = 28045.

## Related tools

- **NMR2Struct** (Provides the preprocessing configuration, the nmr_infer_single_spectrum entry point with the --normalize flag, and the hdf5 spectra file format into which the concatenated vectors are written) — https://github.com/MarklandGroup/NMR2Struct
- **Anaconda** (Recommended package manager for installing the NMR2Struct environment (environment.yml, NMR_env) in which the preprocessing and inference code runs) — https://www.anaconda.com/download

## Examples

```
nmr_infer_single_spectrum --config config.yaml --hnmr_file hnmr.tsv --cnmr_file cnmr.txt --hnmr_shifts hgrid.p --cnmr_shifts cgrid.p --ckpt ckpt.pt --normalize
```

## Evaluation signals

- Output 1H vector has length 28000 on the −2 to 12 ppm grid (0.0005 ppm resolution)
- Output 13C vector is an 80-bin binary vector spanning 3.42 to 231.3 ppm
- 1H intensities are normalized to the 0–1 range after division by the highest intensity
- Concatenated per-spectrum vector matches the multitask model's structure_model_args.source_size of 28045 and, for dataset-wide use, rows of the hdf5 'spectra' dataset
- The spectra_file, label_file, and smiles_file lists in the YAML config have the same number of elements with element-wise correspondence

## Limitations

- The model was trained on MestreNova-simulated spectra that do not fully capture the nuances of real spectra; acquiring large experimental datasets in the required form (original FID files) remains a challenge
- Interpolation is strictly linear onto the 28000-point grid — no peak deconvolution or other spectral enhancement is performed, by design (minimal preprocessing)
- Preprocessing is coupled to the multitask model's fixed grid and bin definitions; the single-spectrum entry point only works for inferring from spectra with a multitask model

## Evidence

- [results] Linearly interpolated 1H NMR spectra from 32768 values down to 28000 values (−2 to 12 ppm, 0.0005 ppm resolution) and normalized intensities to 0–1: "linearly interpolating the spectrum of 32768 values down to a grid of 28000 values corresponding to a shift range of −2 to 12 ppm with a resolution of 0.0005 ppm."
- [results] Extracted 13C chemical shifts and binned them into 80 bins spanning 3.42 to 231.3 ppm: "we processed the spectra by extracting the chemical shifts and binning them into 80 bins spanning the shift range of 3.42 to 231.3 ppm."
- [methods] normalize (1H NMR spectrum normalization by highest intensity): [evidence span withheld — anchor status: not_found]
- [readme] README: spectra hdf5 format for concatenated 1H/13C input vectors: "These should be hdf5 files that contain one dataset called 'spectra' where each row is a one-dimensional vector representing the concatenated <sup>1</sup>H NMR and <sup>13</sup>C NMR spectrum."
- [readme] README: single-spectrum inference entry point with normalize flag: "nmr_infer_single_spectrum \
    --config YAML_CONFIG_FILE \
    --hnmr_file HNMR_FILE \
    --cnmr_file CNMR_FILE \
    --hnmr_shifts HNMR_SHIFTS \
    --cnmr_shifts CNMR_SHIFTS \
    --ckpt CKPT \"
