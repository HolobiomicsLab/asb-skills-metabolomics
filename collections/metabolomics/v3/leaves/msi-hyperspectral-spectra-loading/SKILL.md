---
name: msi-hyperspectral-spectra-loading
description: Use when pixel-level mass spectra must be brought from imzML (exported
  from SCiLS lab) or Metabolomics Workbench into HDF5 and TIC-normalized, without
  peak picking, as input to a deep learning classifier.
when_to_use_negative:
- Downstream analysis requires peak-picked or reduced feature tables — this skill
  deliberately retains the full 85,062-variable spectrum with no preprocessing or
  peak picking
- Data are not pixel-level mass spectra (e.g. bulk LC-MS runs or H&E histology images
  alone)
- Only visualization/exploratory dimensionality reduction is needed (the paper uses
  UMAP for that, not this loading path)
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3174
tools:
- name: massNet
  role: Deep learning pipeline that consumes the loaded TIC-normalized HDF5 spectra
    directly without peak picking
  repo: github.com/wabdelmoula/massNet
- name: Python
  role: Implementation language for loading HDF5 data and running the massNet model
  repo: github.com/wabdelmoula/massNet
- name: SCiLS lab 2020a
  role: Source of the imzML export that is converted to HDF5 for deep learning analysis
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btac032
    title: 'massNet: integrated processing and classification of spatially resolved
      mass spectrometry data using deep learning for rapid tumor delineation'
  source_content_hash: sha256:36e22a4a59e28ae7c16e8781a62d6ad21ac20990408c836342db97516a70d79f
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/wabdelmoula/massNet
  tools:
  - Python
  - SCiLS lab 2020a
  - massNet
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9545454545454546
n_claims: 22
---

# msi-hyperspectral-spectra-loading

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Loads raw mass spectrometry imaging (MSI) spectral data — here MALDI 9.4T FT-ICR MSI with 85,062 m/z variables per pixel — into a deep-learning-ready representation (TIC-normalized spectra in HDF5) for models like massNet that operate on full, un-preprocessed spectra.

## When to use

Use when pixel-level mass spectra must be brought from imzML (exported from SCiLS lab) or Metabolomics Workbench into HDF5 and TIC-normalized, without peak picking, as input to a deep learning classifier.

## When NOT to use

- Downstream analysis requires peak-picked or reduced feature tables — this skill deliberately retains the full 85,062-variable spectrum with no preprocessing or peak picking
- Data are not pixel-level mass spectra (e.g. bulk LC-MS runs or H&E histology images alone)
- Only visualization/exploratory dimensionality reduction is needed (the paper uses UMAP for that, not this loading path)

## Inputs

- imzML file exported from SCiLS lab 2020a (Bruker)
- TIC-normalized MSI spectra in HDF5 format (85,062 m/z variables; 10,024 normal and 3,644 tumor training spectra)
- Pathologist-derived pixel annotations transferred from H&E images to the MSI data
- Metabolomics Workbench project id PR001292 dataset

## Outputs

- HDF5 spectral matrix (pixels × 85,062 m/z variables) ready for deep learning analysis
- Label vector aligning each spectrum to its normal/tumor pixel annotation

## How to apply

Export the MSI data from SCiLS lab 2020a in the standardized imzML format and convert it to HDF5 for deep learning analysis, or download the deposited TIC-normalized HDF5 directly from Metabolomics Workbench under project id PR001292. Keep the full spectrum per pixel — 85,062 m/z variables here — rather than reducing to peaks, since massNet classifies directly from raw mass spectral data without prior preprocessing or peak picking. Load with Python's h5py (README pins h5py 2.7.1, numpy 1.15.4, Python 3.6.12, Keras 2.2.0 on TensorFlow 1.8.0) and preserve the accompanying pathologist-derived pixel annotations (normal vs. tumor) so spectra stay paired with labels. The training split carries 10,024 normal and 3,644 tumor spectra; the test set carries 9,798 normal and 2,641-class spectra per the paper's totals, all at 100 µm spatial resolution from a 9.4 Tesla SolariX instrument.

## Related tools

- **massNet** (Deep learning pipeline that consumes the loaded TIC-normalized HDF5 spectra directly without peak picking) — github.com/wabdelmoula/massNet
- **Python** (Implementation language for loading HDF5 data and running the massNet model) — github.com/wabdelmoula/massNet
- **SCiLS lab 2020a** (Source of the imzML export that is converted to HDF5 for deep learning analysis)

## Examples

```
import h5py; f = h5py.File('PR001292_msi.h5', 'r'); spectra = f['spectra'][:]; labels = f['annotations'][:]
```

## Evaluation signals

- Spectral matrix has the expected dimensionality: total spectra × 85,062 m/z variables per pixel
- Class counts match the paper: training set 10,024 normal and 3,644 tumor spectra; test set 9,798 normal and 2,861 tumor spectra
- Spectra are TIC-normalized before feeding the VAE; subsequent reconstruction MSE should reach ~5.69 × 10^-8 on correctly loaded, normalized data
- Pixel labels remain paired with their spectra after conversion from imzML/HDF5 (no row misalignment between annotations and spectra)
- Load environment matches README pins: Python 3.6.12, h5py 2.7.1, Keras 2.2.0 with TensorFlow 1.8.0 backend

## Limitations

- The paper reports no software version numbers or hyperparameter values in the main text; only the README pins package versions, so reproducibility of the loading/training stack depends on the repository
- Ground-truth pixel labels derive solely from pathologist annotation of H&E images, an approximation that may not capture intra-tumor heterogeneity
- No trained model weights or availability statement for them are provided in the article text, so loaded data can be reproduced but downstream model artifacts may not

## Evidence
