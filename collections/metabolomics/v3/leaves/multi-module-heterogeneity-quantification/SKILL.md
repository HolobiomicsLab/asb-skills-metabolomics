---
name: multi-module-heterogeneity-quantification
description: Use when you have spot-level multi-module spatial data (transcript, metabolite,
  morphology) integrated by haCCA and need to demonstrate that the integrated view
  captures more heterogeneity than any single modality.
when_to_use_negative:
- When only a single spatial modality (e.g. transcript-only) is available and no integrated
  multi-module embedding exists to compare against.
- When the goal is spot-to-spot integration/alignment of spatial transcriptome and
  MALDI-MSI data itself (that is haCCA's direct_alignment / icp_3d_alignment step),
  not downstream heterogeneity quantification.
- When higher-resolution data is used — the article notes haCCA behavior on higher
  resolution data has not been evaluated, so cluster-count comparisons may not be
  calibrated.
edam_operation: http://edamontology.org/operation_3625
edam_topics:
- http://edamontology.org/topic_3382
- http://edamontology.org/topic_3173
tools:
- name: haCCA
  role: Provides the integrated multi-module feature matrix that serves as the input
    embedding space for heterogeneity quantification
  repo: github.com/LittleLittleCloud/haCCA
- name: Scanpy
  role: Builds the weighted nearest-neighbor graph, computes the embedding (e.g. UMAP),
    and performs Leiden clustering on each view
- name: 10X Visium
  role: Source of the spatial transcriptome data used as one modality in the multi-module
    embedding
- name: MALDI-MSI
  role: Source of the spatial metabolome data used as one modality in the multi-module
    embedding
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1101/2024.08.20.608773
    title: 'haCCA: Multi-module Integrating of spatial transcriptomes and metabolomes'
  source_content_hash: sha256:254bc4065d359b1579d3b6d12f1ea7ff45d5855ca2839e2c2693aca285fe1d44
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/LittleLittleCloud/haCCA
  tools:
  - 10X Visium
  - MALDI-MSI
  - Python
  - Scanpy
  - haCCA
  edam_operation: http://edamontology.org/operation_3625
  edam_topics:
  - http://edamontology.org/topic_3382
  - http://edamontology.org/topic_3173
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# multi-module-heterogeneity-quantification

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Quantifies spatial heterogeneity by comparing cluster counts and heterogeneity metrics across transcript-only, metabolite-only, and integrated multi-module (morphology + transcript + metabolite) WNN embeddings built with Scanpy on haCCA-integrated 10X Visium and MALDI-MSI mouse brain data.

## When to use

Use when you have spot-level multi-module spatial data (transcript, metabolite, morphology) integrated by haCCA and need to demonstrate that the integrated view captures more heterogeneity than any single modality.

## When NOT to use

- When only a single spatial modality (e.g. transcript-only) is available and no integrated multi-module embedding exists to compare against.
- When the goal is spot-to-spot integration/alignment of spatial transcriptome and MALDI-MSI data itself (that is haCCA's direct_alignment / icp_3d_alignment step), not downstream heterogeneity quantification.
- When higher-resolution data is used — the article notes haCCA behavior on higher resolution data has not been evaluated, so cluster-count comparisons may not be calibrated.

## Inputs

- haCCA-integrated multi-module feature matrix (morphological + transcript + metabolite) in Scanpy AnnData format (.h5ad)
- Transcript-only feature matrix from 10X Visium data (AnnData)
- Metabolite-only feature matrix from MALDI-MSI data (AnnData)
- Spatial coordinate matrix (obsm['spatial']) for each view

## Outputs

- Number of clusters per view (integrated, transcript-only, metabolite-only)
- Heterogeneity metric per view
- cluster_comparison_figure_4.png — comparison figure of cluster counts and heterogeneity across views

## How to apply

Load the haCCA-integrated multi-module feature matrix, the transcript-only matrix, and the metabolite-only matrix as Scanpy AnnData objects from the pipeline outputs (.h5ad). For each view build a weighted nearest-neighbor graph using Scanpy's neighbors function — applying the combined morphological + transcript + metabolite weighting for the integrated view and single-modality weights for the control views. Compute a low-dimensional embedding (e.g. UMAP on the weighted neighbor graph) and run Leiden clustering in each embedded space, recording the number of clusters and a heterogeneity metric per view. Produce a comparison figure (cluster_comparison_figure_4.png) showing that the integrated multi-module view yields the highest heterogeneity and cluster count, with metabolite-only data the lowest, as reported for the mouse brain cohort.

## Related tools

- **haCCA** (Provides the integrated multi-module feature matrix that serves as the input embedding space for heterogeneity quantification) — github.com/LittleLittleCloud/haCCA
- **Scanpy** (Builds the weighted nearest-neighbor graph, computes the embedding (e.g. UMAP), and performs Leiden clustering on each view)
- **10X Visium** (Source of the spatial transcriptome data used as one modality in the multi-module embedding)
- **MALDI-MSI** (Source of the spatial metabolome data used as one modality in the multi-module embedding)

## Examples

```
import scanpy as sc; adata = sc.read_h5ad('integrated_multimodule.h5ad'); sc.pp.neighbors(adata, use_rep='X'); sc.tl.umap(adata); sc.tl.leiden(adata, resolution=1.0)
```

## Evaluation signals

- The integrated multi-module view yields the highest cluster count among the three views.
- The integrated view yields the highest heterogeneity metric, while the metabolite-only view yields the lowest.
- All three views are clustered on the same spot set (same number of spots from the mouse brain cohort) so cluster counts are directly comparable.
- The comparison figure (cluster_comparison_figure_4.png) renders number-of-clusters and heterogeneity side-by-side for transcript-only, metabolite-only, and integrated views.

## Limitations

- haCCA's behavior on higher-resolution data has not been evaluated, so the heterogeneity ranking may not hold for non-Visium-resolution platforms.
- The skill is demonstrated on mouse brain 10X Visium + MALDI-MSI and ICC model data; other tissue types or modalities (image-based sequencing, spatial proteomics) are outside the tested scope.
- No DOI, publication venue, or versioned parameters are specified for the clustering/embedding step in the article, so exact Leiden resolution values must be chosen by the practitioner.

## Evidence

- [results] WNN embedding of the integrated multi-module data enabled more dedicated exploration of mouse brain heterogeneity by finding more clusters; generally, integrated data has the highest heterogeneity while metabolite data has the lowest.: [evidence span withheld — anchor status: not_found]
