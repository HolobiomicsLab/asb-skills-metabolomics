---
name: cvae-vertical-integration-st-sm
description: Use when you have a joint ST-SM AnnData object from paired sections of
  one sample and need to fuse both modalities into a shared latent embedding, or to
  test which model components (loss, normalization, decoder sharing) drive integration
  quality.
when_to_use_negative:
- Data are at single-cell or subcellular resolution — SpatialMETA is designed for
  low-resolution ST and SM data
- You need to integrate more than two modalities — SpatialMETA currently only supports
  integration of two modalities
- The ST and SM measurements come from non-paired samples across studies — that calls
  for horizontal integration, not same-section vertical integration
edam_operation: http://edamontology.org/operation_3919
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3520
tools:
- name: SpatialMETA
  role: Provides ConditionalVAESTSM (fit, get_normalized_expression, get_latent_embedding,
    get_modality_contribution) and spatialmeta.pp.joint_adata_sm_st for CVAE-based
    vertical ST-SM integration and ablation benchmarking
  repo: https://github.com/WanluLiuLab/SpatialMETA
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-025-63915-z
    title: Integrating cross-sample and cross-modal data for spatial transcriptomics
      and metabolomics with SpatialMETA
  source_content_hash: sha256:1961935d5d6dc4ed0d0e2af6f40071cb88153b1f3ef0ef503cb7510a54405a71
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
  edam_operation: http://edamontology.org/operation_3919
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 15
---

# cvae-vertical-integration-st-sm

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

This skill reproduces SpatialMETA's CVAE-based vertical integration of spatial transcriptomics (ST) and spatial metabolomics (SM) data on the same tissue section, including the ablation study over SM loss function (Gaussian vs MSE), SM normalization, and shared vs independent decoders.

## When to use

Use when you have a joint ST-SM AnnData object from paired sections of one sample and need to fuse both modalities into a shared latent embedding, or to test which model components (loss, normalization, decoder sharing) drive integration quality.

## When NOT to use

- Data are at single-cell or subcellular resolution — SpatialMETA is designed for low-resolution ST and SM data
- You need to integrate more than two modalities — SpatialMETA currently only supports integration of two modalities
- The ST and SM measurements come from non-paired samples across studies — that calls for horizontal integration, not same-section vertical integration

## Inputs

- Joint ST-SM AnnData object built with spatialmeta.pp.joint_adata_sm_st (low-resolution ST spots with reassigned SM intensities from the same sample)
- Manually annotated spatial cluster labels (e.g. DestVI deconvolution against a ccRCC scRNA-seq reference plus marker-based annotation) for biological conservation scoring

## Outputs

- Trained ConditionalVAESTSM models for each ablation configuration
- Reconstructed ST expression and SM intensity profiles via get_normalized_expression
- Fused latent embeddings via get_latent_embedding
- Per-configuration metrics table (ablation_metrics.csv) covering reconstruction error, spatial continuity, marker recovery, biological conservation, and modality contribution

## How to apply

Build a joint ST-SM AnnData object with spatialmeta.pp.joint_adata_sm_st for the ccRCC vertical-integration case, after SM-to-ST alignment via rotation/translation/non-linear distortion and KNN reassignment. Train ConditionalVAESTSM (ConditionalVAESTSM.fit) for every configuration in the ablation grid: Gaussian vs MSE reconstruction loss on the SM modality, SM normalization enabled vs disabled, and shared vs independent decoders — the paper found Gaussian loss and SM normalization are critical, and a shared decoder improves modality fusion. For each model, reconstruct expression/intensity profiles with ConditionalVAESTSM.get_normalized_expression and extract the fused embedding with ConditionalVAESTSM.get_latent_embedding. Score each configuration on reconstruction error of ST and SM feature distributions, spatial continuity of the latent embedding, marker feature recovery, and biological conservation against manually annotated spatial cluster labels. Compute per-configuration modality contribution from the angular similarity between embeddings via ConditionalVAESTSM.get_modality_contribution, then assemble the configuration-by-metric table (e.g. ablation_metrics.csv).

## Related tools

- **SpatialMETA** (Provides ConditionalVAESTSM (fit, get_normalized_expression, get_latent_embedding, get_modality_contribution) and spatialmeta.pp.joint_adata_sm_st for CVAE-based vertical ST-SM integration and ablation benchmarking) — https://github.com/WanluLiuLab/SpatialMETA

## Examples

```
model = spatialmeta.model.ConditionalVAESTSM(adata); model.fit(); emb = model.get_latent_embedding(adata)
```

## Evaluation signals

- Ablation table shows Gaussian SM loss and SM-normalized configurations outperform MSE/unnormalized ones on reconstruction error of ST and SM feature distributions
- Shared-decoder configurations score higher on spatial continuity of the latent embedding and marker feature recovery than independent-decoder ones
- Biological conservation scores (scIB against manually annotated spatial cluster labels) are preserved for the best configuration
- get_modality_contribution yields meaningful angular similarity values between embeddings for both modalities, not degenerate (0 or 1) contributions
- Latent embedding reproduces the manually annotated spatial clusters (e.g. the ccRCC immune cluster with upregulated lipid metabolic pathways)

## Limitations

- Ground-truth labels for biology conservation evaluation were manually annotated from H&E histological features, not an independent gold standard
- Detailed benchmark metric values are provided in source data files rather than in the main text
- SpatialMETA is designed for low-resolution ST and SM data and does not support single-cell or subcellular resolution

## Evidence
