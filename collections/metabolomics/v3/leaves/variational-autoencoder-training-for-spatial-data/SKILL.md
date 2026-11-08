---
name: variational-autoencoder-training-for-spatial-data
description: Use when paired ST-SM samples (e.g., human ccRCC, GBM, or mouse brain)
  need cross-modal, cross-sample integration into a joint low-dimensional embedding
  for downstream clustering, deconvolution-based annotation, or benchmarking against
  integration tools.
when_to_use_negative:
- Data are single-cell or subcellular resolution ST/SM — SpatialMETA is designed for
  low-resolution ST and SM data only
- More than two modalities must be integrated — SpatialMETA currently supports integration
  of only two modalities
- SM and ST are measured on the same tissue section at cellular resolution requiring
  precise cell segmentation — computational alignment at cellular/subcellular levels
  is not yet implemented
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_3934
- http://edamontology.org/topic_3170
tools:
- name: SpatialMETA
  role: CVAE-based framework (ConditionalVAESTSM) trained to jointly embed ST and
    SM data for vertical and horizontal integration
  repo: https://github.com/WanluLiuLab/SpatialMETA
- name: totalVI
  role: Baseline vertical integration method run on the same data for benchmark comparison
- name: Seurat
  role: Baseline vertical integration method (v5 with BNN) run on the same data for
    benchmark comparison
- name: Stabmap
  role: Baseline vertical integration method run on the same data for benchmark comparison
- name: spaMultiVAE
  role: Baseline spatial multi-omics integration method used in the benchmark
- name: SpatialGLUE
  role: Baseline spatial multi-omics integration method used in the benchmark
- name: MISO
  role: Baseline (2-mode and 3-mode) spatial multi-omics integration method used in
    the benchmark
- name: scIB
  role: Metric suite computing reconstruction accuracy, continuity, marker, and biological
    conservation scores on each method's joint embedding
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-025-63915-z
    title: Integrating cross-sample and cross-modal data for spatial transcriptomics
      and metabolomics with SpatialMETA
  source_content_hash: sha256:ec69fc5756831c5baad16e89a8573e436cb3080a94282029839c95ac7db1bbb6
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
  - MISO
  - Seurat
  - SpatialGLUE
  - SpatialMETA
  - Stabmap
  - scIB
  - spaMultiVAE
  - totalVI
  edam_operation: http://edamontology.org/operation_3927
  edam_topics:
  - http://edamontology.org/topic_3934
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# variational-autoencoder-training-for-spatial-data

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Train SpatialMETA's ConditionalVAESTSM, a conditional variational autoencoder (CVAE), to jointly embed spatial transcriptomics (ST) and spatial metabolomics (SM) data for vertical and horizontal integration, producing latent embeddings and normalized expression used for benchmarking and spatial cluster analysis.

## When to use

Use when paired ST-SM samples (e.g., human ccRCC, GBM, or mouse brain) need cross-modal, cross-sample integration into a joint low-dimensional embedding for downstream clustering, deconvolution-based annotation, or benchmarking against integration tools.

## When NOT to use

- Data are single-cell or subcellular resolution ST/SM — SpatialMETA is designed for low-resolution ST and SM data only
- More than two modalities must be integrated — SpatialMETA currently supports integration of only two modalities
- SM and ST are measured on the same tissue section at cellular resolution requiring precise cell segmentation — computational alignment at cellular/subcellular levels is not yet implemented

## Inputs

- Eleven vertically integrated ST-SM AnnData objects from human ccRCC, GBM, and mouse brain datasets loaded via spatialmeta.data.load_adata
- SM data aligned to ST via rotation, translation, non-linear distortion, and KNN reassignment
- Batch/sample labels to condition the CVAE
- Manually H&E-annotated label key (for downstream scIB evaluation)

## Outputs

- Latent joint embedding from get_latent_embedding
- Normalized (denoised) expression from get_normalized_expression
- Ranked overall-score benchmark table (e.g., benchmark_overall_scores.csv) when used for evaluation

## How to apply

Load the vertically integrated ST-SM samples (eleven samples spanning ccRCC, GBM, and mouse brain) via spatialmeta.data.load_adata, after preprocessing steps such as SM-to-ST alignment with rotation, translation, and non-linear distortion, KNN reassignment, and removal of batch-biased SVGs/SVMs (fold change > min_logfc=3, expressed in >= min_frac=0.8 of spots, shared across >= min_samples=2). Train the ConditionalVAESTSM with spatialmeta.model.ConditionalVAESTSM.fit, which uses tailored decoders and loss functions per modality to handle continuous mass-spectrometry intensity (SM) versus sequencing counts (ST) and to enhance modality fusion, batch effect correction, and biological conservation. Extract the joint embedding with get_latent_embedding and denoised expression with get_normalized_expression, then feed the embedding into scIB-style benchmark metrics (reconstruction accuracy, continuity, marker, biological conservation) or spatial clustering and annotation. The conditional (batch-aware) VAE design simultaneously integrates batch information and the joint embedding, so a single trained model supports both vertical (cross-modal) and horizontal (cross-sample) integration.

## Related tools

- **SpatialMETA** (CVAE-based framework (ConditionalVAESTSM) trained to jointly embed ST and SM data for vertical and horizontal integration) — https://github.com/WanluLiuLab/SpatialMETA
- **totalVI** (Baseline vertical integration method run on the same data for benchmark comparison)
- **Seurat** (Baseline vertical integration method (v5 with BNN) run on the same data for benchmark comparison)
- **Stabmap** (Baseline vertical integration method run on the same data for benchmark comparison)
- **spaMultiVAE** (Baseline spatial multi-omics integration method used in the benchmark)
- **SpatialGLUE** (Baseline spatial multi-omics integration method used in the benchmark)
- **MISO** (Baseline (2-mode and 3-mode) spatial multi-omics integration method used in the benchmark)
- **scIB** (Metric suite computing reconstruction accuracy, continuity, marker, and biological conservation scores on each method's joint embedding)

## Examples

```
from spatialmeta.model import ConditionalVAESTSM; model = ConditionalVAESTSM(adata); model.fit(); latent = model.get_latent_embedding(); expr = model.get_normalized_expression()
```

## Evaluation signals

- The output benchmark table contains one row per evaluated method (SpatialMETA plus 7 baselines: totalVI, spaMultiVAE, MISO 2-mode/3-mode, Seurat v5 BNN, Stabmap, SpatialGLUE) with an overall-score column
- SpatialMETA ranks first on the aggregated overall score across the four metric families, reproducing the ordering in Fig. 2b and Supplementary Fig. 2e-h
- The latent embedding preserves manually H&E-annotated histological labels (scIB label key) better than baselines for biological conservation
- Reconstructed expression from get_normalized_expression accurately captures both ST and SM feature distributions relative to input data
- Modality contributions computed via angular similarity between embeddings are interpretable and non-degenerate

## Limitations

- SpatialMETA is designed for low-resolution ST and SM data and does not support single-cell or subcellular resolution
- SpatialMETA currently only supports integration of two modalities; more than two modalities is a future need
- Ground-truth labels for biology conservation evaluation were manually annotated from H&E histological features, not an independent gold standard
- Alignment and reassignment are currently applied only between two adjacent sections within a single sample; multi-section 3D alignment is not implemented
- Detailed benchmark metric values are only provided in source data files rather than in the main text

## Evidence
