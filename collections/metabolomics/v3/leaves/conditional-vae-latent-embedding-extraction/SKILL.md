---
name: conditional-vae-latent-embedding-extraction
description: Use when you have aligned, KNN-reassigned joint ST-SM AnnData (batch-biased
  SVGs/SVMs already removed) and need a single fused embedding that corrects batch
  effects across samples while fusing transcriptomic and metabolomic modalities.
when_to_use_negative:
- Data are at single-cell or subcellular resolution — the article states SpatialMETA
  is designed for low-resolution ST and SM data only
- More than two modalities must be integrated — SpatialMETA currently supports only
  two (ST and SM); >2-modality vertical integration is future work
- SM and ST spots have not been aligned/reassigned yet — the CVAE expects the joint,
  KNN-reassigned AnnData, not raw MSI matrices
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3172
tools:
- name: SpatialMETA (ConditionalVAESTSM)
  role: Conditional VAE whose fit/train step learns the shared latent space and whose
    get_latent_embedding / get_normalized_expression produce the fused outputs
  repo: https://github.com/WanluLiuLab/SpatialMETA
- name: scVI
  role: Baseline non-spatial horizontal-integration embedding run on identical inputs
    for scIB comparison against the ConditionalVAESTSM embedding
- name: scANVI
  role: Label-aware baseline embedding used in the same benchmark to rank against
    SpatialMETA's latent representation
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-025-63915-z
    title: Integrating cross-sample and cross-modal data for spatial transcriptomics
      and metabolomics with SpatialMETA
  source_content_hash: sha256:b4ca2763a3ca5a942367ea76b5bb334d9cc11754d031ee2a0ab10f93df3cc6f5
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
  - SpatialMETA (ConditionalVAESTSM)
  - Stabmap
  - scANVI
  - scPoli
  - scVI
  - spaVAE
  - totalVI
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 14
---

# conditional-vae-latent-embedding-extraction

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Trains SpatialMETA's ConditionalVAESTSM conditional variational autoencoder on preprocessed joint ST-SM AnnData to learn a shared latent space, then extracts the fused latent embedding (and optionally normalized expression) for downstream integration benchmarking, clustering, and modality-contribution analysis.

## When to use

Use when you have aligned, KNN-reassigned joint ST-SM AnnData (batch-biased SVGs/SVMs already removed) and need a single fused embedding that corrects batch effects across samples while fusing transcriptomic and metabolomic modalities.

## When NOT to use

- Data are at single-cell or subcellular resolution — the article states SpatialMETA is designed for low-resolution ST and SM data only
- More than two modalities must be integrated — SpatialMETA currently supports only two (ST and SM); >2-modality vertical integration is future work
- SM and ST spots have not been aligned/reassigned yet — the CVAE expects the joint, KNN-reassigned AnnData, not raw MSI matrices

## Inputs

- Joint ST-SM AnnData object after spot alignment and KNN reassignment (spatialmeta.pp.joint_adata_sm_st pipeline)
- AnnData filtered to remove batch-biased SVGs/SVMs (fold change > 3, expression >= 0.8 of spots across >= 2 samples)
- Batch/sample key column identifying the five ccRCC IM2 samples (or equivalent multi-sample set)
- Manually annotated histology-derived label key (from H&E features) for downstream scIB evaluation

## Outputs

- Fused latent embedding matrix from get_latent_embedding (shared cross-modal, cross-sample representation)
- Normalized expression from get_normalized_expression when downstream tasks need reconstructed values
- Embedding values feeding scIB metrics (Batch ASW, iLISI, PCR batch) and angular-similarity modality contribution scores
- Ranked comparison table (e.g. fig4d_integration_benchmark.csv) when run in benchmark mode

## How to apply

Load the processed joint ST-SM AnnData produced by upstream alignment and KNN reassignment, ensuring batch-biased SVGs/SVMs have been removed using the article defaults (fold change > min_logfc=3, expression >= min_frac=0.8 across >= min_samples=2 samples). Instantiate and fit spatialmeta.model.ConditionalVAESTSM, which simultaneously integrates batch information and the joint features, using tailored decoders and loss functions per modality to handle SM's continuous mass-spectrometry intensities versus ST sequencing counts. Extract the fused representation with get_latent_embedding and, where downstream tasks need expression, get_normalized_expression. Validate the embedding against baselines (scVI, scANVI, scPoli, spaVAE, PCA, Seurat RPCA/CCA, totalVI, Stabmap) run on identical inputs, scoring with scIB metrics (Batch ASW, iLISI, PCR batch) plus cross-modality/cross-sample/overall integration scores and angular similarity between embeddings for modality contribution.

## Related tools

- **SpatialMETA (ConditionalVAESTSM)** (Conditional VAE whose fit/train step learns the shared latent space and whose get_latent_embedding / get_normalized_expression produce the fused outputs) — https://github.com/WanluLiuLab/SpatialMETA
- **scVI** (Baseline non-spatial horizontal-integration embedding run on identical inputs for scIB comparison against the ConditionalVAESTSM embedding)
- **scANVI** (Label-aware baseline embedding used in the same benchmark to rank against SpatialMETA's latent representation)

## Examples

```
from spatialmeta.model import ConditionalVAESTSM; model = ConditionalVAESTSM(adata); model.fit(adata, batch_key='sample'); emb = model.get_latent_embedding(adata)
```

## Evaluation signals

- Latent embedding is obtained for all spots of all samples with finite values and no missing samples in the output matrix
- scIB scores computed with the histology-derived label key: Batch ASW, iLISI and PCR batch indicate batch mixing without loss of biological conservation
- SpatialMETA's cross-modality, cross-sample and overall integration scores place it at or near the top versus Seurat RPCA/CCA, scVI, scANVI, totalVI, scPoli, spaVAE and PCA, consistent with the Fig. 4d finding
- Angular similarity between modality embeddings yields interpretable, nonzero contributions from both ST and SM
- Benchmark CSV contains metric values for every method (validation per the workflow: table must contain all methods and SpatialMETA ranks at/near the top overall)

## Limitations

- SpatialMETA is designed for low-resolution ST and SM data; single-cell or subcellular resolution is unsupported and precise cell segmentation remains unimplemented
- Only two modalities can be integrated; vertical integration involving more than two modalities is not supported
- Ground-truth labels for biological-conservation evaluation were manually annotated from H&E histological features, not an independent gold standard
- Horizontal integration was demonstrated on a limited number of samples from a single study; detailed benchmark metric values are only in source data files

## Evidence

- [discussion] Genes or metabolites with a fold change greater than the threshold ("min_logfc", default is 3) were defined as batch-biased SVGs and SVMs: [evidence span withheld — anchor status: not_found]
