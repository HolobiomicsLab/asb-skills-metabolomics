---
name: scanpy-neighbor-graph-embedding
description: Use when spot-level feature matrices (transcript, metabolite, or fused
  multi-module) from a spatial omics pipeline are available and the goal is to quantify
  cluster counts or heterogeneity per view in an embedded space.
when_to_use_negative:
- The data are single-modality only with no integrated multi-module matrix to compare
  against, so the three-view heterogeneity contrast is meaningless
- The task is spot-to-spot integration/alignment of spatial transcriptome and MALDI-MSI
  sections itself (that is haCCA's alignment role), not downstream clustering comparison
  of already-integrated matrices
- Features are unprocessed raw spectra rather than the standard Scanpy-processed feature/intensity
  matrices the haCCA workflow extracts
edam_operation: http://edamontology.org/operation_2935
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: Scanpy
  role: Standard processing of spatial data, neighbor graph construction (neighbors),
    UMAP embedding, and Leiden clustering used to compare heterogeneity across transcript-only,
    metabolite-only, and integrated views
- name: haCCA
  role: Produces the integrated multi-module (morphological + transcript + metabolite)
    feature matrix whose embedded clustering is compared against single-modality views
  repo: github.com/LittleLittleCloud/haCCA
- name: 10X Visium
  role: Source of the mouse brain spatial transcriptome spot-level data used in the
    three-view clustering comparison
- name: MALDI-MSI
  role: Source of the metabolite (m/z intensity) matrix providing the metabolite-only
    view in the heterogeneity comparison
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1101/2024.08.20.608773
    title: 'haCCA: Multi-module Integrating of spatial transcriptomes and metabolomes'
  source_content_hash: sha256:b0d96074141e77b2b4aef6b7e570ac2c2f1ebdb3e4b327167e6127bb9b63d087
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
  edam_operation: http://edamontology.org/operation_2935
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# scanpy-neighbor-graph-embedding

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Build weighted-nearest-neighbor graphs on spatial feature matrices with Scanpy, embed them (e.g. UMAP on the weighted neighbor graph), and cluster spots with Leiden to compare heterogeneity across single-modality and integrated multi-module views.

## When to use

Use when spot-level feature matrices (transcript, metabolite, or fused multi-module) from a spatial omics pipeline are available and the goal is to quantify cluster counts or heterogeneity per view in an embedded space.

## When NOT to use

- The data are single-modality only with no integrated multi-module matrix to compare against, so the three-view heterogeneity contrast is meaningless
- The task is spot-to-spot integration/alignment of spatial transcriptome and MALDI-MSI sections itself (that is haCCA's alignment role), not downstream clustering comparison of already-integrated matrices
- Features are unprocessed raw spectra rather than the standard Scanpy-processed feature/intensity matrices the haCCA workflow extracts

## Inputs

- AnnData objects (.h5ad) holding the integrated multi-module feature matrix (morphological + transcript + metabolite) from haCCA pipeline outputs
- AnnData objects (.h5ad) holding transcript-only and metabolite-only feature matrices with spot spatial coordinates (obsm['spatial'])
- Modality weighting scheme (morphological + transcript + metabolite weights for the integrated view; single-modality weights for controls)

## Outputs

- Per-view Leiden cluster labels for spatial spots
- Number of clusters and a heterogeneity metric per view
- Comparison figure (saved as cluster_comparison_figure_4.png) showing clusters and heterogeneity for transcript-only, metabolite-only, and integrated views

## How to apply

Load each feature view (transcript-only, metabolite-only, and the haCCA-integrated morphological + transcript + metabolite matrix) as a Scanpy AnnData object, e.g. from a.h5ad / b_prime.h5ad outputs of the haCCA pipeline. Build a weighted nearest-neighbor graph per view with Scanpy's neighbors function, applying the morphological + transcript + metabolite weighting scheme for the integrated view and single-modality weights for the control views. Compute a low-dimensional embedding (e.g. UMAP on the weighted neighbor graph) and run Leiden clustering in each embedded space, recording the number of clusters and a heterogeneity metric per view. Produce a comparison figure across the three views; the expected result is that the integrated multi-module view yields the most clusters and highest heterogeneity while the metabolite-only view yields the lowest.

## Related tools

- **Scanpy** (Standard processing of spatial data, neighbor graph construction (neighbors), UMAP embedding, and Leiden clustering used to compare heterogeneity across transcript-only, metabolite-only, and integrated views)
- **haCCA** (Produces the integrated multi-module (morphological + transcript + metabolite) feature matrix whose embedded clustering is compared against single-modality views) — github.com/LittleLittleCloud/haCCA
- **10X Visium** (Source of the mouse brain spatial transcriptome spot-level data used in the three-view clustering comparison)
- **MALDI-MSI** (Source of the metabolite (m/z intensity) matrix providing the metabolite-only view in the heterogeneity comparison)

## Examples

```
import scanpy as sc; adata = sc.read_h5ad('/path/to/integrated.h5ad'); sc.pp.neighbors(adata); sc.tl.umap(adata); sc.tl.leiden(adata, key_added='leiden_multi')
```

## Evaluation signals

- The integrated multi-module view yields strictly more Leiden clusters than either transcript-only or metabolite-only views
- Heterogeneity metric ordering matches the reported result: integrated > transcript-only > metabolite-only
- The comparison figure is saved under the expected filename (cluster_comparison_figure_4.png) in the output directory
- Each view's neighbor graph is built with the correct weighting scheme (integrated uses morphological + transcript + metabolite weights; controls use single-modality weights)
- Cluster assignments are spatially coherent when plotted back onto Visium spot coordinates

## Limitations

- haCCA integration (and hence the comparison) has only been evaluated on spatial transcriptome + spatial metabolome data; other modalities such as image-based sequencing, spatial proteomics, or histological images are not yet supported
- Behavior on higher-resolution data has not been evaluated
- No data accession identifiers or software versions are specified for the evaluation cohort, complicating exact reproduction

## Evidence

- [results] WNN embedding of the integrated multi-module data enabled more dedicated exploration of mouse brain heterogeneity by finding more clusters: [evidence span withheld — anchor status: not_found]
