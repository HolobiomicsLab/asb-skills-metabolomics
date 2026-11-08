---
name: spatial-transcriptome-clustering-comparison
description: Use when you have haCCA-integrated multi-module spatial data (morphology
  + transcripts + metabolites) plus matched transcript-only or metabolite-only matrices
  and want to test whether integration yields more clusters and higher heterogeneity.
when_to_use_negative:
- Input lacks an integrated multi-module embedding or matched single-modality control
  views — comparing clusters requires all three views from the haCCA pipeline.
- Data are not spatially resolved spot-level matrices (e.g. bulk transcriptomics or
  non-imaging mass spec), since the method relies on spatial neighborhood graphs.
- The goal is aligning adjacent sections rather than quantifying cluster heterogeneity;
  use haCCA's alignment workflow instead.
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_0102
- http://edamontology.org/topic_3316
tools:
- name: Scanpy
  role: Builds weighted nearest-neighbor graphs, computes UMAP embeddings, and runs
    Leiden clustering on each spatial omics view.
- name: haCCA
  role: Produces the integrated multi-module feature matrix whose clustering is compared
    against transcript-only and metabolite-only controls.
  repo: github.com/LittleLittleCloud/haCCA
- name: 10X Visium
  role: Provides the spatial transcriptome spot data used as one of the clustering
    views.
- name: MALDI-MSI
  role: Provides the spatial metabolite (m/z) data used as the metabolite-only clustering
    view.
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1101/2024.08.20.608773
    title: 'haCCA: Multi-module Integrating of spatial transcriptomes and metabolomes'
  source_content_hash: sha256:0d0acdf585ae1aac9b64975bb86c27b46bfa80db92527bc227b49984e8a9b2c0
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
  - http://edamontology.org/topic_0102
  - http://edamontology.org/topic_3316
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6875
n_claims: 16
---

# spatial-transcriptome-clustering-comparison

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Compare clustering outcomes across single-modality and integrated spatial omics embeddings to quantify how multi-module integration (e.g. haCCA-aligned transcripts + metabolites) increases detected heterogeneity in 10X Visium / MALDI-MSI data.

## When to use

Use when you have haCCA-integrated multi-module spatial data (morphology + transcripts + metabolites) plus matched transcript-only or metabolite-only matrices and want to test whether integration yields more clusters and higher heterogeneity.

## When NOT to use

- Input lacks an integrated multi-module embedding or matched single-modality control views — comparing clusters requires all three views from the haCCA pipeline.
- Data are not spatially resolved spot-level matrices (e.g. bulk transcriptomics or non-imaging mass spec), since the method relies on spatial neighborhood graphs.
- The goal is aligning adjacent sections rather than quantifying cluster heterogeneity; use haCCA's alignment workflow instead.

## Inputs

- haCCA-integrated multi-module feature matrix (morphological + transcript + metabolite) as .h5ad AnnData
- transcript-only 10X Visium feature matrix (.h5ad)
- metabolite-only MALDI-MSI feature matrix (.h5ad)
- spot spatial coordinates (obsm['spatial']) for each view

## Outputs

- cluster_comparison_figure_4.png showing number of clusters and heterogeneity per view
- per-view Leiden cluster assignments
- per-view heterogeneity metric values (clusters count and heterogeneity score)

## How to apply

Load the haCCA-integrated multi-module feature matrix, the transcript-only matrix, and the metabolite-only matrix as Scanpy AnnData objects (.h5ad). For each view, build a weighted nearest-neighbor graph with Scanpy's neighbors function using the appropriate weighting scheme (morphology + transcript + metabolite weights for the integrated view; single-modality weights for the controls), then compute a low-dimensional embedding via Scanpy's UMAP on the weighted graph. Cluster spots with Leiden clustering and record both the number of clusters and a per-view heterogeneity metric. Produce a comparison figure (e.g. cluster_comparison_figure_4.png) showing that the integrated multi-module view yields the highest heterogeneity and the most clusters, while metabolite-only data yields the lowest, as reported in the haCCA evaluation on the mouse brain 10X Visium / MALDI-MSI cohort.

## Related tools

- **Scanpy** (Builds weighted nearest-neighbor graphs, computes UMAP embeddings, and runs Leiden clustering on each spatial omics view.)
- **haCCA** (Produces the integrated multi-module feature matrix whose clustering is compared against transcript-only and metabolite-only controls.) — github.com/LittleLittleCloud/haCCA
- **10X Visium** (Provides the spatial transcriptome spot data used as one of the clustering views.)
- **MALDI-MSI** (Provides the spatial metabolite (m/z) data used as the metabolite-only clustering view.)

## Examples

```
import scanpy as sc; adata = sc.read_h5ad('integrated.h5ad'); sc.pp.neighbors(adata, use_rep='X'); sc.tl.umap(adata); sc.tl.leiden(adata)
```

## Evaluation signals

- Integrated multi-module view should yield the highest number of Leiden clusters among the three views.
- Integrated multi-module view should show the highest heterogeneity metric; metabolite-only should show the lowest.
- Each view must have a successfully built neighbor graph and computed UMAP embedding before clustering.
- Output figure cluster_comparison_figure_4.png must be present in the output directory and contain the three-view comparison.
- Cluster counts must be derived from Leiden clustering in the embedded space, not raw feature space.

## Limitations

- Heterogeneity conclusions depend on the weighting scheme used to build the neighbor graphs; the integrated view uses morphology + transcript + metabolite weights.
- The comparison is validated on mouse brain 10X Visium / MALDI-MSI data and an ICC model; performance on other tissues or platforms is untested.
- haCCA integration is limited to spatial transcriptome and spatial metabolome data; other modalities are not yet supported.

## Evidence

- [other] WNN embedding of the integrated multi-module data enabled more dedicated exploration of mouse brain heterogeneity by finding more clusters: [evidence span withheld — anchor status: not_found]
