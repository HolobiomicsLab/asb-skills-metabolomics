---
name: batch-effect-correction-benchmarking
description: Use when comparing integration or batch correction performance of SpatialMETA
  (ConditionalVAESM/ConditionalVAESTSM) against baseline methods on ST/SM data, and
  you must score embeddings with scIB batch and biology metrics.
when_to_use_negative:
- Data at single-cell or subcellular resolution — SpatialMETA is designed for low-resolution
  ST and SM data only
- The goal is batch correction within a single sample without benchmarking against
  baseline integrators — the scIB comparison framework is unnecessary
- Integration of more than two modalities — SpatialMETA currently only supports integration
  of two modalities
edam_operation: http://edamontology.org/operation_2935
edam_topics:
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_3941
tools:
- name: SpatialMETA
  role: CVAE-based integrator (ConditionalVAESM/ConditionalVAESTSM) whose batch-corrected
    latent embeddings are benchmarked against baselines
  repo: https://github.com/WanluLiuLab/SpatialMETA
- name: Seurat
  role: Baseline horizontal/vertical integrator run with CCA and RPCA on the identical
    filtered matrices
- name: scVI
  role: Baseline horizontal integration method scored with scIB metrics
- name: scANVI
  role: Baseline horizontal integration method scored with scIB metrics
- name: scPoli
  role: Baseline horizontal integration method; top performer for SM-only integration
    alongside SpatialMETA
- name: spaVAE
  role: Baseline spatial horizontal integration method scored with scIB metrics
- name: totalVI
  role: Baseline vertical integration method run with default recommended settings
- name: Stabmap
  role: Baseline vertical integration method run with default recommended settings
- name: scIB
  role: Metric framework computing Batch ASW, iLISI and PCR batch plus continuity
    and marker scores on each embedding
provenance:
  source_task_ids:
  - task_003
  - task_002
  source_papers:
  - doi: 10.1038/s41467-025-63915-z
    title: Integrating cross-sample and cross-modal data for spatial transcriptomics
      and metabolomics with SpatialMETA
  source_content_hash: sha256:77f5ade351a3a5e0f43c18eaad88d00e1d1d3adcaafc30f109dcb3d20dadaa7a
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
  edam_operation: http://edamontology.org/operation_2935
  edam_topics:
  - http://edamontology.org/topic_3697
  - http://edamontology.org/topic_3941
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7692307692307693
n_claims: 13
---

# batch-effect-correction-benchmarking

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmark batch effect correction achieved by SpatialMETA's conditional VAE framework against published horizontal and vertical integration methods (Seurat CCA/RPCA, scVI, scANVI, totalVI, Stabmap, scPoli, spaVAE, PCA) using scIB-style continuity, marker (biology conservation), and batch-correction metrics on ccRCC spatial transcriptomics and spatial metabolomics data.

## When to use

Use when comparing integration or batch correction performance of SpatialMETA (ConditionalVAESM/ConditionalVAESTSM) against baseline methods on ST/SM data, and you must score embeddings with scIB batch and biology metrics.

## When NOT to use

- Data at single-cell or subcellular resolution — SpatialMETA is designed for low-resolution ST and SM data only
- The goal is batch correction within a single sample without benchmarking against baseline integrators — the scIB comparison framework is unnecessary
- Integration of more than two modalities — SpatialMETA currently only supports integration of two modalities

## Inputs

- Preprocessed AnnData objects for ccRCC spatial metabolomics (metabolite intensity matrices, positive/negative ion modes merged via merge_sm_pos_neg)
- Joint ST-SM AnnData objects from the alignment and KNN reassignment pipeline (for vertical+horizontal benchmark)
- Manually annotated histology-derived (H&E) ground-truth labels for the scIB label key
- SpatialMETA latent embeddings from ConditionalVAESM/ConditionalVAESTSM get_latent_embedding

## Outputs

- Per-method metric table covering continuity, marker (biology conservation) and batch-correction categories (e.g. sm_horizontal_integration_benchmark_metrics.csv or fig4d_integration_benchmark.csv)
- Method ranking by overall integration performance
- For joint integration: cross-modality, cross-sample and overall integration scores plus modality contribution via angular similarity

## How to apply

First preprocess the ST/SM inputs and remove batch-biased SVGs/SVMs, defined as features with fold change greater than min_logfc (default 3) expressed in at least min_frac (default 0.8) of spots shared across at least min_samples (default 2 samples), so batch-biased features do not inflate correction scores. Train SpatialMETA's model (ConditionalVAESM for SM-only horizontal integration; ConditionalVAESTSM for joint vertical+horizontal fusion) to learn a shared latent space, and extract embeddings via get_latent_embedding. Run each baseline (Seurat CCA/RPCA, scVI, scANVI, scPoli, spaVAE, PCA; add totalVI and Stabmap for vertical integration) with its default recommended settings on the identical filtered matrices. Evaluate every embedding with scIB using manually annotated histology-derived (H&E) labels as the label key, computing Batch ASW, iLISI and PCR batch, plus continuity and marker (biology conservation) scores; for joint integration also compute cross-modality, cross-sample and overall scores and modality contribution via angular similarity between embeddings. Rank all methods by overall integration performance and assemble the metric values into a comparison table mirroring the paper's Supplementary Fig. 11 / Fig. 4d.

## Related tools

- **SpatialMETA** (CVAE-based integrator (ConditionalVAESM/ConditionalVAESTSM) whose batch-corrected latent embeddings are benchmarked against baselines) — https://github.com/WanluLiuLab/SpatialMETA
- **Seurat** (Baseline horizontal/vertical integrator run with CCA and RPCA on the identical filtered matrices)
- **scVI** (Baseline horizontal integration method scored with scIB metrics)
- **scANVI** (Baseline horizontal integration method scored with scIB metrics)
- **scPoli** (Baseline horizontal integration method; top performer for SM-only integration alongside SpatialMETA)
- **spaVAE** (Baseline spatial horizontal integration method scored with scIB metrics)
- **totalVI** (Baseline vertical integration method run with default recommended settings)
- **Stabmap** (Baseline vertical integration method run with default recommended settings)
- **scIB** (Metric framework computing Batch ASW, iLISI and PCR batch plus continuity and marker scores on each embedding)

## Examples

```
vae = spatialmeta.model.ConditionalVAESM.fit(adatas_sm); emb = vae.get_latent_embedding(); scores = scib.metrics.metrics(adata, emb, label_key='histology_label', batch_key='sample')
```

## Evaluation signals

- The comparison table contains metric rows for all benchmarked methods (SpatialMETA plus Seurat CCA/RPCA, scVI, scANVI, scPoli, spaVAE, PCA — and totalVI/Stabmap for vertical) covering continuity, marker and batch-correction categories
- Batch-biased SVMs/SVGs were removed before scoring: fold change > min_logfc (default 3), expression in >= min_frac (default 0.8) of spots across >= 2 samples
- scIB batch metrics (Batch ASW, iLISI, PCR batch) are computed against a manual H&E histology-derived label key, so the label_key parameter is populated with the annotated labels
- SpatialMETA ranks at or near the top of overall integration performance, consistent with the reported superior performance versus published non-spatial single-cell or spatial multi-omics tools
- For SM-only horizontal integration of ccRCC, the top performers include scPoli, SpatialMETA, and Seurat CCA/RPCA, matching the reported finding

## Limitations

- Ground-truth labels for biology conservation evaluation were manually annotated from H&E histological features, not an independent gold standard
- Detailed benchmark metric values are only provided in source data files rather than in the main text
- Horizontal integration was validated only on a limited number of samples from a single study, so rankings may not generalize

## Evidence

- [discussion] Batch-biased features removed before benchmarking: [evidence span withheld — anchor status: not_found]
- [results] Batch ASW, iLISI and PCR batch computed with scIB: [evidence span withheld — anchor status: not_found]
- [discussion] Superior overall performance versus published tools: "Benchmark analyses demonstrate that, compared to previously published non-spatial single-cell or spatial multi-omics tools, SpatialMETA achieved superior overall performance"
- [abstract] CVAE framework with tailored decoders and losses: "SpatialMETA employs tailored decoders and loss functions to enhance modality fusion, batch effect correction and biological conservation"
- [intro] SM-only benchmark baseline list: "methods such as Seurat CCA/RPCA34, scVI37, scANVI38, SCALEX39, scPoli40, and scAtlasVAE41 are designed for horizontal integration"
- [discussion] Low-resolution design limitation: "Firstly, SpatialMETA is designed for low-resolution ST and SM data"
