---
name: conditional-vae-model-training
description: Use when multiple spatial metabolomics (SM) or paired ST-SM AnnData samples
  have been QC-filtered and batch-corrected preprocessing is done, and you need cross-sample/cross-modal
  latent integration embeddings.
when_to_use_negative:
- Data are at single-cell or subcellular resolution — SpatialMETA is designed for
  low-resolution ST and SM data only
- Integration involves more than two modalities — SpatialMETA currently only supports
  integration of two modalities
- Samples come from unaligned serial sections spanning a 3D tissue volume — alignment/reassignment
  is only implemented between two adjacent sections within a single sample
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_3172
tools:
- name: SpatialMETA
  role: Provides ConditionalVAESM (CVAE-based framework) whose .fit/.get_latent_embedding/.get_normalized_expression
    perform the conditional VAE training and embedding extraction
  repo: https://github.com/WanluLiuLab/SpatialMETA
- name: scVI
  role: Baseline horizontal-integration method trained on the same filtered matrices
    for comparison against ConditionalVAESM embeddings
- name: scPoli
  role: Baseline horizontal-integration method; one of the top performers alongside
    SpatialMETA in the SM-only ccRCC benchmark
- name: Seurat
  role: Baseline method (CCA/RPCA) used to benchmark the trained ConditionalVAESM
    latent space
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-025-63915-z
    title: Integrating cross-sample and cross-modal data for spatial transcriptomics
      and metabolomics with SpatialMETA
  source_content_hash: sha256:efc1d7ca52546e8bfa10d86d8738dd4980937bbe7bd785e2fcaa080d6d497ca1
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
  - scPoli
  - scVI
  - spaVAE
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7894736842105263
n_claims: 19
---

# conditional-vae-model-training

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Train SpatialMETA's conditional variational autoencoder (ConditionalVAESM for metabolomics-only or joint ST-SM data) to learn a shared latent space across samples/modalities, then extract integrated embeddings for benchmarking or cluster identification.

## When to use

Use when multiple spatial metabolomics (SM) or paired ST-SM AnnData samples have been QC-filtered and batch-corrected preprocessing is done, and you need cross-sample/cross-modal latent integration embeddings.

## When NOT to use

- Data are at single-cell or subcellular resolution — SpatialMETA is designed for low-resolution ST and SM data only
- Integration involves more than two modalities — SpatialMETA currently only supports integration of two modalities
- Samples come from unaligned serial sections spanning a 3D tissue volume — alignment/reassignment is only implemented between two adjacent sections within a single sample

## Inputs

- AnnDataSM objects of metabolite intensity matrices from multiple ccRCC (or similar) samples, loaded via spatialmeta.data.load_adata
- QC-filtered SM matrices with positive/negative ion-mode acquisitions merged (merge_sm_pos_neg)
- Optional paired ST AnnData with spatial coordinates for joint ST-SM integration

## Outputs

- Trained ConditionalVAESM model over a shared low-resolution latent space
- Integrated latent embeddings per spot/sample via ConditionalVAESM.get_latent_embedding
- Normalized (reconstructed) expression/metabolite intensities via ConditionalVAESM.get_normalized_expression
- Per-method benchmark metric table (continuity, marker, batch-correction) when used in scIB-style evaluation

## How to apply

After preprocessing each sample with spatialmeta.pp.calculate_qc_metrics_sm, filter_cells_sm, and filter_metabolites_sm (merging positive/negative ion modes with merge_sm_pos_neg where applicable), instantiate spatialmeta.model.ConditionalVAESM and call .fit to learn a shared latent space that simultaneously encodes batch information and the joint embedding. The CVAE uses tailored decoders and loss functions to balance modality fusion, batch effect correction, and biological conservation — SM data are continuous mass spectrometry intensities, not sequencing counts, which the SM decoder handles. Before training, remove batch-biased SVMs/SVGs defined by fold change > min_logfc (default 3) expressed in >= min_frac (default 0.8) of spots across at least min_samples (default 2). After fitting, retrieve the integrated representation with ConditionalVAESM.get_latent_embedding and denoised/normalized features with ConditionalVAESM.get_normalized_expression; embeddings can then be scored with scIB-style continuity, marker, and batch-correction metrics or compared against baselines like Seurat CCA/RPCA, scVI, scANVI, and scPoli.

## Related tools

- **SpatialMETA** (Provides ConditionalVAESM (CVAE-based framework) whose .fit/.get_latent_embedding/.get_normalized_expression perform the conditional VAE training and embedding extraction) — https://github.com/WanluLiuLab/SpatialMETA
- **scVI** (Baseline horizontal-integration method trained on the same filtered matrices for comparison against ConditionalVAESM embeddings)
- **scPoli** (Baseline horizontal-integration method; one of the top performers alongside SpatialMETA in the SM-only ccRCC benchmark)
- **Seurat** (Baseline method (CCA/RPCA) used to benchmark the trained ConditionalVAESM latent space)

## Examples

```
import spatialmeta; model = spatialmeta.model.ConditionalVAESM(adata_sm, batch_key='sample'); model.fit(); emb = model.get_latent_embedding(); expr = model.get_normalized_expression()
```

## Evaluation signals

- get_latent_embedding returns embeddings for all input samples in a shared latent space with batch labels encoded via the conditional prior
- In SM-only horizontal integration of ccRCC samples, SpatialMETA should rank at or near the top of scIB-style continuity, marker, and batch-correction metrics versus six baselines (Seurat CCA/RPCA, scVI, scANVI, scPoli, spaVAE, PCA)
- Benchmark metric table (sm_horizontal_integration_benchmark_metrics.csv) contains rows for all seven methods across continuity, marker, and batch-correction categories
- Reconstructed intensities from get_normalized_expression should capture SM feature distributions consistently with original spot-level intensity patterns (superior reconstruction vs. existing tools)
- Batch-biased SVMs (fold change > 3, expression in >= 0.8 of spots) must be removed before training, confirmed by the filtering log

## Limitations

- SpatialMETA is designed for low-resolution ST and SM data and does not support single-cell or subcellular resolution
- Horizontal integration was validated only on a limited number of samples from a single study
- Ground-truth labels for biology conservation evaluation were manually annotated from H&E histological features, not an independent gold standard

## Evidence

- [discussion] Batch-biased SVM removal thresholds: [evidence span withheld — anchor status: not_found]
- [other] ConditionalVAESM fit in workflow: [evidence span withheld — anchor status: not_found]
