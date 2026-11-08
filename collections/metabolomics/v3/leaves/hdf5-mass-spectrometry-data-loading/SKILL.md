---
name: hdf5-mass-spectrometry-data-loading
description: Use when raw MSI data has been exported from SCiLS lab as imzML and must
  be converted to HDF5 for deep learning analysis, or when a trained model requires
  loading labeled spectral matrices (normal/tumor) with tens of thousands of m/z variables.
when_to_use_negative:
- Data is already a peak-picked, feature-reduced table — massNet's design goal is
  classification directly from raw spectra without prior peak picking
- Data is not in imzML/HDF5 (e.g. raw instrument files) — conversion via SCiLS lab
  export must happen first
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3365
tools:
- name: Python
  role: Runtime environment (3.6.12) in which HDF5 files are read and arrays prepared
- name: h5py
  role: Python library used to read/write the HDF5-formatted MSI spectral data
- name: numpy
  role: Array representation of the loaded spectra and labels
- name: SCiLS lab 2020a
  role: Source tool from which MSI data is exported as imzML prior to HDF5 conversion
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btac032
    title: 'massNet: integrated processing and classification of spatially resolved
      mass spectrometry data using deep learning for rapid tumor delineation'
  source_content_hash: sha256:1a7013ee3fbe42509bac5657a0f92299ae3b4abec778582de31d1229cf71b685
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
  - h5py
  - numpy
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3365
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# hdf5-mass-spectrometry-data-loading

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Loads MALDI FT-ICR mass spectrometry imaging spectra (e.g. 85,062 m/z variables across ~10,000–13,000 spectra per split) stored in HDF5 format, converted from imzML, so the raw spectral matrices can feed deep learning models like massNet without peak picking.

## When to use

Use when raw MSI data has been exported from SCiLS lab as imzML and must be converted to HDF5 for deep learning analysis, or when a trained model requires loading labeled spectral matrices (normal/tumor) with tens of thousands of m/z variables.

## When NOT to use

- Data is already a peak-picked, feature-reduced table — massNet's design goal is classification directly from raw spectra without prior peak picking
- Data is not in imzML/HDF5 (e.g. raw instrument files) — conversion via SCiLS lab export must happen first

## Inputs

- imzML file exported from SCiLS lab 2020a (Bruker)
- HDF5 file of MALDI FT-ICR MSI spectra with 85,062 m/z variables
- Ground-truth normal/tumor labels transferred from pathologist-annotated H&E images

## Outputs

- In-memory spectral matrix and label arrays ready for Keras/TensorFlow model input
- TIC-normalized spectra suitable for VAE reconstruction and classification

## How to apply

Export the MALDI FT-ICR MSI data from SCiLS lab 2020a in the standardized imzML format and convert it to HDF5 using h5py, as the massNet pipeline does for its 85,062 m/z variable GBM PDX dataset. In Python (3.6.12) with h5py (2.7.1), read the spectral matrix and ground-truth labels into numpy (1.15.4) arrays, preserving the class split (e.g. test set: 9,798 normal and 2,861 tumor spectra). Optionally TIC-normalize the spectra, since massNet's VAE reconstruction is validated against 'the original TIC-normalized MSI data'. No preprocessing or peak picking should be applied — the architecture consumes raw high-dimensional spectra directly. Verify dimensions before handing off: each spectrum is a vector of 85,062 m/z variables and total counts must match the annotated spectrum counts per class.

## Related tools

- **Python** (Runtime environment (3.6.12) in which HDF5 files are read and arrays prepared)
- **h5py** (Python library used to read/write the HDF5-formatted MSI spectral data)
- **numpy** (Array representation of the loaded spectra and labels)
- **SCiLS lab 2020a** (Source tool from which MSI data is exported as imzML prior to HDF5 conversion)

## Examples

```
import h5py; with h5py.File('msi_test.h5','r') as f: spectra = f['spectra'][:]; labels = f['labels'][:]  # 12,659 spectra x 85,062 m/z variables
```

## Evaluation signals

- Each loaded spectrum contains exactly 85,062 m/z variables as in the published dataset
- Class counts match the published splits (test set: 9,798 normal and 2,861 tumor spectra; training set: 10,024 normal and 3,644 tumor spectra)
- No peak picking or dimensionality reduction was applied downstream of the HDF5 load before model input
- Spectra can be reconstructed with total mean squared error on the order of 5.69 × 10^-8 after TIC normalization, as the optimized massNet VAE achieved

## Limitations

- The pipeline was demonstrated only on binary (normal/tumor) classification of GBM PDX MALDI FT-ICR MSI data; multi-class use is untested
- Ground-truth labels are pathologist annotations of consecutive H&E sections manually transferred to the MSI data — an approximation that may not capture intra-tumor heterogeneity
- The README specifies exact library versions (e.g. h5py 2.7.1, Tensorflow 1.8.0, Keras 2.2.0); newer versions are unverified

## Evidence
