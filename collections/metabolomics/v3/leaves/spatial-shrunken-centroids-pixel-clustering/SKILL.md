---
name: spatial-shrunken-centroids-pixel-clustering
description: Use when you have a pixel-by-ion feature matrix from imaging MS (e.g.,
  14,260 pixels x 3,044 ion peaks) and need anatomically meaningful pixel clusters
  or region-of-interest definitions to benchmark or replace web-tool clustering.
when_to_use_negative:
- You are already working inside SMAnalyst and can use its built-in pixel clustering
  algorithms (Seurat-LV, Seurat-LM, Seurat-SLM, UMAP-kmeans) instead of switching
  to R/Cardinal
- The input is raw imaging MS data rather than a preprocessed feature matrix — Cardinal
  shrunken centroids in this workflow expects the pixel-by-ion intensity table
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_0121
tools:
- name: Cardinal
  role: Provides the spatial shrunken centroids method that assigns pixels into 25
    spatial categories used as region-of-interest definitions
- name: SMAnalyst
  role: Supplies the example mouse brain feature matrix (Tutorial Panel example data)
    and the clustering/differential results used as the benchmark comparison target
    (89% overlap)
  repo: https://github.com/mzlab-research/SManalyst
- name: R
  role: Runtime environment for running Cardinal's spatial shrunken centroids clustering
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.3390/biom15111562
    title: 'SMAnalyst: A Web Server for Spatial Metabolomic Data Analysis and Annotation'
  source_content_hash: sha256:3c2243792dfcad5a71eaa86a095ac18a666b6b28c748fb9bdb95b341d86bf609
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mzlab-research/SManalyst
  tools:
  - Cardinal
  - R
  - SMAnalyst
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# spatial-shrunken-centroids-pixel-clustering

## Summary

Apply Cardinal's spatial shrunken centroids method in R to assign pixels of a spatial metabolomics feature matrix into spatially coherent clusters that map to anatomical regions (e.g., 25 categories on a mouse brain section), providing cluster-defined ROIs for downstream differential analysis.

## When to use

Use when you have a pixel-by-ion feature matrix from imaging MS (e.g., 14,260 pixels x 3,044 ion peaks) and need anatomically meaningful pixel clusters or region-of-interest definitions to benchmark or replace web-tool clustering.

## When NOT to use

- You are already working inside SMAnalyst and can use its built-in pixel clustering algorithms (Seurat-LV, Seurat-LM, Seurat-SLM, UMAP-kmeans) instead of switching to R/Cardinal
- The input is raw imaging MS data rather than a preprocessed feature matrix — Cardinal shrunken centroids in this workflow expects the pixel-by-ion intensity table

## Inputs

- Feature matrix CSV of spatial metabolomics pixel data (14,260 pixels x 3,044 ion peaks; X/Y coordinates in columns 1-2, m/z ion intensities in remaining columns)
- Mouse brain coronal section dataset acquired in AFAD-ESI (+) mode at 100 um pixel size (NGDC OMIX OMIX009541 or SMAnalyst Tutorial Panel example data)

## Outputs

- Per-pixel spatial cluster assignments (25 spatial categories forming anatomically recognizable regions)
- Cluster-defined region groups (e.g., midbrain MB and hippocampal HIP pixel sets) usable as samples for differential analysis

## How to apply

Load the feature matrix (columns 1-2 are pixel X/Y coordinates, subsequent columns are ion intensities at each m/z value) into R and run Cardinal's spatial shrunken centroids to assign pixels into a chosen number of spatial categories — in the SMAnalyst benchmark, 25 categories produced anatomically recognizable mouse brain clusters. Assign the resulting clusters to biological regions (e.g., midbrain MB vs hippocampal HIP groups) so that all pixels in a cluster group serve as samples. This clustering-then-differential design (Wilcoxon rank-sum test with Benjamini-Hochberg FDR correction, FC > 2 or < 0.5 and FDR < 0.01) is used to benchmark against SMAnalyst's own pixel clustering (Seurat-LV/LM/SLM, UMAP-kmeans) and reached an 89% overlap in differential metabolites, validating clustering-based ROI definition.

## Related tools

- **Cardinal** (Provides the spatial shrunken centroids method that assigns pixels into 25 spatial categories used as region-of-interest definitions)
- **SMAnalyst** (Supplies the example mouse brain feature matrix (Tutorial Panel example data) and the clustering/differential results used as the benchmark comparison target (89% overlap)) — https://github.com/mzlab-research/SManalyst
- **R** (Runtime environment for running Cardinal's spatial shrunken centroids clustering)

## Examples

```
library(Cardinal); ms <- read.csv("feature_matrix.csv"); ms <- as.MSImagingExperiment(ms, coord=coord(ms[,1:2])); clusters <- spatialShrunkenCentroids(ms, r=1, k=25); table(clusters$classes)
```

## Evaluation signals

- The 25 spatial clusters correspond to anatomically recognizable brain structures when visualized on the pixel grid
- Differential analysis on the Cardinal-defined MB vs HIP groups (Wilcoxon rank-sum + BH FDR, FC > 2 or < 0.5, FDR < 0.01) yields 90 upregulated (MB) and 94 downregulated (HIP) metabolites
- Overlap between Cardinal-derived differential metabolites and SMAnalyst's differential results reaches 89% (Supplementary Figure S16B,C)
- Cluster count matches the benchmark setting of 25 spatial categories on the 14,260-pixel mouse brain dataset

## Limitations

- Cluster-defined ROI quality depends on choosing a suitable number of categories (25 in this benchmark); other counts may not yield anatomically recognizable regions
- Treating all pixels in a cluster group as samples for the Wilcoxon test ignores within-region spatial correlation, which is inherent to this benchmarking design
- The benchmark values (90/94 metabolites, 89% overlap) are specific to the AFAD-ESI (+) 100 um mouse brain dataset (OMIX009541) and may not transfer to other tissues or acquisition modes

## Evidence

- [other] run Cardinal's spatial shrunken centroids method on the feature matrix to assign the pixels into 25 spatial categories, producing anatomically recognizable clusters: [evidence span withheld — anchor status: not_found]
- [other] the overlap of differential metabolites with SMAnalyst's results reached 89%: [evidence span withheld — anchor status: not_found]
- [readme] Pixel-level clustering: Identify tissue regions using 4 algorithms: "**Pixel-level clustering**: Identify tissue regions using 4 algorithms:

    *   Seurat-LV, Seurat-LM, Seurat-SLM, UMAP-kmeans"
- [readme] Example data: Mouse brain coronal section, AFAD-ESI (+) mode, 100um resolution, 14,260 pixels x 3,044 ions: "Test dataset: **Mouse brain coronal section**

*   Acquisition: AFAD-ESI (+) mode, 100μm resolution
*   Content: 14,260 pixels × 3,044 ions"
