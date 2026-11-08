---
name: spatial-metabolomics-horizontal-integration
description: Use when multiple spatial metabolomics samples from the same tissue type
  (e.g., ccRCC MSI metabolite intensity matrices) must be integrated into a shared
  latent space and benchmarked against existing horizontal integration methods.
when_to_use_negative:
- The task is vertical (cross-modal ST+SM) integration rather than SM-only horizontal
  integration across samples
- Data are at single-cell or subcellular resolution — SpatialMETA is designed for
  low-resolution ST and SM data
- More than two modalities must be integrated; SpatialMETA currently only supports
  integration of two modalities
edam_operation: http://edamontology.org/operation_2935
edam_topics:
- http://edamontology.org/topic_3817
- http://edamontology.org/topic_3308
tools:
- name: SpatialMETA
  role: ConditionalVAESM model trains the SM-only horizontal integration embedding
    via spatialmeta.model.ConditionalVAESM.fit
  repo: https://github.com/WanluLiuLab/SpatialMETA
- name: Seurat
  role: Provides CCA/RPCA baseline horizontal-integration embeddings run on the same
    filtered SM matrices
- name: scVI
  role: Baseline horizontal integration method benchmarked against SpatialMETA
- name: scANVI
  role: Baseline horizontal integration method benchmarked against SpatialMETA
- name: scPoli
  role: Baseline horizontal integration method; among top performers alongside SpatialMETA
    and Seurat CCA/RPCA
- name: spaVAE
  role: Baseline spatial multi-omics horizontal integration method benchmarked against
    SpatialMETA
- name: scIB
  role: Supplies continuity, marker (biology conservation), and batch-correction metrics
    used to score all embeddings
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-025-63915-z
    title: Integrating cross-sample and cross-modal data for spatial transcriptomics
      and metabolomics with SpatialMETA
  source_content_hash: sha256:d6dd65261dfb19a70c308717c1d796af56706222a88feb1891a0df6cc35aaac9
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
  - http://edamontology.org/topic_3817
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6923076923076923
n_claims: 13
---

# spatial-metabolomics-horizontal-integration

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmark SM-only horizontal integration of spatial metabolomics (SM) samples using SpatialMETA's ConditionalVAESM against non-spatial baselines (Seurat CCA/RPCA, scVI, scANVI, scPoli, spaVAE, PCA), scoring embeddings with scIB-style continuity, marker, and batch-correction metrics.

## When to use

Use when multiple spatial metabolomics samples from the same tissue type (e.g., ccRCC MSI metabolite intensity matrices) must be integrated into a shared latent space and benchmarked against existing horizontal integration methods.

## When NOT to use

- The task is vertical (cross-modal ST+SM) integration rather than SM-only horizontal integration across samples
- Data are at single-cell or subcellular resolution — SpatialMETA is designed for low-resolution ST and SM data
- More than two modalities must be integrated; SpatialMETA currently only supports integration of two modalities

## Inputs

- SM metabolite intensity matrices from multiple samples of one tissue type (e.g., ccRCC), as AnnDataSM objects loadable via spatialmeta.data.load_adata
- Positive and negative ion-mode SM acquisitions for merging via merge_sm_pos_neg
- H&E-annotated ground-truth spatial labels for biology-conservation scoring
- Filtered feature matrices shared across all benchmarked methods

## Outputs

- Integrated latent embeddings per sample from ConditionalVAESM.get_latent_embedding
- Normalized SM expression via ConditionalVAESM.get_normalized_expression
- Per-method scIB metric values (continuity, marker, batch correction) assembled into sm_horizontal_integration_benchmark_metrics.csv, mirroring Supplementary Fig. 11

## How to apply

Load each sample's metabolite intensity matrix into an AnnDataSM object with spatialmeta.data.load_adata, then preprocess with spatialmeta.pp.calculate_qc_metrics_sm, filter_cells_sm, and filter_metabolites_sm, merging positive/negative ion-mode acquisitions via merge_sm_pos_neg where applicable. Train the SM-only integration model with spatialmeta.model.ConditionalVAESM.fit to learn a shared latent space across samples, and extract embeddings with ConditionalVAESM.get_latent_embedding and get_normalized_expression. Run the baseline horizontal-integration methods (Seurat CCA/RPCA, scVI, scANVI, scPoli, spaVAE, PCA) on the same filtered matrices. Before scoring, remove batch-biased SVMs using the min_logfc (default 3) and min_frac (default 0.8 of spots) thresholds, then compute scIB-style continuity, marker (biology conservation against H&E-annotated ground-truth labels), and batch-correction metrics and assemble the per-method table.

## Related tools

- **SpatialMETA** (ConditionalVAESM model trains the SM-only horizontal integration embedding via spatialmeta.model.ConditionalVAESM.fit) — https://github.com/WanluLiuLab/SpatialMETA
- **Seurat** (Provides CCA/RPCA baseline horizontal-integration embeddings run on the same filtered SM matrices)
- **scVI** (Baseline horizontal integration method benchmarked against SpatialMETA)
- **scANVI** (Baseline horizontal integration method benchmarked against SpatialMETA)
- **scPoli** (Baseline horizontal integration method; among top performers alongside SpatialMETA and Seurat CCA/RPCA)
- **spaVAE** (Baseline spatial multi-omics horizontal integration method benchmarked against SpatialMETA)
- **scIB** (Supplies continuity, marker (biology conservation), and batch-correction metrics used to score all embeddings)

## Examples

```
python -c "import spatialmeta; adata_sm = spatialmeta.data.load_Vicari_2024_msi(); spatialmeta.pp.calculate_qc_metrics_sm(adata_sm)"
```

## Evaluation signals

- The output table contains metric rows for all seven methods (SpatialMETA plus Seurat CCA/RPCA, scVI, scANVI, scPoli, spaVAE, PCA) covering continuity, marker, and batch-correction categories
- SpatialMETA ranks at or near the top of the metrics, consistent with its reported superior overall performance
- Batch-biased SVMs are removed before scoring: fold change > min_logfc (default 3) and expression in >= min_frac (default 0.8) of spots
- Marker/biology-conservation scores use H&E-annotated ground-truth labels via the scIB 'label key' parameter
- Embeddings are extracted with ConditionalVAESM.get_latent_embedding after ConditionalVAESM.fit converges on the shared latent space

## Limitations

- Horizontal integration was validated only on a limited number of samples from a single study
- Ground-truth labels for biology conservation were manually annotated from H&E staining, not an independent gold standard
- Detailed benchmark metric values are only provided in source data files rather than in the main text

## Evidence

- [discussion] Genes or metabolites with a fold change greater than the threshold ("min_logfc", default is 3) were defined as batch-biased SVGs and SVMs: [evidence span withheld — anchor status: not_found]
