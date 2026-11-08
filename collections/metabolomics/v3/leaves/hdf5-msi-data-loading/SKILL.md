---
name: hdf5-msi-data-loading
description: Use when raw MSI data is in imzML format exported from SCiLS lab and
  must be converted/loaded into HDF5 (via Python h5py) for pixel-wise classification
  or benchmarking on withheld test spectra.
when_to_use_negative:
- The analysis requires a reduced peak table or peak-picked features rather than full-resolution
  raw spectra, since massNet explicitly operates without prior preprocessing or peak
  picking
- The data is not in imzML/HDF5 form (e.g., already an annotated feature matrix or
  a different proprietary vendor format not exported via SCiLS lab)
- Multi-class tissue classification is needed; the demonstrated pipeline is binary
  normal/tumor classification only
edam_operation: http://edamontology.org/operation_3359
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3382
tools:
- name: SCiLS lab 2020a
  role: Source application from which MSI data is exported in the standardized imzML
    format prior to HDF5 conversion
- name: h5py
  role: Python library (pinned to 2.7.1 in the massNet README) for reading the converted
    HDF5 MSI files into spectral arrays
  repo: github.com/wabdelmoula/massNet
- name: Python
  role: Runtime environment (3.6.12 in the README) in which the HDF5 MSI data is loaded
    and passed to Keras/TensorFlow for massNet or to sklearn for SVM benchmarking
  repo: github.com/wabdelmoula/massNet
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btac032
    title: 'massNet: integrated processing and classification of spatially resolved
      mass spectrometry data using deep learning for rapid tumor delineation'
  source_content_hash: sha256:a4b5de20368d143337a17ba5cdd64561cf7d9570a4cb59631f6029fc2bf1278d
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
  edam_operation: http://edamontology.org/operation_3359
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3382
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# hdf5-msi-data-loading

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Loads MALDI FT-ICR mass spectrometry imaging data that has been exported from SCiLS lab as imzML and converted to HDF5, so spectra with tens of thousands of m/z variables can feed deep learning (massNet) or classical ML (SVM) classifiers directly.

## When to use

Use when raw MSI data is in imzML format exported from SCiLS lab and must be converted/loaded into HDF5 (via Python h5py) for pixel-wise classification or benchmarking on withheld test spectra.

## When NOT to use

- The analysis requires a reduced peak table or peak-picked features rather than full-resolution raw spectra, since massNet explicitly operates without prior preprocessing or peak picking
- The data is not in imzML/HDF5 form (e.g., already an annotated feature matrix or a different proprietary vendor format not exported via SCiLS lab)
- Multi-class tissue classification is needed; the demonstrated pipeline is binary normal/tumor classification only

## Inputs

- imzML file exported from SCiLS lab 2020a (Bruker) containing MALDI 9.4 Tesla FT-ICR MSI data
- Converted HDF5 (.h5) file with mass spectra (85,062 m/z variables) and pixel-wise normal/tumor labels

## Outputs

- In-memory spectral matrices (spectra × 85,062 m/z variables) for training (10,024 normal / 3,644 tumor) and test (9,798 normal / 2,861 tumor) splits
- Label vectors for pixel-wise normal/tumor classification used by massNet and SVM

## How to apply

Export the MSI data from SCiLS lab 2020a in the standardized imzML format, then convert it to HDF5 format for deep learning analysis, as the paper does for the 9.4 Tesla SolariX FT-ICR acquisitions at 100 µm spatial resolution. Load the HDF5 file in Python using h5py (the README pins h5py 2.7.1 alongside Python 3.6.12, numpy 1.15.4, sklearn 0.23.2, scipy 1.0.0, and pandas 1.1.1). Verify that each loaded dataset carries the full spectral dimensionality — 85,062 m/z variables in this study — and the pixel-wise class labels (e.g., the training split of 10,024 normal and 3,644 tumor spectra, and the withheld test split of 9,798 normal and 2,861 tumor spectra). Rationale: massNet consumes raw, TIC-normalized spectra without prior preprocessing or peak picking, so the loader must preserve the full-resolution spectral vectors rather than peak tables. The loaded arrays then serve both massNet training/inference and the SVM benchmark comparison.

## Related tools

- **SCiLS lab 2020a** (Source application from which MSI data is exported in the standardized imzML format prior to HDF5 conversion)
- **h5py** (Python library (pinned to 2.7.1 in the massNet README) for reading the converted HDF5 MSI files into spectral arrays) — github.com/wabdelmoula/massNet
- **Python** (Runtime environment (3.6.12 in the README) in which the HDF5 MSI data is loaded and passed to Keras/TensorFlow for massNet or to sklearn for SVM benchmarking) — github.com/wabdelmoula/massNet

## Examples

```
import h5py; f = h5py.File('gbm_msi.h5', 'r'); spectra = f['spectra'][:]; labels = f['labels'][:]  # 13,795 spectra x 85,062 m/z
```

## Evaluation signals

- Loaded HDF5 dataset exposes 85,062 m/z variables per spectrum, matching the reported spectral dimensionality
- Spectral counts match the published splits: training set 10,024 normal and 3,644 tumor spectra; test set 9,798 normal and 2,861 tumor spectra
- Spectra are TIC-normalized raw profiles (no peak picking applied), consistent with massNet's no-preprocessing design
- Pixel labels align with pathologist-annotated H&E tumor regions transferred onto the MSI pixel grid

## Limitations

- Ground truth labels derive solely from pathologist-annotated H&E histology, which the authors note is an approximation that may not capture intra-tumor heterogeneity
- The demonstrated pipeline supports binary normal/tumor classification only; multi-class extension is left to future development
- The README does not document the exact HDF5 schema or a conversion script, so the imzML-to-HDF5 conversion step must be reproduced separately

## Evidence

- [methods] acquisition parameters: [evidence span withheld — anchor status: not_found]
