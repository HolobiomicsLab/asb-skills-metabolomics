---
name: biological-conservation-scoring
description: Use when evaluating a spatial multi-omics integration result (ST and/or
  SM embeddings) and you need to confirm the embedding preserves biological structure,
  e.g. spatial cluster identity from H&E annotation.
when_to_use_negative:
- No manually annotated spatial cluster labels exist for the tissue — the biology-conservation
  metric requires an H&E-derived label key for scIB
- The goal is quantitative reconstruction accuracy of SM intensity distributions rather
  than preservation of biological identity
- Data are single-cell or subcellular resolution ST/SM — SpatialMETA and this scoring
  workflow are designed for low-resolution spot-level data
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3934
- http://edamontology.org/topic_0203
tools:
- name: SpatialMETA
  role: Provides the integrated latent embeddings (ConditionalVAESM/ConditionalVAESTSM.get_latent_embedding)
    and the preprocessed AnnData that are scored for biological conservation
  repo: https://github.com/WanluLiuLab/SpatialMETA
- name: scIB
  role: Supplies the scIB-style continuity, marker (biology conservation) and batch-correction
    metrics, with the ground-truth label key parameter
- name: DestVI
  role: Deconvolution of ST with a ccRCC scRNA-seq reference to support manual annotation
    of ground-truth spatial cluster labels
- name: Seurat
  role: Benchmarked baseline (CCA/RPCA) whose embedding is scored with the same biology-conservation
    metrics for comparison
- name: scPoli
  role: Benchmarked baseline included in the horizontal-integration comparison against
    the conservation score
provenance:
  source_task_ids:
  - task_004
  - task_003
  source_papers:
  - doi: 10.1038/s41467-025-63915-z
    title: Integrating cross-sample and cross-modal data for spatial transcriptomics
      and metabolomics with SpatialMETA
  source_content_hash: sha256:1ed0d2d59ecaa33fb6c4f4354c131ddfee0a6980ba6c58201e96e4f393da0e59
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
  - DestVI
  - Seurat
  - SpatialMETA
  - scANVI
  - scIB
  - scPoli
  - scVI
  - spaVAE
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3934
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# biological-conservation-scoring

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Scores integrated ST-SM embeddings for biological conservation by comparing learned spatial clusters against manually annotated H&E-derived ground-truth labels, plus marker-feature recovery, as used to benchmark SpatialMETA against horizontal-integration baselines.

## When to use

Use when evaluating a spatial multi-omics integration result (ST and/or SM embeddings) and you need to confirm the embedding preserves biological structure, e.g. spatial cluster identity from H&E annotation.

## When NOT to use

- No manually annotated spatial cluster labels exist for the tissue — the biology-conservation metric requires an H&E-derived label key for scIB
- The goal is quantitative reconstruction accuracy of SM intensity distributions rather than preservation of biological identity
- Data are single-cell or subcellular resolution ST/SM — SpatialMETA and this scoring workflow are designed for low-resolution spot-level data

## Inputs

- Latent embedding matrix (AnnData obsm) from an integration model, e.g. ConditionalVAESM.get_latent_embedding output
- AnnData with spatial cluster labels manually annotated from H&E staining and marker genes (label key for scIB)
- Batch-bias-filtered feature set (SVGs/SVMs with fold change > 3, expression in >= 0.8 of spots removed)

## Outputs

- Per-method/per-configuration biology-conservation (marker) metric values, alongside continuity and batch-correction scores
- Configuration-by-metric benchmark table (e.g. sm_horizontal_integration_benchmark_metrics.csv or ablation_metrics.csv)

## How to apply

First remove batch-biased SVGs/SVMs — features with fold change > min_logfc (default 3) expressed in >= min_frac (default 0.8) of spots shared across min_samples (default 2) — so batch signatures do not inflate the score. Then take the latent embedding (e.g. from ConditionalVAESM.get_latent_embedding or ConditionalVAESTSM.get_latent_embedding) and pass it to scIB-style scoring with a 'label key' pointing to ground-truth spatial cluster labels that were manually annotated from H&E histological features using marker gene expression and DestVI deconvolution with a ccRCC scRNA-seq reference. Combine the biology-conservation (marker) score with continuity and batch-correction metrics into a per-method table; in the SpatialMETA ccRCC benchmark the top performers (scPoli, SpatialMETA, Seurat CCA/RPCA) were compared across these three metric categories. A configuration is judged correctly applied when metric values for all methods and all metric categories are present and the reference method ranks consistently with the published benchmark (Supplementary Fig. 11).

## Related tools

- **SpatialMETA** (Provides the integrated latent embeddings (ConditionalVAESM/ConditionalVAESTSM.get_latent_embedding) and the preprocessed AnnData that are scored for biological conservation) — https://github.com/WanluLiuLab/SpatialMETA
- **scIB** (Supplies the scIB-style continuity, marker (biology conservation) and batch-correction metrics, with the ground-truth label key parameter)
- **DestVI** (Deconvolution of ST with a ccRCC scRNA-seq reference to support manual annotation of ground-truth spatial cluster labels)
- **Seurat** (Benchmarked baseline (CCA/RPCA) whose embedding is scored with the same biology-conservation metrics for comparison)
- **scPoli** (Benchmarked baseline included in the horizontal-integration comparison against the conservation score)

## Examples

```
adata = model.get_latent_embedding(); scores = scib.metrics.metrics(adata, adata_int, 'batch', 'spatial_cluster_label', silhouette_=True, nmi_=True, ari_=True)
```

## Evaluation signals

- The benchmark table contains metric rows for all seven compared methods (SpatialMETA plus Seurat CCA/RPCA, scVI, scANVI, scPoli, spaVAE, PCA) covering continuity, marker and batch-correction categories
- Batch-biased SVMs (fold change > min_logfc default 3, expressed in >= min_frac default 0.8 of spots) were removed before scoring
- The ground-truth label key passed to scIB corresponds to manually annotated H&E-based spatial clusters, not batch or sample identifiers
- For the ccRCC SM-only benchmark, SpatialMETA ranks at or near the top on continuity, marker and batch-correction metrics, mirroring Supplementary Fig. 11
- In ablation use, configurations using Gaussian loss and SM normalization should score higher on biology conservation than their ablated counterparts

## Limitations

- Ground-truth labels for biology conservation were manually annotated from H&E histological features, not an independent gold standard
- Detailed benchmark metric values are provided only in source data files rather than the main text
- Horizontal integration was validated only on a limited number of samples from a single study

## Evidence

- [discussion] The ground-truth labels used for evaluation (specified via the "label key" parameter in scIB) were manually annotated based on histological features identified through H&E staining: [evidence span withheld — anchor status: not_found]
- [discussion] Genes or metabolites with a fold change greater than the threshold ("min_logfc", default is 3) were defined as batch-biased SVGs and SVMs: [evidence span withheld — anchor status: not_found]
- [discussion] we manually annotated the spatial clusters according to marker gene expression: "Referring to the deconvolution prediction results, we manually annotated the spatial clusters according to marker gene expression"
- [other] scPoli, SpatialMETA, and Seurat CCA/RPCA were the top performers, outperforming the other benchmarked methods across continuity score, marker score, and batch correction metrics.: [evidence span withheld — anchor status: not_found]
- [readme] # github.com__WanluLiuLab__SpatialMETA: "spatialMETA is a method for integrating spatial multi-omics data. SMOI aligns ST and SM to a unified resolution, integrates single or multiple sample data"
