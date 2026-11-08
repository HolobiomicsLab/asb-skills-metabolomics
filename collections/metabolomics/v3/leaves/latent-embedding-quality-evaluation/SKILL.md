---
name: latent-embedding-quality-evaluation
description: Use when you have latent embeddings (e.g., from ConditionalVAESM, Seurat
  CCA/RPCA, scVI, scANVI, scPoli, spaVAE, or PCA) computed on multiple spatial samples
  and need to quantify which integration balances batch correction and biology conservation.
when_to_use_negative:
- When ground-truth labels are unavailable or cannot be manually annotated from H&E
  histological features, since the biology-conservation metric depends on them
- When evaluating embeddings from single-sample data with no batch structure — batch-correction
  metrics are undefined or trivially satisfied
- When the data are single-cell or subcellular resolution, as SpatialMETA and its
  benchmark are designed for low-resolution ST and SM data
edam_operation: http://edamontology.org/operation_2935
edam_topics:
- http://edamontology.org/topic_3934
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_0091
tools:
- name: SpatialMETA
  role: Provides the CVAE-based integrated latent embeddings being evaluated, and
    is the method benchmarked for superior overall performance
  repo: https://github.com/WanluLiuLab/SpatialMETA
- name: scIB
  role: Supplies the continuity, marker (biology conservation), and batch-correction
    metric framework with the 'label key' parameter for ground-truth labels
- name: scVI
  role: Horizontal integration baseline whose embeddings are scored on the same filtered
    matrices
- name: scANVI
  role: Horizontal integration baseline whose embeddings are scored on the same filtered
    matrices
- name: scPoli
  role: Horizontal integration baseline; a top performer alongside SpatialMETA in
    the SM-only ccRCC benchmark
- name: Seurat
  role: CCA/RPCA horizontal integration baseline whose embeddings are scored against
    SpatialMETA
- name: spaVAE
  role: Spatial multi-omics baseline method included in the embedding benchmark comparison
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-025-63915-z
    title: Integrating cross-sample and cross-modal data for spatial transcriptomics
      and metabolomics with SpatialMETA
  source_content_hash: sha256:57d9b07063305cc48de6e6f54a2c2048445e4674cac3503fd4e7be567112cb43
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
  - Seurat
  - SpatialMETA
  - scANVI
  - scIB
  - scPoli
  - scVI
  - spaVAE
  edam_operation: http://edamontology.org/operation_2935
  edam_topics:
  - http://edamontology.org/topic_3934
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7857142857142857
n_claims: 14
---

# Latent Embedding Quality Evaluation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Scores integrated latent embeddings from spatial transcriptomics/metabolomics integration methods using scIB-style continuity, marker (biology conservation), and batch-correction metrics, after removing batch-biased features. This enables quantitative benchmarking of integration tools such as SpatialMETA against horizontal/vertical integration baselines.

## When to use

Use when you have latent embeddings (e.g., from ConditionalVAESM, Seurat CCA/RPCA, scVI, scANVI, scPoli, spaVAE, or PCA) computed on multiple spatial samples and need to quantify which integration best balances batch correction and biology conservation.

## When NOT to use

- When ground-truth labels are unavailable or cannot be manually annotated from H&E histological features, since the biology-conservation metric depends on them
- When evaluating embeddings from single-sample data with no batch structure — batch-correction metrics are undefined or trivially satisfied
- When the data are single-cell or subcellular resolution, as SpatialMETA and its benchmark are designed for low-resolution ST and SM data

## Inputs

- Latent embeddings (integrated low-dimensional representations) from SpatialMETA ConditionalVAESM.get_latent_embedding or baseline methods (Seurat CCA/RPCA, scVI, scANVI, scPoli, spaVAE, PCA)
- Filtered and QC'd AnnDataSM metabolite intensity matrices (post calculate_qc_metrics_sm, filter_cells_sm, filter_metabolites_sm, merge_sm_pos_neg)
- H&E-derived manually annotated ground-truth spatial cluster labels (scIB 'label key')
- Batch/sample covariate annotations per spot

## Outputs

- Per-method scIB metric values for continuity, marker (biology conservation), and batch-correction categories
- Method ranking across metric categories (e.g., sm_horizontal_integration_benchmark_metrics.csv mirroring Supplementary Fig. 11)
- Identification of top-performing integration methods (e.g., scPoli, SpatialMETA, Seurat CCA/RPCA for SM-only ccRCC integration)

## How to apply

First remove batch-biased SVGs/SVMs — features with fold change > min_logfc (default 3) expressed in >= min_frac (default 0.8) of spots shared across min_samples (default 2) — so they do not inflate metric scores. Then compute scIB-style metrics on each method's embedding across three categories: continuity score, marker score (biology conservation, using H&E-annotated ground-truth labels passed via the scIB 'label key' parameter), and batch-correction metrics. In the SM-only ccRCC horizontal-integration benchmark, SpatialMETA's ConditionalVAESM embeddings (obtained via get_latent_embedding) were compared against Seurat CCA/RPCA, scVI, scANVI, scPoli, spaVAE, and PCA on the same filtered metabolite matrices. Assemble per-method metric values into a comparison table (mirroring Supplementary Fig. 11) and rank methods per category; a method is judged superior when it ranks at or near the top across all three metric categories rather than excelling in only one. Detailed metric values are stored in source data files rather than reported in the main text.

## Related tools

- **SpatialMETA** (Provides the CVAE-based integrated latent embeddings being evaluated, and is the method benchmarked for superior overall performance) — https://github.com/WanluLiuLab/SpatialMETA
- **scIB** (Supplies the continuity, marker (biology conservation), and batch-correction metric framework with the 'label key' parameter for ground-truth labels)
- **scVI** (Horizontal integration baseline whose embeddings are scored on the same filtered matrices)
- **scANVI** (Horizontal integration baseline whose embeddings are scored on the same filtered matrices)
- **scPoli** (Horizontal integration baseline; a top performer alongside SpatialMETA in the SM-only ccRCC benchmark)
- **Seurat** (CCA/RPCA horizontal integration baseline whose embeddings are scored against SpatialMETA)
- **spaVAE** (Spatial multi-omics baseline method included in the embedding benchmark comparison)

## Examples

```
from spatialmeta.model import ConditionalVAESM; emb = model.get_latent_embedding(adata_sm); score_embeddings({'SpatialMETA': emb, 'scPoli': emb_scpoli, 'Seurat': emb_seurat}, labels_key='he_annotation', remove_batch_biased=True, min_logfc=3, min_frac=0.8)
```

## Evaluation signals

- The benchmark metrics table must contain rows for all seven methods (SpatialMETA plus Seurat CCA/RPCA, scVI, scANVI, scPoli, spaVAE, PCA) covering continuity, marker, and batch-correction categories
- SpatialMETA should rank at or near the top across metric categories, consistent with its reported superior overall performance; for SM-only ccRCC integration, scPoli, SpatialMETA, and Seurat CCA/RPCA are expected top performers
- Batch-biased SVMs (fold change > 3, expressed in >= 0.8 of spots across >= 2 samples) must be removed before scoring — verify no such features remain in the evaluation set
- Biology-conservation scores must be computed against the H&E-annotated manual ground-truth labels, not unsupervised cluster labels
- Per-method metric values should match the source data files / Supplementary Fig. 11 when reproducing the published benchmark

## Limitations

- Ground-truth labels for biology conservation are manually annotated from H&E histological features rather than an independent gold standard
- Detailed benchmark metric values are only provided in source data files, not in the main text
- Horizontal integration was validated only on a limited number of samples from a single study, limiting generalizability of the evaluation
- Batch-biased feature removal thresholds (min_logfc=3, min_frac=0.8) may need tuning per dataset; miscalibration could bias metric comparisons

## Evidence

- [methods] scIB-style continuity, marker and batch-correction metrics, removing batch-biased SVMs (fold change > min_logfc default 3, expression in >= min_frac default 0.8 of spots) beforehand: [evidence span withheld — anchor status: not_found]
- [discussion] The ground-truth labels used for evaluation (specified via the "label key" parameter in scIB) were manually annotated based on histological features identified through H&E staining: [evidence span withheld — anchor status: not_found]
- [discussion] Genes or metabolites with a fold change greater than the threshold ("min_logfc", default is 3) were defined as batch-biased SVGs and SVMs: [evidence span withheld — anchor status: not_found]
- [results] for SM-only horizontal integration of ccRCC samples, scPoli, SpatialMETA, and Seurat CCA/RPCA were the top performers: [evidence span withheld — anchor status: not_found]
