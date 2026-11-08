---
name: spatial-continuity-metric-computation
description: Use when comparing integrated ST-SM embeddings across model configurations
  and you need a metric that scores whether the fused latent space varies smoothly
  across adjacent tissue spots rather than forming spatially discontinuous noise.
when_to_use_negative:
- Input embedding lacks spatial coordinates for spots/cells, so no neighborhood structure
  can be defined
- Data are non-spatial single-cell embeddings where spatial autocorrelation is undefined
- Evaluating a single model without any comparison configurations — the score is designed
  for relative ablation ranking
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_3170
tools:
- name: SpatialMETA
  role: Provides ConditionalVAESTSM.get_latent_embedding to extract the fused ST-SM
    embedding whose spatial continuity is scored during ablation benchmarking
  repo: https://github.com/WanluLiuLab/SpatialMETA
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-025-63915-z
    title: Integrating cross-sample and cross-modal data for spatial transcriptomics
      and metabolomics with SpatialMETA
  source_content_hash: sha256:fde9b6f1fb700e62f644e7e83249f30723f50c166f5dc0ec9355fe0131e566cd
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
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 4
---

# spatial-continuity-metric-computation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Quantify the spatial continuity (spatial smoothness/autocorrelation) of latent embeddings produced by SpatialMETA's ConditionalVAESTSM, used as one of the scoring criteria when ablating SM loss function, normalization, and decoder architecture. Continuous embeddings over tissue should preserve spatial neighborhoods, so continuity serves as a proxy for biologically plausible integration.

## When to use

Use when comparing integrated ST-SM embeddings across model configurations and you need a metric that scores whether the fused latent space varies smoothly across adjacent tissue spots rather than forming spatially discontinuous noise.

## When NOT to use

- Input embedding lacks spatial coordinates for spots/cells, so no neighborhood structure can be defined
- Data are non-spatial single-cell embeddings where spatial autocorrelation is undefined
- Evaluating a single model without any comparison configurations — the score is designed for relative ablation ranking

## Inputs

- Fused latent embedding matrix from ConditionalVAESTSM.get_latent_embedding (cells/spots x latent dimensions)
- Spot/cell spatial coordinate table from the joint ST-SM AnnData object built with spatialmeta.pp.joint_adata_sm_st
- Trained ConditionalVAESTSM models for each ablation configuration

## Outputs

- Per-configuration spatial continuity score column in the configuration-by-metric table (ablation_metrics.csv)
- Ranked comparison of spatial continuity across the ablation grid (Gaussian vs MSE loss, normalization on/off, shared vs independent decoders)

## How to apply

For each trained ConditionalVAESTSM configuration (Gaussian vs MSE loss on SM, SM normalization on/off, shared vs independent decoders), extract the fused embedding with ConditionalVAESTSM.get_latent_embedding, keeping each spot's spatial coordinates attached. Compute a spatial continuity score that measures autocorrelation of the embedding across neighboring spots (consistent with the paper's use of Moran's I for spatially aware feature quantification), then record it in the configuration-by-metric table written to ablation_metrics.csv alongside reconstruction error, marker feature recovery, and biological conservation. Rationale: a correctly integrated model should place spatially adjacent tissue structures close together in latent space; configurations that break spatial structure (e.g., MSE loss without SM normalization) will score lower. Use the metric comparatively — rank ablation configurations rather than interpreting absolute values in isolation.

## Related tools

- **SpatialMETA** (Provides ConditionalVAESTSM.get_latent_embedding to extract the fused ST-SM embedding whose spatial continuity is scored during ablation benchmarking) — https://github.com/WanluLiuLab/SpatialMETA

## Examples

```
emb = model.get_latent_embedding(adata); continuity = Moran_sI(emb, coords=adata.obsm['spatial'])  # per ablation config -> ablation_metrics.csv
```

## Evaluation signals

- Spatial continuity scores are assembled into ablation_metrics.csv with one row per configuration in the ablation grid
- The best-performing configuration should combine the Gaussian loss, SM normalization, and shared decoder per the paper's finding that these components are critical for effective vertical integration
- Continuity of the fused embedding should exceed or match that of embeddings degraded by removing SM normalization or switching to MSE loss
- Score is reported jointly with reconstruction error of ST and SM feature distributions, marker feature recovery, and biological conservation against manually annotated spatial cluster labels

## Limitations

- Benchmark metric values are only provided in source data files rather than in the main text, so expected score ranges must be pulled from source data
- Ground-truth labels for biological conservation evaluation were manually annotated from H&E histological features, not an independent gold standard, which indirectly affects confidence in continuity-based rankings
- SpatialMETA is designed for low-resolution ST and SM data; continuity metrics are not meaningful at single-cell or subcellular resolution which the tool does not support

## Evidence

- [other] Score each configuration: reconstruction error of ST and SM feature distributions, spatial continuity of the latent embedding, marker feature recovery, and biological conservation: [evidence span withheld — anchor status: not_found]
- [other] Gaussian loss and SM normalization are critical for vertical integration: [evidence span withheld — anchor status: not_found]
