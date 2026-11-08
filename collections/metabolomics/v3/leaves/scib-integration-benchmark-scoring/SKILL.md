---
name: scib-integration-benchmark-scoring
description: Use when two or more integration methods (e.g. SpatialMETA, Seurat RPCA/CCA,
  scVI, scANVI, totalVI, Stabmap, scPoli, spaVAE, PCA) have produced latent embeddings
  of the same joint ST-SM AnnData and a fair, metric-based ranking of batch correction
  vs. biological conservation is needed.
when_to_use_negative:
- You only have one method's embedding and no baselines — no comparative ranking can
  be computed
- Your ST/SM data are at single-cell or subcellular resolution — SpatialMETA and this
  benchmark setup are designed for low-resolution ST and SM data
- You need ground-truth-validated biological conservation scoring — the label key
  here is manually annotated from H&E histology, not an independent gold standard
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: scIB
  role: Computes batch-correction and biological-conservation metrics (Batch ASW,
    iLISI, PCR batch) on each embedding using the histology-derived label key
- name: SpatialMETA
  role: Method under evaluation; its ConditionalVAESTSM latent embedding is scored
    and provides cross-modality/overall integration scores via angular similarity
  repo: https://github.com/WanluLiuLab/SpatialMETA
- name: Seurat
  role: Baseline integration method (RPCA and CCA) run on identical inputs for vertical
    and horizontal integration comparison
- name: scVI
  role: Baseline horizontal integration method scored against SpatialMETA
- name: scANVI
  role: Baseline horizontal integration method scored against SpatialMETA
- name: totalVI
  role: Baseline vertical integration method scored against SpatialMETA
- name: Stabmap
  role: Baseline vertical integration method scored against SpatialMETA
- name: scPoli
  role: Baseline horizontal integration method scored against SpatialMETA
- name: spaVAE
  role: Baseline spatial integration method scored against SpatialMETA
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-025-63915-z
    title: Integrating cross-sample and cross-modal data for spatial transcriptomics
      and metabolomics with SpatialMETA
  source_content_hash: sha256:73be97ccd888f0be593a4441e2881b5bb748a5970009582a34893343bf958a70
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
  - Stabmap
  - scANVI
  - scIB
  - scPoli
  - scVI
  - spaVAE
  - totalVI
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8461538461538461
n_claims: 13
---

# scib-integration-benchmark-scoring

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmark vertical and horizontal integration methods for spatial transcriptomics (ST) and spatial metabolomics (SM) data by scoring every method's fused embedding with scIB metrics (Batch ASW, iLISI, PCR batch) plus SpatialMETA's cross-modality, cross-sample and overall integration scores, then ranking methods by overall integration performance.

## When to use

Use when two or more integration methods (e.g. SpatialMETA, Seurat RPCA/CCA, scVI, scANVI, totalVI, Stabmap, scPoli, spaVAE, PCA) have produced latent embeddings of the same joint ST-SM AnnData and a fair, metric-based ranking of batch correction vs. biological conservation is needed.

## When NOT to use

- You only have one method's embedding and no baselines — no comparative ranking can be computed
- Your ST/SM data are at single-cell or subcellular resolution — SpatialMETA and this benchmark setup are designed for low-resolution ST and SM data
- You need ground-truth-validated biological conservation scoring — the label key here is manually annotated from H&E histology, not an independent gold standard

## Inputs

- Joint ST-SM AnnData objects (one per sample, e.g. five ccRCC IM2 samples) after KNN reassignment and removal of batch-biased SVGs/SVMs
- Latent embeddings from each integration method (SpatialMETA ConditionalVAESTSM, Seurat RPCA/CCA, totalVI, Stabmap, scVI, scANVI, scPoli, spaVAE, PCA) on identical inputs
- Manually annotated histology-derived label key from H&E staining for biological conservation evaluation

## Outputs

- Comparison table of metric values for all methods and metrics (e.g. fig4d_integration_benchmark.csv)
- Per-method scIB scores: Batch ASW, iLISI, PCR batch
- SpatialMETA cross-modality, cross-sample, overall integration scores and modality contribution (angular similarity between embeddings)
- Ranked method list by overall integration performance

## How to apply

Run each integration method (SpatialMETA's ConditionalVAESTSM via get_latent_embedding, plus baselines with default recommended settings) on the identical processed joint ST-SM AnnData objects — the ones produced by joint_adata_sm_st with batch-biased SVGs/SVMs removed (fold change > min_logfc=3, expression >= min_frac=0.8 across >= 2 samples). Score every embedding with scIB, passing manually annotated histology-derived labels (from H&E features) as the label key and computing Batch ASW, iLISI and PCR batch. Additionally compute SpatialMETA's cross-modality, cross-sample and overall integration scores and modality contribution via angular similarity between embeddings. Aggregate all metric values into a comparison table (e.g. fig4d_integration_benchmark.csv), rank methods by overall performance, and verify the table contains values for every method. Rationale: uniform inputs and a fixed label key make the scIB batch-correction/biological-conservation trade-off comparable across non-spatial single-cell and spatial multi-omics tools.

## Related tools

- **scIB** (Computes batch-correction and biological-conservation metrics (Batch ASW, iLISI, PCR batch) on each embedding using the histology-derived label key)
- **SpatialMETA** (Method under evaluation; its ConditionalVAESTSM latent embedding is scored and provides cross-modality/overall integration scores via angular similarity) — https://github.com/WanluLiuLab/SpatialMETA
- **Seurat** (Baseline integration method (RPCA and CCA) run on identical inputs for vertical and horizontal integration comparison)
- **scVI** (Baseline horizontal integration method scored against SpatialMETA)
- **scANVI** (Baseline horizontal integration method scored against SpatialMETA)
- **totalVI** (Baseline vertical integration method scored against SpatialMETA)
- **Stabmap** (Baseline vertical integration method scored against SpatialMETA)
- **scPoli** (Baseline horizontal integration method scored against SpatialMETA)
- **spaVAE** (Baseline spatial integration method scored against SpatialMETA)

## Examples

```
import scib; scores = scib.metrics.metrics(adata, adata_emb, batch_key='batch', label_key='histology_label', asw_batch=True, ilisi=True, pcr_batch=True)
```

## Evaluation signals

- The output CSV (e.g. fig4d_integration_benchmark.csv) contains metric values for every compared method (SpatialMETA, Seurat RPCA/CCA, scVI, scANVI, totalVI, Stabmap, scPoli, spaVAE, PCA)
- All methods were run on identical input AnnData objects with the same batch-biased feature filtering (min_logfc=3, min_frac=0.8, min_samples=2)
- scIB was called with the manually annotated histology-derived label key, and Batch ASW, iLISI and PCR batch are all present for each method
- SpatialMETA's overall integration score ranks at or near the top, consistent with the reported superior performance versus published non-spatial single-cell and spatial multi-omics tools
- Benchmark metric values should match the values deposited in the paper's source data files

## Limitations

- Ground-truth labels for biology conservation evaluation were manually annotated from H&E histological features, not an independent gold standard
- Detailed benchmark metric values are only provided in source data files rather than in the main text
- Horizontal integration was validated only on a limited number of samples from a single study

## Evidence

- [discussion] scIB label key from H&E: [evidence span withheld — anchor status: not_found]
