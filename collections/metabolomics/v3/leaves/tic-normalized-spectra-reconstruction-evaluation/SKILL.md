---
name: tic-normalized-spectra-reconstruction-evaluation
description: Use when raw high-dimensional MSI spectra (imzML/HDF5, tens of thousands
  of m/z variables) need TIC-normalized reconstruction error quantification via a
  VAE, or when validating reported reconstruction MSE targets.
when_to_use_negative:
- Input is already a reduced feature/peak table or a peak-picked spectral matrix —
  the VAE operates on full TIC-normalized spectra without prior preprocessing or peak
  picking
- The analysis goal is multi-class tissue classification; the paper's demonstrated
  model only supports binary (normal vs tumor) classification
- Data lacks pixel-level histology annotations or cannot be TIC-normalized, so reconstruction
  error against the paper's setup would not be comparable
edam_operation: http://edamontology.org/operation_0286
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3474
tools:
- name: massNet
  role: Deep learning VAE module whose probabilistic encoder/decoder with a 5-dimensional
    Code layer is trained and evaluated for reconstruction of TIC-normalized MSI spectra
  repo: github.com/wabdelmoula/massNet
- name: Keras
  role: Platform used to implement the VAE architecture (probabilistic encoder/decoder)
    in Python
- name: tensorflow
  role: Backend for the Keras-implemented VAE during variational lower bound optimization
- name: Python
  role: Language in which the massNet architecture and the reconstruction MSE evaluation
    are implemented
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btac032
    title: 'massNet: integrated processing and classification of spatially resolved
      mass spectrometry data using deep learning for rapid tumor delineation'
  source_content_hash: sha256:897242e17e2072d9b71051ea99906da05ba49e3f4788077f7783d240f77f73ee
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/wabdelmoula/massNet
  tools:
  - Keras
  - Python
  - massNet
  - tensorflow
  edam_operation: http://edamontology.org/operation_0286
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 19
---

# tic-normalized-spectra-reconstruction-evaluation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Reproduces the massNet VAE training loop on TIC-normalized MALDI FT-ICR MSI spectra (85,062 m/z variables) and verifies reconstruction fidelity against the reported total MSE of 5.69 × 10^-8 under an 80/20 split with 5-fold cross-validation.

## When to use

Use when raw high-dimensional MSI spectra (imzML/HDF5, tens of thousands of m/z variables) need TIC-normalized reconstruction error quantification via a VAE, or when validating reported reconstruction MSE targets.

## When NOT to use

- Input is already a reduced feature/peak table or a peak-picked spectral matrix — the VAE operates on full TIC-normalized spectra without prior preprocessing or peak picking
- The analysis goal is multi-class tissue classification; the paper's demonstrated model only supports binary (normal vs tumor) classification
- Data lacks pixel-level histology annotations or cannot be TIC-normalized, so reconstruction error against the paper's setup would not be comparable

## Inputs

- TIC-normalized MALDI FT-ICR MSI training spectra in HDF5 format (85,062 m/z variables)
- Training spectra labels: 10,024 normal and 3,644 tumor spectra from 4 intracranial GBM PDX models
- Pathologist-derived pixel annotations transferred from H&E images
- massNet VAE module (Keras/TF) with 5-dimensional latent Code layer

## Outputs

- Trained VAE model weights
- JSON file recording the training reconstruction MSE (target: 5.69 × 10^-8)
- 5-fold cross-validation convergence/learning-stability assessment
- Reconstructed TIC-normalized MSI spectra from the learned latent variable

## How to apply

Download the MALDI FT-ICR MSI data from Metabolomics Workbench (project PR001292), load the TIC-normalized spectra in HDF5 format (85,062 m/z variables; 10,024 normal and 3,644 tumor training spectra) with pathologist-derived pixel annotations. Implement the massNet VAE in Python using Keras (2.2.0) with a Tensorflow backend, with a probabilistic encoder/decoder and a 5-dimensional latent variable sampled from a normal distribution. Split 80% training / 20% validation, optimize encoder/decoder parameters by maximizing the variational lower bound over 100 epochs with random shuffling of data batches, repeating under 5-fold cross-validation to assess learning stability. Use the optimized decoder to reconstruct the original TIC-normalized MSI spectra from the learned latent variable and compute the total MSE; save model weights and a JSON of the training MSE. Rationale: the VAE operates directly on full spectra without prior preprocessing or peak picking, so reconstruction MSE is the direct fidelity check that latent learning captured spectral structure.

## Related tools

- **massNet** (Deep learning VAE module whose probabilistic encoder/decoder with a 5-dimensional Code layer is trained and evaluated for reconstruction of TIC-normalized MSI spectra) — github.com/wabdelmoula/massNet
- **Keras** (Platform used to implement the VAE architecture (probabilistic encoder/decoder) in Python)
- **tensorflow** (Backend for the Keras-implemented VAE during variational lower bound optimization)
- **Python** (Language in which the massNet architecture and the reconstruction MSE evaluation are implemented)

## Examples

```
pip install numpy==1.15.4 scikit-learn==0.23.2 scipy==1.0.0 seaborn==0.9.0 pandas==1.1.1 h5py==2.7.1  # then run the massNet VAE trainer (Keras 2.2.0 / TF 1.8.0) on the PR001292 HDF5 spectra
```

## Evaluation signals

- Total reconstruction MSE over the TIC-normalized training spectra matches the reported target of 5.69 × 10^-8
- VAE optimization shows stable convergence within 100 epochs with random shuffling of data batches (paper reports 55.7 min total running time)
- The optimized model captures a 5-dimensional latent variable at the Code layer
- Training/validation split is 80/20 and the 5-fold CV over the training set is repeatable across folds (stability of learning)
- Saved JSON records a training MSE consistent with the reported value and saved weights reproduce it on re-load

## Limitations

- No software version numbers or hyperparameter values for the trained massNet model are reported in the paper text (versions are only in the README); exact reproduction of MSE 5.69 × 10^-8 may require the released code and pinned dependencies (Python 3.6.12, Keras 2.2.0, Tensorflow 1.8.0)
- Only binary (normal vs tumor) classification is demonstrated; the approach was shown on GBM PDX MSI and extension to multi-class is left for future development
- Ground truth annotation based solely on annotated histology is an approximation and may not be optimal, especially for intra-tumor heterogeneity

## Evidence
