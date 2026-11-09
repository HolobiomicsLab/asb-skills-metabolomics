---
name: cvae-architecture-implementation
description: Use when building or reproducing joint spatial transcriptomics + spatial
  metabolomics models that require a shared latent embedding across modalities and
  samples, with batch-label-conditioned decoding and per-spot modality contribution
  quantification.
when_to_use_negative:
- Input ST/SM data are at single-cell or subcellular resolution — SpatialMETA is designed
  for low-resolution ST and SM data
- The task requires integrating more than two modalities — SpatialMETA currently only
  supports integration of two modalities
- The modality data are sequencing counts on both sides needing a single likelihood
  — the ZINB/Gaussian split is specific to count-based ST plus continuous SM intensity
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: SpatialMETA
  role: Provides ConditionalVAESTSM CVAE with ZINB/Gaussian decoders, MMD batch-correction
    loss, get_latent_embedding and get_modality_contribution, plus load_adata for
    joint ST/SM AnnData
  repo: https://github.com/WanluLiuLab/SpatialMETA
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-025-63915-z
    title: Integrating cross-sample and cross-modal data for spatial transcriptomics
      and metabolomics with SpatialMETA
  source_content_hash: sha256:104052dc57e8b21bf2b56f987cac5197df59d6fdcdb977af34e2c28367ca8b77
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/WanluLiuLab/SpatialMETA
  repo_url: https://github.com/WanluLiuLab/SpatialMETA
  tools:
  - SpatialMETA
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6923076923076923
n_claims: 13
---

# cvae-architecture-implementation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Implement and train the SpatialMETA ConditionalVAESTSM, a conditional variational autoencoder that jointly embeds ST gene expression (ZINB likelihood) and SM metabolite intensities (Gaussian likelihood) with batch labels during decoding, enabling simultaneous vertical and horizontal integration with MMD-based batch correction.

## When to use

Use when building or reproducing joint spatial transcriptomics + spatial metabolomics models that require a shared latent embedding across modalities and samples, with batch-label-conditioned decoding and per-spot modality contribution quantification.

## When NOT to use

- Input ST/SM data are at single-cell or subcellular resolution — SpatialMETA is designed for low-resolution ST and SM data
- The task requires integrating more than two modalities — SpatialMETA currently only supports integration of two modalities
- The modality data are sequencing counts on both sides needing a single likelihood — the ZINB/Gaussian split is specific to count-based ST plus continuous SM intensity

## Inputs

- Preprocessed joint ST/SM AnnData (single or multiple samples) with sample/batch condition key in obs
- Batch/sample condition labels in adata.obs for conditional decoding and MMD loss
- Optional trained-model checkpoint for reload/inference

## Outputs

- Trained ConditionalVAESTSM model checkpoint (re-loadable)
- Per-spot shared latent embedding vector from get_latent_embedding()
- Per-spot modality contribution vector (angular similarity between ST- and SM-derived embeddings), e.g. written to modality_contribution.csv

## How to apply

Load the preprocessed joint ST/SM AnnData (e.g. human ccRCC) via spatialmeta.data.load_adata and confirm the sample/batch condition key in obs. Instantiate spatialmeta.model.ConditionalVAESTSM with batch-invariant ST and SM encoders whose posteriors are concatenated into a joint latent embedding. Train with ConditionalVAESTSM.fit(), optimizing a zero-inflated negative binomial (ZINB) reconstruction loss for ST count data and a Gaussian reconstruction loss for continuous SM intensity data, together with an MMD loss across samples so batch effects are corrected while biology is conserved; batch labels are incorporated during decoding. Extract the per-spot shared latent embedding with get_latent_embedding() and compute per-spot modality contribution as the angular similarity between the ST- and SM-derived embeddings via get_modality_contribution(). Rationale: SM data are continuous mass spectrometry intensities rather than sequencing counts, so the decoders and loss functions are tailored per modality to enhance modality fusion, batch effect correction, and biological conservation within a single VAE framework.

## Related tools

- **SpatialMETA** (Provides ConditionalVAESTSM CVAE with ZINB/Gaussian decoders, MMD batch-correction loss, get_latent_embedding and get_modality_contribution, plus load_adata for joint ST/SM AnnData) — https://github.com/WanluLiuLab/SpatialMETA

## Examples

```
from spatialmeta.model import ConditionalVAESTSM; model = ConditionalVAESTSM(adata, condition_key='sample'); model.fit(); emb = model.get_latent_embedding(); mc = model.get_modality_contribution()
```

## Evaluation signals

- Trained checkpoint reloads successfully and ConditionalVAESTSM.get_latent_embedding() returns a vector for every spot
- get_modality_contribution() angular-similarity values are complete (one per spot) and all finite
- Reconstruction quality: model should demonstrate superior reconstruction capability and fused modality representation, accurately capturing ST and SM feature distributions versus baselines
- Batch mixing: MMD loss across samples converges and horizontal integration yields unified spatial clusters with consistent interactions across samples
- Latent embedding reproduces biologically conserved structure (e.g. tumor-infiltrating immune clusters with upregulated lipid metabolic pathways in ccRCC) rather than batch artifacts

## Limitations

- SpatialMETA is designed for low-resolution ST and SM data and does not support single-cell or subcellular resolution
- SpatialMETA currently only supports integration of two modalities; more than two modalities is a future need
- Horizontal integration was validated only on a limited number of samples from a single study
- Modality interpretability depends on the angular-similarity design; detailed benchmark metric values are provided only in source data files

## Evidence

- [methods] ZINB/Gaussian likelihoods with batch-conditioned decoding: [evidence span withheld — anchor status: not_found]
