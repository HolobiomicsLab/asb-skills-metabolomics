---
name: infonce-contrastive-alignment
description: Use when pre-training a spectral encoder on multi-modal spectra (IR,
  Raman, UV-Vis) whose embeddings must be aligned with 3D structural embeddings for
  downstream spectra-conditioned molecular generation.
when_to_use_negative:
- The task is training the Diffusion Molecule Transformer itself for denoising-based
  structure generation; contrastive alignment is a pre-training step for the spectral
  encoder, not the denoising network.
- The spectral encoder will process spectra from modalities outside the three supported
  types (e.g. NMR or mass spectrometry); the article states applicability to such
  modalities remains to be explored.
- Only a single spectrum type is available and no 3D structural embeddings can be
  produced; contrastive alignment requires paired spectrum-structure data.
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_2269
- http://edamontology.org/topic_0154
tools:
- name: SpecFormer
  role: Transformer-based spectrum encoder producing spectral embeddings z_s that
    are contrastively aligned with structural embeddings; patch-segments each spectrum
    type (IR, Raman, UV-Vis) and pre-trained with masked patches reconstruction plus
    InfoNCE alignment
  repo: https://github.com/AzureLeon1/DiffSpectra
- name: DiffSpectra
  role: Framework whose downstream DMT is conditioned on the spectral embeddings produced
    by this contrastive pre-training; repo contains scripts to launch the two-stage
    pre-training and save the SpecFormer checkpoint
  repo: https://github.com/AzureLeon1/DiffSpectra
- name: PyTorch
  role: Deep learning framework used to implement the InfoNCE loss and the two-stage
    pre-training pipeline
provenance:
  source_task_ids:
  - task_007
  source_content_hash: sha256:aeb42cf10e0a79a8db1c494f28326f8636300f54aea63cd2e410a1daa5c070a0
  source_papers:
  - doi: 10.48550/arxiv.2507.06853
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/AzureLeon1/DiffSpectra
  tools:
  - DiffSpectra
  - PyTorch
  - SpecFormer
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_2269
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 14
---

# infonce-contrastive-alignment

## Summary

Aligns spectral embeddings from a Transformer-based spectrum encoder with 3D molecular structure embeddings using the InfoNCE loss, so that the spectral encoder learns representations suitable for conditioning molecular structure diffusion generation.

## When to use

Use when pre-training a spectral encoder on multi-modal spectra (IR, Raman, UV-Vis) whose embeddings must be aligned with 3D structural embeddings for downstream spectra-conditioned molecular generation.

## When NOT to use

- The task is training the Diffusion Molecule Transformer itself for denoising-based structure generation; contrastive alignment is a pre-training step for the spectral encoder, not the denoising network.
- The spectral encoder will process spectra from modalities outside the three supported types (e.g. NMR or mass spectrometry); the article states applicability to such modalities remains to be explored.
- Only a single spectrum type is available and no 3D structural embeddings can be produced; contrastive alignment requires paired spectrum-structure data.

## Inputs

- Multi-modal spectra from the QM9S dataset (IR, Raman, UV-Vis) as patch sequences
- 3D molecular structures (e.g. PubChemQC for stage one; QM9S training set for stage two)
- Pre-trained SpecFormer spectral encoder weights
- Pre-trained 3D structural encoder producing structural embeddings z_x

## Outputs

- Pre-trained SpecFormer spectral encoder checkpoint (e.g. exp/pretrained_specformer.ckpt)
- Spectral embeddings z_s of dimensionality d suitable for conditioning molecular structure generation
- Training loss trace with decreasing combined loss L_Contrast during training

## How to apply

First, obtain 3D structural embeddings z_x via a denoising pre-training objective on a large 3D-structure dataset, and spectral embeddings z_s via the SpecFormer encoder that segments each spectrum type into patches of length P_i with stride D_i (overlapping when D_i < P_i) and projects them with a learnable linear transformation W_i plus positional encoding W_i_pos. Then compute pairwise similarity as the inner product f_x(z_x,z_s) = <z_x,z_s> and train with the InfoNCE contrastive loss L_Contrast, pulling together positive (spectrum, structure) pairs and pushing apart negatives. Run this as the second stage of a two-stage pre-training pipeline: the first stage performs denoising-only pre-training on the large 3D-structure dataset (PubChemQC); the second stage jointly optimizes L = beta_Denoising*L_Denoising + beta_MPR*L_MPR + beta_Contrast*L_Contrast using only the QM9S training set for spectra to avoid data leakage. Masked patches reconstruction (MPR) is trained alongside contrastive alignment in stage two, with masked patches replaced by zero vectors and an MSE reconstruction loss over the masked patches. Finally, save the pre-trained SpecFormer encoder weights as a checkpoint (e.g. exp/pretrained_specformer.ckpt) and export the spectral embeddings z_s for use as conditioning input to the Diffusion Molecule Transformer.

## Related tools

- **SpecFormer** (Transformer-based spectrum encoder producing spectral embeddings z_s that are contrastively aligned with structural embeddings; patch-segments each spectrum type (IR, Raman, UV-Vis) and pre-trained with masked patches reconstruction plus InfoNCE alignment) — https://github.com/AzureLeon1/DiffSpectra
- **DiffSpectra** (Framework whose downstream DMT is conditioned on the spectral embeddings produced by this contrastive pre-training; repo contains scripts to launch the two-stage pre-training and save the SpecFormer checkpoint) — https://github.com/AzureLeon1/DiffSpectra
- **PyTorch** (Deep learning framework used to implement the InfoNCE loss and the two-stage pre-training pipeline)

## Examples

```
CUDA_VISIBLE_DEVICES=0,1 python main.py --config configs/diffspectra_qm9s.py --config_original_qm9 configs/base_qm9.py --mode train --workdir exp/allspectra_pretrained_specformer --config.data.spectra_version allspectra --config.model.name DMT --config.model.pretrained_specformer_path exp/pretrained_specformer.ckpt
```

## Evaluation signals

- The combined pre-training loss L = beta_Denoising*L_Denoising + beta_MPR*L_MPR + beta_Contrast*L_Contrast decreases monotonically over training epochs.
- A checkpoint file exp/pretrained_specformer.ckpt exists and loads successfully into a fresh SpecFormer instance.
- Exported spectral embeddings z_s have the expected dimensionality d and can be passed as `--config.model.pretrained_specformer_path` to downstream DMT training.
- InfoNCE alignment improves retrieval of the correct structure for a given spectrum: positive (z_x, z_s) pairs rank above negatives under inner-product similarity f_x = <z_x,z_s>.

## Limitations

- The second stage uses only the QM9S training set to avoid data leakage, limiting spectral-diversity of alignment; performance benefits substantially from this pre-training but the pipeline is confined to three spectral types (UV-Vis, IR, Raman).
- The article notes validation to date has been primarily computational, so the aligned spectral embeddings have not been validated against experimental laboratory spectra.
- Masked patch reconstruction and contrastive alignment are trained jointly in stage two; if beta_Contrast is set too low relative to beta_MPR, alignment quality may be under-optimized relative to reconstruction.

## Evidence

- [abstract] SpecFormer is pre-trained using two strategies: masked patch reconstruction over the spectra and contrastive alignment between spectral representations and molecular structures.: [evidence span withheld — anchor status: not_found]
- [methods] InfoNCE loss: [evidence span withheld — anchor status: not_found]
