---
name: masked-patch-reconstruction-pretraining
description: Reference for pre-training a spectral encoder with masked patch reconstruction
  before using its embeddings to condition molecular structure generation.
when_to_use_negative:
- Downstream task uses only a single spectral modality or no spectral data at all
  — pre-training benefits come from cross-spectrum contrastive alignment and may not
  transfer.
- The 3D structure pre-training set is not disjoint from the downstream evaluation
  set; the paper pre-trains exclusively on the QM9S training set to avoid data leakage.
- Task requires generating molecular structures directly without a pre-trained spectral
  encoder — the pre-training stage is only useful when its embeddings condition a
  downstream generator like DMT.
edam_operation: http://edamontology.org/operation_3936
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_3332
tools:
- name: SpecFormer
  role: Transformer-based spectral encoder that receives patch embeddings and is trained
    with the masked patch reconstruction and contrastive objectives
  repo: https://github.com/AzureLeon1/DiffSpectra
- name: DiffSpectra
  role: Framework whose downstream DMT diffusion model consumes the pre-trained SpecFormer
    embeddings via config.model.pretrained_specformer_path
  repo: https://github.com/AzureLeon1/DiffSpectra
provenance:
  source_task_ids:
  - task_007
  source_content_hash: sha256:4f4163835cc1789464e1a1737334443f943cf68ed4fbef363ad140bf2facccdd
  source_papers:
  - doi: 10.48550/arxiv.2507.06853
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/AzureLeon1/DiffSpectra
  tools:
  - DiffSpectra
  - SpecFormer
  edam_operation: http://edamontology.org/operation_3936
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_3332
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# masked-patch-reconstruction-pretraining

## Summary

A self-supervised pre-training strategy in which spectra are segmented into patches, a masked fraction is reconstructed via an MSE objective, and spectral embeddings are additionally contrastively aligned with 3D molecular structure embeddings to yield high-quality conditioning representations for downstream spectra-conditioned diffusion generation.

## When to use

Reference for pre-training a spectral encoder with masked patch reconstruction before using its embeddings to condition molecular structure generation.

## When NOT to use

- Downstream task uses only a single spectral modality or no spectral data at all — pre-training benefits come from cross-spectrum contrastive alignment and may not transfer.
- The 3D structure pre-training set is not disjoint from the downstream evaluation set; the paper pre-trains exclusively on the QM9S training set to avoid data leakage.
- Task requires generating molecular structures directly without a pre-trained spectral encoder — the pre-training stage is only useful when its embeddings condition a downstream generator like DMT.

## Inputs

- QM9S spectra per molecule for IR, Raman, and UV-Vis modalities
- 3D molecular structures from PubChemQC (stage-1) and QM9S (stage-2)
- Hyperparameters: patch length P_i, stride D_i, masked ratio alpha, loss weights beta_Denoising, beta_MPR, beta_Contrast

## Outputs

- Pre-trained SpecFormer encoder checkpoint (exp/pretrained_specformer.ckpt)
- Spectral embeddings z_s conditioned on multi-modal spectra
- Training loss curve combining L_Denoising, L_MPR, and L_Contrast

## How to apply

Split each spectrum type (IR, Raman, UV-Vis from QM9S) into patches of length P_i with stride D_i, project them via a learnable linear transformation plus learnable positional encoding, and concatenate patch embeddings across spectra into one sequence for the Transformer encoder. Randomly mask a fraction of patches (masked ratio alpha) by replacing them with zero vectors, and train SpecFormer with the MSE reconstruction loss L_MPR over the masked patches for each spectrum type. Combine this with an InfoNCE contrastive objective that aligns spectral embeddings z_s with 3D structural embeddings z_x via inner-product similarity, and run a two-stage pipeline: stage 1 is denoising-only pre-training on PubChemQC 3D structures, stage 2 optimizes L = beta_Denoising*L_Denoising + beta_MPR*L_MPR + beta_Contrast*L_Contrast using only the QM9S training set to avoid data leakage. Save the pre-trained encoder as a checkpoint (e.g. exp/pretrained_specformer.ckpt) and pass it to the downstream DMT diffusion model via config.model.pretrained_specformer_path.

## Related tools

- **SpecFormer** (Transformer-based spectral encoder that receives patch embeddings and is trained with the masked patch reconstruction and contrastive objectives) — https://github.com/AzureLeon1/DiffSpectra
- **DiffSpectra** (Framework whose downstream DMT diffusion model consumes the pre-trained SpecFormer embeddings via config.model.pretrained_specformer_path) — https://github.com/AzureLeon1/DiffSpectra

## Examples

```
CUDA_VISIBLE_DEVICES=0,1 python main.py --config configs/diffspectra_qm9s.py --config_original_qm9 configs/base_qm9.py --mode train --workdir exp/allspectra_pretrained_specformer --config.data.spectra_version allspectra --config.model.pretrained_specformer_path exp/pretrained_specformer.ckpt
```

## Evaluation signals

- Checkpoint file exp/pretrained_specformer.ckpt exists and loads into the downstream model without shape errors.
- Combined two-stage loss (denoising + MPR + contrastive) decreases monotonically over training epochs.
- Masked-patch reconstruction MSE per spectrum type converges to a low, stable value on the QM9S training split.
- InfoNCE alignment separates positive spectral–structure pairs from negatives, verified by decreasing L_Contrast.
- Downstream DMT training with --config.model.pretrained_specformer_path runs without loading errors.

## Limitations

- Pre-training was validated only on QM9S with three spectral types (UV-Vis, IR, Raman); transfer to NMR or mass spectrometry is unexplored.
- The contrastive objective requires paired 3D structures; spectra without ground-truth 3D geometry cannot be aligned.
- The README states only 'Install the environment according to the commands in env.sh' — no pinned dependency versions are given in the article section.

## Evidence

- [methods] Contrastive objective alongside MPR.: [evidence span withheld — anchor status: not_found]
