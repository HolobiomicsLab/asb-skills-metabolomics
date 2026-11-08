---
name: sm-intensity-normalization-assessment
description: Use when tuning SpatialMETA's ConditionalVAESTSM for vertical integration
  of ST with SM mass-spectrometry intensities and you must decide between Gaussian
  vs MSE SM loss, SM normalization on/off, and shared vs independent decoders.
when_to_use_negative:
- ST data are sequencing counts or SM data have already been normalized and a single
  fixed model configuration is required — this skill is for comparing loss/normalization/decoder
  choices, not for a single fixed run
- The task is horizontal (cross-sample) integration or involves more than two modalities
  — SpatialMETA supports integration of only two modalities and its ablation grid
  addresses vertical ST-SM fusion
- Data are single-cell or subcellular resolution ST/SM — SpatialMETA is designed for
  low-resolution ST and SM data
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: SpatialMETA
  role: Provides ConditionalVAESTSM with fit, get_normalized_expression, get_latent_embedding,
    and get_modality_contribution for training the ablation grid and scoring SM loss/normalization/decoder
    configurations
  repo: https://github.com/WanluLiuLab/SpatialMETA
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-025-63915-z
    title: Integrating cross-sample and cross-modal data for spatial transcriptomics
      and metabolomics with SpatialMETA
  source_content_hash: sha256:8a4496d1b275e543b10dcab2567d338cdbe53e74ebaad1842d3ad1af1dd95a2a
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
  edam_operation: http://edamontology.org/operation_3799
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# sm-intensity-normalization-assessment

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

An ablation-based assessment skill for judging how SM data normalization, the SM reconstruction loss (Gaussian vs MSE), and shared vs independent decoder architecture in a ConditionalVAESTSM affect vertical ST-SM integration quality. It exists because SM data are continuous mass spectrometry intensities, not sequencing counts, so normalization and loss design are critical model choices.

## When to use

Use when tuning SpatialMETA's ConditionalVAESTSM for vertical integration of ST with SM mass-spectrometry intensities and you must decide between Gaussian vs MSE SM loss, SM normalization on/off, and shared vs independent decoders.

## When NOT to use

- ST data are sequencing counts or SM data have already been normalized and a single fixed model configuration is required — this skill is for comparing loss/normalization/decoder choices, not for a single fixed run
- The task is horizontal (cross-sample) integration or involves more than two modalities — SpatialMETA supports integration of only two modalities and its ablation grid addresses vertical ST-SM fusion
- Data are single-cell or subcellular resolution ST/SM — SpatialMETA is designed for low-resolution ST and SM data

## Inputs

- Preprocessed joint ST-SM AnnData object built with spatialmeta.pp.joint_adata_sm_st (ccRCC vertical-integration case)
- Manually annotated spatial cluster labels (DestVI deconvolution with ccRCC scRNA-seq reference + marker-based annotation) as ground truth for biological conservation scoring

## Outputs

- Configuration-by-metric ablation table written to ablation_metrics.csv (reconstruction error, spatial continuity, marker feature recovery, biological conservation, modality contribution)
- Per-configuration normalized expression/intensity reconstructions from ConditionalVAESTSM.get_normalized_expression
- Per-configuration fused latent embeddings from ConditionalVAESTSM.get_latent_embedding
- Per-configuration modality contribution scores from ConditionalVAESTSM.get_modality_contribution (angular similarity between embeddings)

## How to apply

Build the joint ST-SM AnnData with spatialmeta.pp.joint_adata_sm_st for the ccRCC vertical-integration case, then train ConditionalVAESTSM with ConditionalVAESTSM.fit across the full ablation grid: Gaussian vs MSE reconstruction loss on the SM modality, SM normalization enabled vs disabled, and shared vs independent decoders. For each configuration, reconstruct ST and SM profiles with ConditionalVAESTSM.get_normalized_expression and extract the fused embedding with ConditionalVAESTSM.get_latent_embedding. Score each configuration on reconstruction error of ST and SM feature distributions, spatial continuity of the latent embedding, marker feature recovery, and biological conservation against the manually annotated spatial cluster labels (DestVI deconvolution with the ccRCC scRNA-seq reference). Quantify modality fusion per configuration from the angular similarity between embeddings via ConditionalVAESTSM.get_modality_contribution. Assemble all metrics into a configuration-by-metric table (ablation_metrics.csv); expect the Gaussian loss and SM normalization to be critical for effective vertical integration, and the shared decoder to improve modality fusion over independent decoders.

## Related tools

- **SpatialMETA** (Provides ConditionalVAESTSM with fit, get_normalized_expression, get_latent_embedding, and get_modality_contribution for training the ablation grid and scoring SM loss/normalization/decoder configurations) — https://github.com/WanluLiuLab/SpatialMETA

## Examples

```
from spatialmeta.model import ConditionalVAESTSM; model = ConditionalVAESTSM.fit(joint_adata, reconstruction_loss_sm='gaussian', normalize_sm=True, shared_decoder=True)
```

## Evaluation signals

- Gaussian-loss and SM-normalization configurations show markedly lower ST and SM reconstruction error than MSE-loss / no-normalization variants
- Shared-decoder configurations yield higher angular-similarity modality contribution scores than independent-decoder configurations
- Latent embeddings of winning configurations show spatial continuity consistent with manually annotated spatial cluster labels (biological conservation)
- Ablation table ablation_metrics.csv contains one complete row per configuration in the 2x2x2 grid with all metrics populated
- Marker feature recovery in reconstructions matches manually annotated spatial clusters based on marker gene expression

## Limitations

- Ground-truth labels for biological conservation evaluation were manually annotated from H&E histological features, not an independent gold standard
- Detailed benchmark metric values are provided only in source data files rather than the main text, so expected metric ranges must be fetched from source data
- SpatialMETA is designed for low-resolution ST and SM data and currently supports integration of only two modalities

## Evidence
