---
name: modality-contribution-angular-similarity-analysis
description: Use when a fused ST-SM latent embedding has been trained with SpatialMETA's
  ConditionalVAESTSM and you need to quantify each modality's contribution to the
  shared representation, e.g. during the Fig. 4d ccRCC IM2 integration benchmark.
when_to_use_negative:
- The embedding was produced by a baseline method (Seurat RPCA/CCA, scVI, scANVI,
  totalVI, scPoli, spaVAE, PCA) that does not expose separate ST and SM modality embeddings
  — use scIB metrics instead
- The data are single-cell or subcellular resolution ST/SM — SpatialMETA's angular-similarity
  interpretability is designed for low-resolution spot-level ST and SM data
- More than two modalities are being integrated — SpatialMETA currently supports only
  two modalities (ST and SM)
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3172
tools:
- name: SpatialMETA
  role: Provides ConditionalVAESTSM, get_latent_embedding and the model design that
    enables angular-similarity-based modality contribution calculation
  repo: github.com/WanluLiuLab/SpatialMETA
- name: scVI
  role: Horizontal integration baseline against which SpatialMETA's modality-contribution-weighted
    integration scores are compared
- name: spaVAE
  role: Spatial horizontal integration baseline in the same benchmark table
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-025-63915-z
    title: Integrating cross-sample and cross-modal data for spatial transcriptomics
      and metabolomics with SpatialMETA
  source_content_hash: sha256:c942cea4da4611700513fbf53f15bdbbf2b40523d831cc5e2e40798fd1aa1565
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/WanluLiuLab/SpatialMETA
  tools:
  - Seurat
  - SpatialMETA
  - Stabmap
  - scANVI
  - scPoli
  - scVI
  - spaVAE
  - totalVI
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# Modality contribution angular similarity analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Quantifies how much the spatial transcriptomics (ST) versus spatial metabolomics (SM) modalities contribute to SpatialMETA's ConditionalVAESTSM fused latent embedding by computing angular similarity between modality-specific embeddings, improving interpretability of the cross-modal integration.

## When to use

Use when a fused ST-SM latent embedding has been trained with SpatialMETA's ConditionalVAESTSM and you need to quantify each modality's contribution to the shared representation, e.g. during the Fig. 4d ccRCC IM2 integration benchmark.

## When NOT to use

- The embedding was produced by a baseline method (Seurat RPCA/CCA, scVI, scANVI, totalVI, scPoli, spaVAE, PCA) that does not expose separate ST and SM modality embeddings — use scIB metrics instead
- The data are single-cell or subcellular resolution ST/SM — SpatialMETA's angular-similarity interpretability is designed for low-resolution spot-level ST and SM data
- More than two modalities are being integrated — SpatialMETA currently supports only two modalities (ST and SM)

## Inputs

- Processed joint ST-SM AnnData object for one or more ccRCC IM2 samples (post KNN SM-to-ST reassignment, batch-biased features removed)
- Fitted spatialmeta.model.ConditionalVAESTSM model with per-modality latent embeddings

## Outputs

- Per-spot or per-sample angular similarity values quantifying ST vs SM modality contribution to the fused latent embedding
- Modality-contribution values contributing to SpatialMETA's overall integration score in the benchmark comparison table (e.g. fig4d_integration_benchmark.csv)

## How to apply

Train the ConditionalVAESTSM conditional variational autoencoder on the joint ST-SM AnnData (built via spatialmeta.pp.joint_adata_sm_st with batch-biased SVGs/SVMs removed using fold change > min_logfc default 3 and expression >= min_frac default 0.8 across >= 2 samples). Extract the fused latent embedding with ConditionalVAESTSM.get_latent_embedding and compute the angular similarity between the ST and SM modality-specific embeddings, as enabled by the model's tailored decoders and loss functions for modality fusion, batch effect correction and biological conservation. Report the angular similarity per spot/sample alongside the scIB metrics (Batch ASW, iLISI, PCR batch) and cross-modality, cross-sample and overall integration scores, then aggregate into the comparison table (e.g. fig4d_integration_benchmark.csv) to rank SpatialMETA against Seurat RPCA/CCA, scVI, scANVI, totalVI, scPoli, spaVAE and PCA baselines.

## Related tools

- **SpatialMETA** (Provides ConditionalVAESTSM, get_latent_embedding and the model design that enables angular-similarity-based modality contribution calculation) — github.com/WanluLiuLab/SpatialMETA
- **scVI** (Horizontal integration baseline against which SpatialMETA's modality-contribution-weighted integration scores are compared)
- **spaVAE** (Spatial horizontal integration baseline in the same benchmark table)

## Examples

```
from spatialmeta.model import ConditionalVAESTSM; model = ConditionalVAESTSM.fit(adata); emb = model.get_latent_embedding(adata)
```

## Evaluation signals

- Angular similarity values are computed for all five ccRCC IM2 samples and are finite (no NaN) for every spot with both ST and SM modalities
- Modality contribution values fall in a consistent range across samples, with no sample dominated entirely by a single modality unless biologically justified
- Overall integration score derived with modality contributions keeps SpatialMETA at or near the top of the method ranking (vs Seurat RPCA/CCA, scVI, scANVI, totalVI, scPoli, spaVAE, PCA), consistent with Fig. 4d
- scIB Batch ASW, iLISI and PCR batch computed on the same embeddings using the histology-derived manual label key are reported alongside, so batch correction and biological conservation are balanced rather than traded off
- The output CSV (e.g. fig4d_integration_benchmark.csv) contains metric values for every evaluated method, not only SpatialMETA

## Limitations

- Angular-similarity interpretability depends on SpatialMETA's CVAE design and cannot be applied to embeddings from tools lacking separate modality representations
- Ground-truth labels for biology conservation evaluation were manually annotated from H&E histological features, not an independent gold standard
- Benchmark metric values are only provided in source data files rather than in the main text

## Evidence
