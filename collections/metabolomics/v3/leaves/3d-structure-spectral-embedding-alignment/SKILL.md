---
name: 3d-structure-spectral-embedding-alignment
description: Use when you need joint embeddings of multi-modal spectra (IR, Raman,
  UV-Vis) and 3D molecular structures to condition a diffusion-based generative model
  on spectral data.
when_to_use_negative:
- When only a single spectrum type is available and no paired 3D structures exist
  — the contrastive alignment requires positive spectrum–structure pairs and is therefore
  ill-posed for unpaired data.
- 'When the goal is fine-tuning DMT for downstream generation rather than obtaining
  spectral encodings: use the saved checkpoint as conditioning input instead of re-running
  the alignment pre-training.'
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_0092
tools:
- name: SpecFormer
  role: Transformer-based spectral encoder that consumes patched IR, Raman, and UV-Vis
    sequences and produces z_s embeddings for contrastive alignment
  repo: https://github.com/AzureLeon1/DiffSpectra
- name: DiffSpectra
  role: Downstream framework whose DMT generator is conditioned on the aligned spectral
    embeddings produced by this skill
  repo: https://github.com/AzureLeon1/DiffSpectra
- name: PyTorch
  role: Deep-learning framework used to implement and train the contrastive and masked-reconstruction
    objectives
provenance:
  source_task_ids:
  - task_007
  source_content_hash: sha256:d4bc9836cca01827027aca0da0485535023420a8a6c323cbad66e40c3da0f310
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
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# 3d-structure-spectral-embedding-alignment

## Summary

Aligns a Transformer-based spectral encoder (SpecFormer) with a 3D molecular-structure encoder via InfoNCE contrastive learning so that spectral embeddings from IR, Raman, and UV-Vis patches are conditioned on structural context for downstream DMT-based structure generation.

## When to use

Use when you need joint embeddings of multi-modal spectra (IR, Raman, UV-Vis) and 3D molecular structures to condition a diffusion-based generative model on spectral data.

## When NOT to use

- When only a single spectrum type is available and no paired 3D structures exist — the contrastive alignment requires positive spectrum–structure pairs and is therefore ill-posed for unpaired data.
- When the goal is fine-tuning DMT for downstream generation rather than obtaining spectral encodings: use the saved checkpoint as conditioning input instead of re-running the alignment pre-training.

## Inputs

- QM9S training set: simulated spectra (IR, Raman, UV-Vis) paired with 3D molecular structures (from QM9)
- PubChemQC 3D-structure dataset (first-stage denoising pre-training, no spectra)
- Spectra patch sequences p_i per spectrum type
- Pre-trained 3D structural encoder producing z_x embeddings

## Outputs

- Pre-trained SpecFormer encoder checkpoint (e.g. exp/pretrained_specformer.ckpt)
- Spectral embeddings z_s aligned with structural embeddings z_x
- Training loss trace showing joint decline of L_Denoising, L_MPR, and L_Contrast

## How to apply

First split each spectrum type (IR, Raman, UV-Vis) into patches of length P_i with stride D_i, project them through a learnable linear transform W_i, and add learnable positional encodings; concatenate all spectra patch embeddings into one sequence for the SpecFormer Transformer encoder. Compute 3D structural embeddings z_x with a denoising pre-trained encoder and spectral embeddings z_s with SpecFormer, then train both with the InfoNCE contrastive loss L_Contrast using inner-product similarity f_x(z_x,z_s)=<z_x,z_s> to attract positive (molecule, spectrum) pairs and repel negatives. Combine this with the masked-patch reconstruction (MPR) loss over randomly masked spectral patches (masked ratio alpha, zero-vector replacement) as L = beta_Denoising*L_Denoising + beta_MPR*L_MPR + beta_Contrast*L_Contrast. Run a two-stage pipeline: stage one pre-trains the 3D encoder with denoising only on the large PubChemQC 3D-structure dataset, stage two fine-tunes jointly on only the QM9S training split to avoid data leakage. Save the pre-trained SpecFormer weights as a checkpoint (e.g. exp/pretrained_specformer.ckpt) and export z_s embeddings for conditioning the Diffusion Molecule Transformer.

## Related tools

- **SpecFormer** (Transformer-based spectral encoder that consumes patched IR, Raman, and UV-Vis sequences and produces z_s embeddings for contrastive alignment) — https://github.com/AzureLeon1/DiffSpectra
- **DiffSpectra** (Downstream framework whose DMT generator is conditioned on the aligned spectral embeddings produced by this skill) — https://github.com/AzureLeon1/DiffSpectra
- **PyTorch** (Deep-learning framework used to implement and train the contrastive and masked-reconstruction objectives)

## Examples

```
CUDA_VISIBLE_DEVICES=0,1 python main.py --config configs/diffspectra_qm9s.py --config_original_qm9 configs/base_qm9.py --mode train --workdir exp/allspectra_pretrained_specformer --config.data.spectra_version allspectra --config.model.name DMT --config.model.pretrained_specformer_path exp/pretrained_specformer.ckpt
```

## Evaluation signals

- The combined loss L = beta_Denoising*L_Denoising + beta_MPR*L_MPR + beta_Contrast*L_Contrast decreases monotonically during second-stage training.
- The checkpoint file exp/pretrained_specformer.ckpt exists and loads without error into the DMT model via --config.model.pretrained_specformer_path.
- Exported spectral embeddings z_s have the expected dimensionality d and, when paired with z_x, positive pairs score higher under inner-product similarity than negatives (InfoNCE behaviour).
- Zero data leakage: stage-two fine-tuning uses only the QM9S training split, never the validation/test spectra.

## Limitations

- Alignment is limited to three spectral modalities — UV-Vis, IR, and Raman; applicability to other modalities such as NMR or MS remains unexplored.
- The contrastive objective requires paired spectra–3D-structure samples; unpaired or single-modality data cannot be aligned.
- Stage-two pre-training is restricted to the QM9S training set, so generalisation to experimental (rather than simulated) spectra is not validated.

## Evidence

- [methods] contrastive alignment between spectral representations and molecular structures: [evidence span withheld — anchor status: not_found]
- [methods] patching and InfoNCE loss: "For each type of spectrum, we first segment the data into patches and independently encode these patches"
- [methods] contrastive learning with InfoNCE: [evidence span withheld — anchor status: not_found]
- [methods] QM9S-only second stage: "In the second stage, SpecFormer is pre-trained exclusively on the QM9S training set to avoid data leakage."
- [readme] two-stage pipeline and checkpoint usage: "CUDA_VISIBLE_DEVICES=0,1 python main.py --config configs/diffspectra_qm9s.py --config_original_qm9 configs/base_qm9.py --mode train --workdir exp/allspectra_pretrained_specformer"
