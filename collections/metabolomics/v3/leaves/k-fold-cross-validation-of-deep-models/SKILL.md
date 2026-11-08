---
name: k-fold-cross-validation-of-deep-models
description: Use when training a deep learning model on high-dimensional spectra (e.g.,
  85,062 m/z variables) and you need to demonstrate that optimization converges stably
  rather than depending on a lucky train/validation split.
when_to_use_negative:
- When the goal is generalization assessment on held-out data — massNet evaluates
  final classification on a separate test set (9,798 normal / 2,861 tumor spectra)
  rather than relying on CV folds
- When input is a small, low-dimensional feature table where classical methods (e.g.,
  SVM with standard cross-validation) suffice
- When the data require peak picking or other preprocessing first — massNet's design
  deliberately avoids prior preprocessing and peak picking
edam_operation: http://edamontology.org/operation_0306
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_3172
tools:
- name: Keras
  role: Framework in which the VAE architecture is implemented and optimized over
    the 5-fold cross-validation splits
  repo: https://github.com/wabdelmoula/massNet
- name: tensorflow
  role: Backend executing the Keras model across 100 epochs with random shuffling
    of data batches
  repo: https://github.com/wabdelmoula/massNet
- name: massNet
  role: Deep learning model whose VAE module's training stability is evaluated via
    5-fold cross-validation
  repo: https://github.com/wabdelmoula/massNet
- name: Python
  role: Language for implementing the CV loop, data splitting, and MSE computation
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btac032
    title: 'massNet: integrated processing and classification of spatially resolved
      mass spectrometry data using deep learning for rapid tumor delineation'
  source_content_hash: sha256:39adc9ad89636872144792d9d813f2170da3e5ffd636a0b4d20f7a214dc74472
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/wabdelmoula/massNet
  repo_url: https://github.com/wabdelmoula/massNet
  tools:
  - Keras
  - Python
  - massNet
  - tensorflow
  edam_operation: http://edamontology.org/operation_0306
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# k-fold cross-validation of deep models

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Assess training stability of deep models (e.g., a variational autoencoder on mass spectrometry imaging data) by repeating optimization under 5-fold cross-validation on the training set and confirming stable convergence and reproducible reconstruction error.

## When to use

Use when training a deep learning model on high-dimensional spectra (e.g., 85,062 m/z variables) and you need to demonstrate that optimization converges stably rather than depending on a lucky train/validation split.

## When NOT to use

- When the goal is generalization assessment on held-out data — massNet evaluates final classification on a separate test set (9,798 normal / 2,861 tumor spectra) rather than relying on CV folds
- When input is a small, low-dimensional feature table where classical methods (e.g., SVM with standard cross-validation) suffice
- When the data require peak picking or other preprocessing first — massNet's design deliberately avoids prior preprocessing and peak picking

## Inputs

- TIC-normalized MALDI FT-ICR MSI spectra in HDF5 format (85,062 m/z variables; training set of 10,024 normal and 3,644 tumor spectra)
- Pathologist-derived pixel annotations defining normal vs. tumor classes
- Model architecture definition (e.g., probabilistic encoder/decoder VAE with 5-dimensional latent variable)

## Outputs

- Per-fold training/validation loss curves and convergence distribution across the 5 folds
- Trained model weights (e.g., Keras/V AE checkpoints) per fold
- JSON record of reconstruction MSE per fold and total (target: 5.69 × 10^-8)

## How to apply

Split the TIC-normalized spectra into an 80% training / 20% validation partition, then run 5-fold cross-validation on the training set. In each fold, implement the model in Keras with a TensorFlow backend (massNet used Python 3.6.12, Keras 2.2.0, TensorFlow 1.8.0) and optimize the encoder/decoder parameters by maximizing the variational lower bound over 100 epochs with random shuffling of data batches. Inspect the convergence distribution across folds (e.g., via a supplementary convergence plot) to confirm stable learning; massNet's VAE with a 5-dimensional latent variable converged stably in 100 epochs with a total running time of 55.7 minutes. Finally, use the optimized decoder to reconstruct the original TIC-normalized MSI spectra and verify the reconstruction quality is reproducible (massNet reported a total MSE of 5.69 × 10^-8). Rationale: cross-validation over repeated folds guards against instability from user parameter optimization and stochastic batch shuffling before the model is trusted for downstream classification.

## Related tools

- **Keras** (Framework in which the VAE architecture is implemented and optimized over the 5-fold cross-validation splits) — https://github.com/wabdelmoula/massNet
- **tensorflow** (Backend executing the Keras model across 100 epochs with random shuffling of data batches) — https://github.com/wabdelmoula/massNet
- **massNet** (Deep learning model whose VAE module's training stability is evaluated via 5-fold cross-validation) — https://github.com/wabdelmoula/massNet
- **Python** (Language for implementing the CV loop, data splitting, and MSE computation)

## Examples

```
python train_vae_5fold.py --data spectra.h5 --split 0.8 --folds 5 --epochs 100 --latent_dim 5 --json_out cv_metrics.json --weights_out vae_weights.h5
```

## Evaluation signals

- The convergence distribution across the 5 folds is stable (no diverging folds), replicating the reported 100-epoch convergence behavior
- Total reconstruction MSE of the TIC-normalized MSI data reproduces the reported value of 5.69 × 10^-8 (order of 10^-8)
- Latent variable at the Code layer is 5-dimensional as in the published architecture
- Total optimization time is on the order of the reported 55.7 minutes for comparable hardware/data size
- The 80/20 train/validation split preserves class proportions (10,024 normal / 3,644 tumor spectra in the training set)

## Limitations

- No software version numbers or full hyperparameter values for the trained model are reported in the article text; the README gives Python 3.6.12, Keras 2.2.0, TensorFlow 1.8.0 as the tested stack
- Learning stabilization relies on batch normalization and dropout; reproducing exact MSE requires matching these regularizers
- Cross-validation assesses training stability only — it does not substitute for evaluation on an independent test set or address intra-tumor heterogeneity in the ground-truth annotations

## Evidence
