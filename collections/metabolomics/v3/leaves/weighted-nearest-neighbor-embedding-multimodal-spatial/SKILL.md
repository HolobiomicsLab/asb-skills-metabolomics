---
name: weighted-nearest-neighbor-embedding-multimodal-spatial
description: Use when you have spot-level feature matrices from spatial transcriptomics
  (e.g. 10X Visium .h5ad), spatial metabolomics (MALDI-MSI peak/intensity matrices),
  or haCCA-integrated multi-module matrices, and want to compare cluster counts and
  heterogeneity across modality views.
when_to_use_negative:
- Data has not been through haCCA integration — there are no shared spots or features
  between spatial transcriptomics and MALDI-MSI data, so a naive WNN comparison across
  unaligned modalities is not meaningful
- Only one modality is available and the analysis goal is single-view clustering rather
  than cross-modality heterogeneity comparison
- The goal is evaluating integration/alignment accuracy itself (label transfer accuracy,
  label transfer ARI) rather than downstream heterogeneity exploration
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0092
tools:
- name: haCCA
  role: Produces the aligned, integrated multi-module feature matrices (via manual_gross_alignment,
    icp_3d_alignment, direct_alignment) that serve as the input views for WNN embedding
  repo: github.com/LittleLittleCloud/haCCA
- name: Scanpy
  role: Used to load .h5ad AnnData objects, build weighted neighbor graphs (neighbors),
    compute UMAP embeddings on the weighted graph, and run Leiden clustering to count
    clusters per view
- name: 10X Visium
  role: Source platform for the spatial transcriptome spots (mouse brain public cohort)
    contributing the transcript-only view
- name: MALDI-MSI
  role: Source platform for the metabolome m/z peak/intensity matrices contributing
    the metabolite-only view
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1101/2024.08.20.608773
    title: 'haCCA: Multi-module Integrating of spatial transcriptomes and metabolomes'
  source_content_hash: sha256:a55f878fd9abf0e630440e6226a109f5b882b61598f1e09000eac6fdb8495608
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
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6923076923076923
n_claims: 13
---

# weighted-nearest-neighbor-embedding-multimodal-spatial

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Builds weighted-nearest-neighbor (WNN) embeddings on haCCA-integrated multi-module spatial data (morphology, 10X Visium transcripts, MALDI-MSI metabolites) and clusters spots per view to quantify how much additional heterogeneity the integrated view reveals over single-modality views.

## When to use

Use when you have spot-level feature matrices from spatial transcriptomics (e.g. 10X Visium .h5ad), spatial metabolomics (MALDI-MSI peak/intensity matrices), or haCCA-integrated multi-module matrices, and want to compare cluster counts and heterogeneity across modality views.

## When NOT to use

- Data has not been through haCCA integration — there are no shared spots or features between spatial transcriptomics and MALDI-MSI data, so a naive WNN comparison across unaligned modalities is not meaningful
- Only one modality is available and the analysis goal is single-view clustering rather than cross-modality heterogeneity comparison
- The goal is evaluating integration/alignment accuracy itself (label transfer accuracy, label transfer ARI) rather than downstream heterogeneity exploration

## Inputs

- haCCA-integrated multi-module feature matrix (morphological + transcript + metabolite) as a Scanpy .h5ad AnnData object
- transcript-only feature matrix from 10X Visium spatial data (.h5ad)
- metabolite-only peak/intensity feature matrix from MALDI-MSI data (.h5ad)
- spatial coordinates (obsm['spatial']) for spot-to-spot alignment

## Outputs

- per-view cluster counts (Leiden) and heterogeneity metrics for transcript-only, metabolite-only, and integrated views
- low-dimensional WNN embeddings (e.g. UMAP) per view
- comparison figure saved as cluster_comparison_figure_4.png in the output directory

## How to apply

Load the haCCA pipeline outputs — the integrated multi-module feature matrix (morphological + transcript + metabolite), the transcript-only matrix, and the metabolite-only matrix — as Scanpy AnnData objects from .h5ad files. For each view, build a weighted nearest-neighbor graph with Scanpy's neighbors function, applying the morphological + transcript + metabolite weighting scheme to the integrated view and single-modality weights to the control views, then compute a low-dimensional WNN embedding (e.g. UMAP on the weighted neighbor graph). Cluster spots in each embedded space with Leiden clustering in Scanpy and record the number of clusters plus a heterogeneity metric per view. Produce a comparison figure showing clusters and heterogeneity for transcript-only, metabolite-only, and integrated views; the expected outcome is that the integrated view yields the most clusters and highest heterogeneity while metabolite-only data yields the lowest. The rationale is that haCCA's high-correlated feature pairs plus modified spatial morphological alignment give spot-to-spot integration quality that makes cross-modality WNN meaningful.

## Related tools

- **haCCA** (Produces the aligned, integrated multi-module feature matrices (via manual_gross_alignment, icp_3d_alignment, direct_alignment) that serve as the input views for WNN embedding) — github.com/LittleLittleCloud/haCCA
- **Scanpy** (Used to load .h5ad AnnData objects, build weighted neighbor graphs (neighbors), compute UMAP embeddings on the weighted graph, and run Leiden clustering to count clusters per view)
- **10X Visium** (Source platform for the spatial transcriptome spots (mouse brain public cohort) contributing the transcript-only view)
- **MALDI-MSI** (Source platform for the metabolome m/z peak/intensity matrices contributing the metabolite-only view)

## Examples

```
import scanpy as sc; adata = sc.read_h5ad('/path/to/integrated.h5ad'); sc.pp.neighbors(adata); sc.tl.umap(adata); sc.tl.leiden(adata)
```

## Evaluation signals

- Integrated multi-module view yields the highest number of Leiden clusters compared with transcript-only and metabolite-only views
- Metabolite-only view yields the lowest heterogeneity metric of the three views
- All three views are embedded with the correct weighting scheme (multi-module weights for integrated, single-modality weights for controls) before clustering
- Comparison figure is produced and saved as cluster_comparison_figure_4.png in the output directory
- Upstream haCCA alignment quality is plausible — the paper reports haCCA outperforms morphological alignment (STUtility) by 6-20% on label transfer accuracy and ARI, so integrated-view clustering should not be run on poorly aligned inputs

## Limitations

- haCCA integration is currently limited to spatial transcriptome and spatial metabolome data; image-based sequencing, spatial proteomics, and histological-image modalities are not yet supported
- Behavior of haCCA on higher-resolution data has not been evaluated
- There are no shared spots or features between spatial transcriptomics and MALDI-MSI data, making integration accuracy the critical prerequisite for meaningful WNN clustering
- No software version is specified for hacca (pip install hacca) or for benchmark comparators such as STUtility

## Evidence

- [results] WNN finding of integrated heterogeneity: [evidence span withheld — anchor status: not_found]
