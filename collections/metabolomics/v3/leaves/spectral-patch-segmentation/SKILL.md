---
name: spectral-patch-segmentation
description: Use when preparing simulated multi-modal spectra (IR, Raman, UV-Vis from
  QM9S) for a Transformer-based spectral encoder that conditions a molecular-structure
  diffusion model and needs patch-level embeddings rather than point-wise encodings.
when_to_use_negative:
- The downstream model encodes each spectral frequency point individually rather than
  via a Transformer that consumes patch sequences — patch segmentation would be unnecessary
  overhead.
- Only a single spectral modality is available and the analysis does not require cross-spectrum
  (inter-modality) dependencies; a simpler point-wise encoder may suffice.
- The spectra are experimental/real measurements of different modality types (e.g.
  NMR or MS) outside the IR/Raman/UV-Vis scope that the DiffSpectra pipeline was trained
  on.
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3473
- http://edamontology.org/topic_0154
tools:
- name: DiffSpectra
  role: Downstream conditional diffusion framework whose DMT denoiser consumes the
    patch-encoded spectral embeddings produced by this skill.
  repo: https://github.com/AzureLeon1/DiffSpectra
- name: SpecFormer
  role: Transformer-based spectral encoder that ingests the concatenated patch embeddings
    and is pre-trained with masked patch reconstruction and contrastive alignment.
  repo: https://github.com/AzureLeon1/DiffSpectra
provenance:
  source_task_ids:
  - task_007
  source_content_hash: sha256:af7862e16cfa472f141dd278e59d6916f5c49df0f77d9e267e35464cb599e70d
  source_papers:
  - doi: 10.48550/arxiv.2507.06853
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/AzureLeon1/DiffSpectra
  tools:
  - DiffSpectra
  - SpecFormer
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3473
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# spectral-patch-segmentation

## Summary

Segments each type of multi-modal molecular spectrum (IR, Raman, UV-Vis) into patches and projects them with learnable linear transformations plus positional encodings before feeding them to the SpecFormer Transformer encoder, capturing both intra- and inter-spectrum dependencies.

## When to use

Use when preparing simulated multi-modal spectra (IR, Raman, UV-Vis from QM9S) for a Transformer-based spectral encoder that conditions a molecular-structure diffusion model and needs patch-level embeddings rather than point-wise encodings.

## When NOT to use

- The downstream model encodes each spectral frequency point individually rather than via a Transformer that consumes patch sequences — patch segmentation would be unnecessary overhead.
- Only a single spectral modality is available and the analysis does not require cross-spectrum (inter-modality) dependencies; a simpler point-wise encoder may suffice.
- The spectra are experimental/real measurements of different modality types (e.g. NMR or MS) outside the IR/Raman/UV-Vis scope that the DiffSpectra pipeline was trained on.

## Inputs

- Multi-modal simulated spectra for each molecule: IR, Raman, and UV-Vis curves from the QM9S dataset (QM9 augmented with simulated spectra)
- Patch length P_i and stride D_i hyperparameters per spectrum type
- Masked ratio alpha for the masked patch reconstruction pre-training phase

## Outputs

- Concatenated sequence of patch embeddings (linear projections + positional encodings) ready for the SpecFormer Transformer encoder
- Pre-trained SpecFormer checkpoint (e.g. exp/pretrained_specformer.ckpt) containing spectral encoder weights
- Spectral embeddings z_s used to condition the Diffusion Molecule Transformer (DMT) during denoising

## How to apply

For each spectrum type, segment the spectral data into patches of length P_i with stride D_i, allowing overlap when D_i < P_i, to yield patch sequences p_i. Project each patch sequence via a learnable linear transformation W_i and add a learnable positional encoding W_i_pos so that the encoder can recover frequency locality. Concatenate the patch embeddings from all spectrum types into a single sequence for the Transformer encoder, enabling joint intra- and inter-spectrum dependency modeling across IR, Raman, and UV-Vis inputs. In the pre-training phase, randomly mask a fraction of patches (masked ratio alpha, replacing masked patches with zero vectors) and train SpecFormer with the MSE reconstruction loss L_MPR over the masked patches per spectrum type. The patched embeddings ultimately feed a conditional diffusion model (DiffSpectra), where they guide joint 2D topology and 3D geometry generation via the Diffusion Molecule Transformer (DMT).

## Related tools

- **DiffSpectra** (Downstream conditional diffusion framework whose DMT denoiser consumes the patch-encoded spectral embeddings produced by this skill.) — https://github.com/AzureLeon1/DiffSpectra
- **SpecFormer** (Transformer-based spectral encoder that ingests the concatenated patch embeddings and is pre-trained with masked patch reconstruction and contrastive alignment.) — https://github.com/AzureLeon1/DiffSpectra

## Examples

```
CUDA_VISIBLE_DEVICES=0,1 python main.py --config configs/diffspectra_qm9s.py --config_original_qm9 configs/base_qm9.py --mode train --workdir exp/allspectra_pretrained_specformer --config.data.spectra_version allspectra --config.model.name DMT --config.model.pretrained_specformer_path exp/pretrained_specformer.ckpt
```

## Evaluation signals

- The patching routine yields per-spectrum-type patch sequences p_i whose length equals ceil((n_i - P_i) / D_i) + 1 for a spectrum of n_i points, confirming correct stride/overlap handling.
- During MPR pre-training, the combined loss L_MPR decreases monotonically over training epochs for each of the three spectrum types, indicating the encoder learns to reconstruct masked patches.
- The saved checkpoint file (e.g. exp/pretrained_specformer.ckpt) exists and loads without error into the downstream DMT training run.
- Exported spectral embeddings z_s have consistent dimensionality d across the dataset and show decreasing InfoNCE contrastive loss when jointly trained with 3D structural embeddings.

## Limitations

- Patch segmentation was designed for three spectral types (UV-Vis, IR, Raman); applicability to other modalities such as NMR or MS remains unexplored.
- Patch length P_i and stride D_i must be tuned per spectrum type; suboptimal choices can lose fine spectral features or inflate sequence length.
- The masked patch reconstruction pre-training uses the QM9S training set only, so patch embeddings are tied to simulated spectra rather than real experimental measurements.

## Evidence
