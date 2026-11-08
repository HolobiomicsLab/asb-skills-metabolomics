---
name: metabolite-spatial-coexpression-clustering
description: Use when a spatial metabolomics feature matrix (pixel X/Y coordinates
  x m/z ion intensities, e.g. AFAD-ESI 100 um data) is QC-filtered and the goal is
  metabolite-level spatial co-expression pattern discovery, not pixel-level region
  clustering.
when_to_use_negative:
- The analysis goal is pixel-level tissue region identification — SMAnalyst separately
  offers Seurat-LV/Seurat-LM/Seurat-SLM/UMAP-kmeans pixel clustering, which is a different
  operation
- Input is raw imaging-MS spectra rather than a feature matrix of pixel coordinates
  and ion intensities
- Ions have not passed QC2 noise-ion filtering (default noise score >= 30), so co-expression
  clusters would be driven by noise peaks
edam_operation: http://edamontology.org/operation_3432
tools:
- name: SMAnalyst
  role: Web/R platform hosting the metabolite-level spatial co-expression clustering
    step within its Spatial Analysis workflow (metabolite/pixel clustering)
- name: SpaGene
  role: Clustering engine used by SMAnalyst to group metabolic ions into spatial co-expression
    patterns; installed as an R dependency
- name: ggplot2
  role: Rendering of the spatial pattern map over the tissue section
- name: tidyverse
  role: Data manipulation of the ion-by-pixel matrix and cluster membership tables
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.3390/biom15111562
    title: 'SMAnalyst: A Web Server for Spatial Metabolomic Data Analysis and Annotation'
  source_content_hash: sha256:843c6bbcc46a867fbb11d8f08440d6617336ba0b3661bc7e741c63c0f0e08d14
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R (>=)
  - SMAnalyst
  - SpaGene
  - ggplot2
  - tidyverse
  edam_operation: http://edamontology.org/operation_3432
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# metabolite-spatial-coexpression-clustering

## Summary

Cluster metabolic ions by spatial co-expression across tissue pixels — via SpaGene inside SMAnalyst — to define metabolite spatial expression patterns on spatial metabolomics feature matrices, then map and export the ions assigned to each pattern.

## When to use

Use when a spatial metabolomics feature matrix (pixel X/Y coordinates x m/z ion intensities, e.g. AFAD-ESI 100 um data) is QC-filtered and the goal is metabolite-level spatial co-expression pattern discovery, not pixel-level region clustering.

## When NOT to use

- The analysis goal is pixel-level tissue region identification — SMAnalyst separately offers Seurat-LV/Seurat-LM/Seurat-SLM/UMAP-kmeans pixel clustering, which is a different operation
- Input is raw imaging-MS spectra rather than a feature matrix of pixel coordinates and ion intensities
- Ions have not passed QC2 noise-ion filtering (default noise score >= 30), so co-expression clusters would be driven by noise peaks

## Inputs

- Feature matrix CSV: columns 1-2 pixel X/Y coordinates, remaining columns ion intensities per m/z (one row per spatial pixel)
- Tissue-pixel selections from QC (lasso-selected tissue/background regions)
- Noise-ion-filtered ion set (noise score threshold >= 30 retained ions)

## Outputs

- Table of ions assigned to each metabolite spatial expression pattern cluster (e.g. eight clusters)
- Spatial map image (PNG) overlaying the co-expression pattern groups on the tissue section
- Per-pattern ion membership lists for downstream annotation or differential analysis

## How to apply

Load the feature matrix (e.g. 14,260 spatial pixels x 3,044 unique ion peaks from the mouse brain Dataset 1, NGDC OMIX OMIX009541) in SMAnalyst's required CSV layout where columns 1-2 are pixel coordinates and later columns are ion intensities. First pass QC2 noise-ion filtering with the default threshold score >= 30 and retain tissue pixels. Then run the SpaGene metabolite-level clustering on the ion intensity matrix to group ions by spatial co-expression; the reference mouse-brain run resolves eight major metabolite spatial expression patterns. Export the per-cluster ion membership table and render the spatial map of the pattern groups (analogous to Figure 5A / Supplementary Figure S11). Validate against the reported biology: Pattern 4 enriched in cerebral cortex (low in midbrain), Pattern 7 uniquely enriched at tissue edges.

## Related tools

- **SMAnalyst** (Web/R platform hosting the metabolite-level spatial co-expression clustering step within its Spatial Analysis workflow (metabolite/pixel clustering))
- **SpaGene** (Clustering engine used by SMAnalyst to group metabolic ions into spatial co-expression patterns; installed as an R dependency)
- **ggplot2** (Rendering of the spatial pattern map over the tissue section)
- **tidyverse** (Data manipulation of the ion-by-pixel matrix and cluster membership tables)

## Examples

```
install.packages(c("ggplot2", "tidyverse", "SpaGene", "Seurat"))  # then run metabolite-level clustering via the SMAnalyst Spatial Analysis tab
```

## Evaluation signals

- Clustering resolves exactly the expected number of major metabolite spatial expression patterns (eight in the mouse brain reference run)
- Pattern 4 shows high expression in the cerebral cortex and low expression in the midbrain, matching the reported result
- Pattern 7 is uniquely enriched in tissue edge regions, matching the reported result
- The exported per-cluster ion table partitions the retained ions (post noise-score >= 30 filter, e.g. ~3,044 ions on Dataset 1) consistently with the rendered pattern map
- Pattern spatial maps visually align with tissue anatomy on the 100 um mouse brain coronal section

## Limitations

- Requires SMAnalyst's specific feature-matrix CSV format (X, Y pixel coordinates followed by ion intensity columns); other layouts must be converted first
- Cluster count and pattern numbering are algorithm/data dependent — the eight-pattern, Pattern 4 cortex / Pattern 7 edge result is specific to the mouse brain Dataset 1 run and does not transfer automatically
- Tested scale is large but bounded: SMAnalyst was validated on >14,000 pixels and >3,000 ion peaks; far larger datasets may strain the R/Shiny implementation
- Co-expression clustering is metabolite-level only; it does not by itself assign anatomical ROIs, which requires pixel-level clustering or manual lasso ROIs

## Evidence

- [readme] Metabolite-level clustering: Group ions by spatial co-expression patterns: "**Metabolite-level clustering**: Group ions by spatial co-expression patterns"
- [abstract] dual-dimension spatial pattern discovery: "dual-dimension spatial pattern discovery (metabolite co-expression and pixel clustering)"
- [readme] noise ion threshold default: "*   Adjust noise ion threshold (default: score ≥30)"
- [readme] SpaGene dependency listing: ""qvalue", "SpaGene", "data.table", "Matrix", "rjson", "RColorBrewer","
- [readme] example dataset dimensions and accession: "*   Acquisition: AFAD-ESI (+) mode, 100μm resolution
*   Content: 14,260 pixels × 3,044 ions"
