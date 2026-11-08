---
name: reconstruction-quality-evaluation-vae
description: Use when a trained ConditionalVAESTSM (or similar CVAE) model must be
  scored on reconstruction of ST gene expression and SM mass-spectrometry intensity
  profiles, latent embedding quality, and modality fusion, e.g. during ablation or
  tool benchmarking.
when_to_use_negative:
- Data are single-cell or subcellular resolution — SpatialMETA is designed for low-resolution
  ST and SM data only
- 'You need ground-truth gold-standard labels for biological conservation: only manually
  annotated labels from H&E histological features are available'
- The model has more than two modalities — SpatialMETA currently only supports integration
  of two modalities
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0091
tools:
- name: SpatialMETA
  role: Provides ConditionalVAESTSM and its get_normalized_expression, get_latent_embedding,
    and get_modality_contribution methods used to reconstruct profiles, extract the
    fused embedding, and compute modality contribution via angular similarity
  repo: https://github.com/WanluLiuLab/SpatialMETA
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-025-63915-z
    title: Integrating cross-sample and cross-modal data for spatial transcriptomics
      and metabolomics with SpatialMETA
  source_content_hash: sha256:48d8833b81b43d2f4b5722bd4d9be1691aece1a8e54cd1170c8bd0a0c361fe19
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
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# reconstruction-quality-evaluation-vae

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Evaluates VAE-based spatial multi-omics integration by scoring how well a trained model reconstructs ST and SM feature distributions and how biologically faithful its fused latent embedding is. Central to SpatialMETA benchmarking, where reconstruction capability and fused modality representation are the headline claims.

## When to use

Use when a trained ConditionalVAESTSM (or similar CVAE) model must be scored on reconstruction of ST gene expression and SM mass-spectrometry intensity profiles, latent embedding quality, and modality fusion, e.g. during ablation or tool benchmarking.

## When NOT to use

- Data are single-cell or subcellular resolution — SpatialMETA is designed for low-resolution ST and SM data only
- You need ground-truth gold-standard labels for biological conservation: only manually annotated labels from H&E histological features are available
- The model has more than two modalities — SpatialMETA currently only supports integration of two modalities

## Inputs

- Preprocessed joint ST-SM AnnData object (e.g. from spatialmeta.pp.joint_adata_sm_st)
- Trained ConditionalVAESTSM model (per ablation configuration: loss type, SM normalization, decoder architecture)
- Manually annotated spatial cluster labels (label key for scIB-style biological conservation scoring)

## Outputs

- Reconstructed normalized ST and SM expression/intensity profiles from get_normalized_expression
- Fused latent embedding matrix from get_latent_embedding
- Per-configuration metric table (reconstruction error, spatial continuity, marker recovery, biological conservation, modality contribution), e.g. ablation_metrics.csv

## How to apply

After training ConditionalVAESTSM on a joint ST-SM AnnData built with spatialmeta.pp.joint_adata_sm_st, reconstruct expression/intensity profiles with ConditionalVAESTSM.get_normalized_expression and extract the fused embedding with ConditionalVAESTSM.get_latent_embedding. Score reconstruction by comparing predicted vs. observed ST and SM feature distributions. Assess the embedding for spatial continuity and biological conservation against manually annotated spatial cluster labels (derived from H&E histological features via DestVI deconvolution with a ccRCC scRNA-seq reference). Quantify modality fusion via the angular similarity between modality embeddings returned by ConditionalVAESTSM.get_modality_contribution, and recover marker features to check biological signal retention. Assemble per-configuration metrics into a table (e.g. ablation_metrics.csv) so Gaussian-vs-MSE loss, SM normalization on/off, and shared-vs-independent decoder settings can be compared head-to-head.

## Related tools

- **SpatialMETA** (Provides ConditionalVAESTSM and its get_normalized_expression, get_latent_embedding, and get_modality_contribution methods used to reconstruct profiles, extract the fused embedding, and compute modality contribution via angular similarity) — https://github.com/WanluLiuLab/SpatialMETA

## Examples

```
model = ConditionalVAESTSM(adata); ConditionalVAESTSM.fit(model, adata); recon = model.get_normalized_expression(adata); emb = model.get_latent_embedding(adata); contrib = model.get_modality_contribution(adata)
```

## Evaluation signals

- Reconstruction error of both ST gene-expression and SM intensity feature distributions is computed and comparable across configurations
- Latent embedding shows spatial continuity consistent with tissue structure
- Marker feature recovery confirms biological signal is retained in the fused representation
- Biological conservation scores agree with manually annotated spatial cluster labels supplied as the label key
- Modality contribution (angular similarity between modality embeddings) is reported per configuration; Gaussian loss + SM normalization + shared decoder should outperform ablated variants

## Limitations

- Ground-truth labels for biology conservation evaluation were manually annotated from H&E histological features, not an independent gold standard
- Detailed benchmark metric values are only provided in source data files rather than in the main text
- SpatialMETA is designed for low-resolution ST and SM data and does not support single-cell or subcellular resolution

## Evidence

- [discussion] ground-truth labels manually annotated from H&E: [evidence span withheld — anchor status: not_found]
