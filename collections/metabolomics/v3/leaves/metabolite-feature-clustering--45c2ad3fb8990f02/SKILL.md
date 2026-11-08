---
name: metabolite-feature-clustering
description: Use when you have spot-resolved metabolite peak/intensity matrices (e.g.
  from MALDI-MSI of 10X Visium-matched mouse brain or ICC sections) and need to cluster
  metabolite features or compare cluster counts/heterogeneity against transcript-only
  and integrated views.
when_to_use_negative:
- The metabolite data are not spot-resolved or not aligned to the transcriptomic spots
  — haCCA's spot-to-spot integration relies on neighbor tissue sections and the Data
  object requires a spatial matrix D
- The question concerns molecular alignment itself (manual_gross_alignment, icp_3d_alignment,
  direct_alignment) rather than downstream clustering of already-aligned metabolite
  features
- The goal is higher-resolution (e.g. single-cell) analysis — the article notes haCCA's
  behavior on higher-resolution data has not been evaluated
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: haCCA
  role: Produces the spot-aligned metabolite feature matrix (b_predict) whose peak/intensity
    features are subsequently clustered per spot
  repo: github.com/LittleLittleCloud/haCCA
- name: Scanpy
  role: Standard processing, extraction of the peak/intensity feature matrix, weighted
    neighbor graph construction, UMAP embedding, and Leiden clustering of the metabolite
    view
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1101/2024.08.20.608773
    title: 'haCCA: Multi-module Integrating of spatial transcriptomes and metabolomes'
  source_content_hash: sha256:e13c6579c6bb08fd8c10335944406cca25bba9c01857ab8bb473a54292a94c0b
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
  - Python
  - Scanpy
  - haCCA
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# metabolite-feature-clustering

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Clusters MALDI-MSI metabolite peak/intensity features at spot resolution (alone and within haCCA-integrated multi-module data) using Scanpy neighbor graphs, embeddings, and Leiden clustering, to quantify metabolic heterogeneity across tissue sections.

## When to use

Use when you have spot-resolved metabolite peak/intensity matrices (e.g. from MALDI-MSI of 10X Visium-matched mouse brain or ICC sections) and need to cluster metabolite features or compare cluster counts/heterogeneity against transcript-only and integrated views.

## When NOT to use

- The metabolite data are not spot-resolved or not aligned to the transcriptomic spots — haCCA's spot-to-spot integration relies on neighbor tissue sections and the Data object requires a spatial matrix D
- The question concerns molecular alignment itself (manual_gross_alignment, icp_3d_alignment, direct_alignment) rather than downstream clustering of already-aligned metabolite features
- The goal is higher-resolution (e.g. single-cell) analysis — the article notes haCCA's behavior on higher-resolution data has not been evaluated

## Inputs

- Metabolite-only feature matrix from MALDI-MSI as AnnData (.h5ad), with X = peak/intensity matrix and obsm['spatial'] = spot coordinates
- haCCA-integrated multi-module feature matrix (morphological + transcript + metabolite) as AnnData
- Transcript-only 10X Visium feature matrix as AnnData (control view)

## Outputs

- Leiden cluster labels per spot in the metabolite (and integrated) embedding space
- Per-view cluster count and heterogeneity metric
- Comparison figure (cluster_comparison_figure_4.png) showing clusters/heterogeneity for transcript-only, metabolite-only, and integrated views

## How to apply

Load the metabolite-only feature matrix — and optionally the haCCA-integrated morphological+transcript+metabolite matrix — as Scanpy AnnData objects from the haCCA pipeline outputs (a.h5ad / b_prime.h5ad, using X as the feature matrix and obsm['spatial'] for coordinates). After standard Scanpy processing extracts the peak/intensity feature matrix, build a weighted nearest-neighbor graph with sc.pp.neighbors (single-modality weights for the metabolite view; morphological+transcript+metabolite weighting for the integrated view), compute a low-dimensional embedding (e.g. UMAP on the weighted neighbor graph), then run Leiden clustering to assign clusters per spot. Record the number of clusters and a heterogeneity metric per view and compare transcript-only, metabolite-only, and integrated views; the expected pattern is that the integrated view yields the most clusters and highest heterogeneity while metabolite-only data yields the lowest. Rationale: clustering metabolite profiles in embedded space reveals distinct metabolic states, and the comparison tests whether haCCA's high-correlated feature pair integration adds heterogeneity signal beyond either modality alone.

## Related tools

- **haCCA** (Produces the spot-aligned metabolite feature matrix (b_predict) whose peak/intensity features are subsequently clustered per spot) — github.com/LittleLittleCloud/haCCA
- **Scanpy** (Standard processing, extraction of the peak/intensity feature matrix, weighted neighbor graph construction, UMAP embedding, and Leiden clustering of the metabolite view)

## Examples

```
import scanpy as sc; adata = sc.read_h5ad('/path/to/b_prime.h5ad'); sc.pp.neighbors(adata); sc.tl.umap(adata); sc.tl.leiden(adata)
```

## Evaluation signals

- Leiden clustering runs on the weighted neighbor graph for each view (metabolite-only, transcript-only, integrated) and a cluster count is recorded per view
- The metabolite-only view yields fewer clusters and lower heterogeneity than the integrated multi-module view
- The integrated view shows the highest heterogeneity and cluster count of the three views
- Comparison figure is saved as cluster_comparison_figure_4.png in the output directory

## Limitations

- Spatial transcriptomics and MALDI-MSI data share no spots or features natively, making integration challenging before clustering can be trusted
- Only spatial transcriptome and spatial metabolome data have been used; other modalities (image-based sequencing, spatial proteomics, histological images) are not yet supported
- haCCA's behavior on higher-resolution data has not been evaluated, so metabolite-feature clustering inherits that resolution limit

## Evidence
