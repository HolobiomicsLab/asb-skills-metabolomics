---
name: spatial-pattern-annotation-against-brain-anatomy
description: Use when a spatial metabolomics feature matrix (pixel × ion intensities,
  e.g. AFAD-ESI+ 100 um mouse brain section) has been ion-clustered and clusters must
  be labeled/validated against anatomical regions like cortex or tissue edges.
when_to_use_negative:
- The goal is pixel-level tissue region clustering rather than metabolite-level co-expression
  annotation — SMAnalyst provides separate pixel clustering algorithms (Seurat-LV,
  Seurat-LM, Seurat-SLM, UMAP-kmeans) for that purpose
- Input is raw spectra or peak-picked data rather than the required SMAnalyst feature
  matrix format (pixel coordinates + ion intensity columns)
- The goal is metabolite identity annotation against databases (HMDB/KEGG/LIPIDMAPS)
  via mass accuracy/isotope/adduct scoring — that is SMAnalyst's annotation scoring
  module, not spatial pattern annotation
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: SMAnalyst
  role: Integrated web/R platform that hosts the metabolite-level spatial co-expression
    clustering, spatial map rendering, CSV/PNG export, and the Tutorial Panel example
    data
  repo: https://github.com/mzlab-research/SManalyst
- name: SpaGene
  role: Clustering algorithm used by SMAnalyst to group metabolic ions by spatial
    co-expression into pattern clusters
- name: ggplot2
  role: Rendering of the spatial expression-pattern map over the tissue section
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.3390/biom15111562
    title: 'SMAnalyst: A Web Server for Spatial Metabolomic Data Analysis and Annotation'
  source_content_hash: sha256:efa572056a838d4bdb99487edf5fc002d01567ec473f96d63bb014daf8f26598
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mzlab-research/SManalyst
  tools:
  - R (>=)
  - SMAnalyst
  - SpaGene
  - ggplot2
  - tidyverse
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9285714285714286
n_claims: 14
---

# spatial-pattern-annotation-against-brain-anatomy

## Summary

Annotates metabolite-level spatial co-expression clusters (SpaGene within SMAnalyst) of a mouse brain MSI feature matrix against gross tissue anatomy — verifying that resolved spatial expression patterns match anatomical regions such as cerebral cortex (high-expression) and tissue edge regions.

## When to use

Use when a spatial metabolomics feature matrix (pixel × ion intensities, e.g. AFAD-ESI+ 100 um mouse brain section) has been ion-clustered and clusters must be labeled/validated against anatomical regions like cortex or tissue edges.

## When NOT to use

- The goal is pixel-level tissue region clustering rather than metabolite-level co-expression annotation — SMAnalyst provides separate pixel clustering algorithms (Seurat-LV, Seurat-LM, Seurat-SLM, UMAP-kmeans) for that purpose
- Input is raw spectra or peak-picked data rather than the required SMAnalyst feature matrix format (pixel coordinates + ion intensity columns)
- The goal is metabolite identity annotation against databases (HMDB/KEGG/LIPIDMAPS) via mass accuracy/isotope/adduct scoring — that is SMAnalyst's annotation scoring module, not spatial pattern annotation

## Inputs

- Feature matrix CSV: columns 1-2 are pixel X/Y coordinates, remaining columns are ion intensities per m/z, one row per spatial pixel
- Mouse brain coronal section MSI dataset (AFAD-ESI (+) mode, 100 um resolution, 14,260 pixels × 3,044 ions), e.g. NGDC OMIX OMIX009541 or SMAnalyst's built-in Tutorial Panel example data

## Outputs

- Table of ion members belonging to each metabolite spatial expression pattern cluster
- Spatial map image (e.g. PNG) of the metabolite co-expression patterns overlaid on the tissue section
- CSV exports of clustering results downloadable from SMAnalyst

## How to apply

Load the feature matrix (Dataset 1: 14,260 spatial pixels × 3,044 ion peaks, AFAD-ESI positive mode, 100 um pixel size) from OMIX009541 or SMAnalyst's Tutorial Panel example data. Preprocess by retaining tissue pixels and ions passing the noise-ion filter (noise score threshold >= 30, the SMAnalyst default) so that downstream clustering operates on tissue-relevant ions. Run the SpaGene metabolite-level clustering (metabolite co-expression pattern discovery, one of SMAnalyst's dual-dimension spatial pattern analyses, distinct from pixel clustering via Seurat-LV/LM/SLM or UMAP-kmeans) to resolve the major metabolite spatial expression patterns — eight in the reference result. Export per-cluster ion membership tables and render a spatial map of the patterns over the tissue (analogous to Figure 5A / Supplementary Figure S11), then read the maps anatomically: compare each pattern's spatial envelope to the brain section's anatomy, expecting Pattern 4 to be high in the cerebral cortex (and low in midbrain) and Pattern 7 uniquely enriched at tissue edge regions.

## Related tools

- **SMAnalyst** (Integrated web/R platform that hosts the metabolite-level spatial co-expression clustering, spatial map rendering, CSV/PNG export, and the Tutorial Panel example data) — https://github.com/mzlab-research/SManalyst
- **SpaGene** (Clustering algorithm used by SMAnalyst to group metabolic ions by spatial co-expression into pattern clusters)
- **ggplot2** (Rendering of the spatial expression-pattern map over the tissue section)

## Examples

```
install.packages(c("tidyverse", "ggplot2", "SpaGene", "Seurat", "data.table"))  # R packages SMAnalyst requires to run SpaGene metabolite-level clustering
```

## Evaluation signals

- Clustering resolves exactly eight major metabolite spatial expression patterns on Dataset 1
- Pattern 4 shows high expression in the cerebral cortex and low expression in the midbrain
- Pattern 7 is uniquely enriched in tissue edge regions of the section
- Only ions passing the noise-ion filter (score >= 30) and tissue pixels are included before clustering
- Input/output dimensionality is consistent: ~14,260 pixels × 3,044 ions in, per-cluster ion membership tables and spatial maps out

## Limitations

- Anatomical validation relies on the structure of a mouse brain coronal section; the pattern-to-anatomy mapping (e.g. cortex vs. edge) is tissue- and section-specific and does not transfer to arbitrary tissues
- Cluster count and pattern numbering depend on the preprocessing state (tissue-pixel retention, noise score >= 30) and clustering resolution; deviations from the paper's preprocessing can change which pattern is cortex- or edge-enriched
- SMAnalyst provides no changelog/substantive discussion of known failure modes, so failure behavior must be checked empirically against the reference patterns

## Evidence

- [readme] Metabolite-level clustering: Group ions by spatial co-expression patterns: "Metabolite-level clustering: Group ions by spatial co-expression patterns"
- [abstract] dual-dimension spatial pattern discovery (metabolite co-expression and pixel clustering): "dual-dimension spatial pattern discovery (metabolite co-expression and pixel clustering)"
- [intro] Adjust noise ion threshold (default: score ≥30): "Adjust noise ion threshold (default: score ≥30)"
- [readme] Acquisition: AFAD-ESI (+) mode, 100μm resolution: "Acquisition: AFAD-ESI (+) mode, 100μm resolution"
- [readme] Built-in dataset in SManalyst (Tutorial Panel → Example Data): "Built-in dataset in SManalyst (Tutorial Panel → Example Data)"
