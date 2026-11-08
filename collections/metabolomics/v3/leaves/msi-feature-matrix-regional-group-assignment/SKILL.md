---
name: msi-feature-matrix-regional-group-assignment
description: Use when a spatial metabolomics feature matrix (pixels x m/z ion peaks)
  needs pixel-level regional grouping before Wilcoxon/FC-based differential comparison,
  e.g. benchmarking SMAnalyst against Cardinal-defined anatomical clusters.
when_to_use_negative:
- The input is raw mass-spectrometry imaging data rather than an already-extracted
  pixel x m/z feature matrix (SMAnalyst requires a feature matrix in CSV format)
- The comparison regions are already defined by manual ROI drawing or SMAnalyst's
  auto-clustering, so no external clustering benchmark is needed
- The tissue lacks recognizable anatomical structure, making cluster-to-region assignment
  (e.g., MB vs HIP) unjustifiable
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_0092
tools:
- name: Cardinal
  role: Performs spatial shrunken centroids clustering to assign pixels into 25 spatial
    categories used to define the MB and HIP comparison regions
- name: SMAnalyst
  role: Source of the mouse brain feature matrix (Tutorial Panel Example Data) and
    reference differential-analysis results (89% overlap) against which the Cardinal-based
    grouping is benchmarked
  repo: mzlab-research/SManalyst
- name: R (>= v4.1.0)
  role: Execution environment for Cardinal clustering and downstream Wilcoxon differential
    analysis
  repo: mzlab-research/SManalyst
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.3390/biom15111562
    title: 'SMAnalyst: A Web Server for Spatial Metabolomic Data Analysis and Annotation'
  source_content_hash: sha256:9758bdca73ddcd55867095949098d8957acb8c02a4c87ae0dd84464353f07bb5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mzlab-research/SManalyst
  tools:
  - Cardinal
  - R (>= v4.1.0)
  - SMAnalyst
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# msi-feature-matrix-regional-group-assignment

## Summary

Assign pixels of an MSI feature matrix to anatomically meaningful region groups (e.g., midbrain vs hippocampus) using an external clustering tool such as Cardinal's spatial shrunken centroids, so that downstream differential analysis treats all pixels in a region as replicate samples.

## When to use

Use when a spatial metabolomics feature matrix (pixels x m/z ion peaks) needs pixel-level regional grouping before Wilcoxon/FC-based differential comparison, e.g. benchmarking SMAnalyst against Cardinal-defined anatomical clusters.

## When NOT to use

- The input is raw mass-spectrometry imaging data rather than an already-extracted pixel x m/z feature matrix (SMAnalyst requires a feature matrix in CSV format)
- The comparison regions are already defined by manual ROI drawing or SMAnalyst's auto-clustering, so no external clustering benchmark is needed
- The tissue lacks recognizable anatomical structure, making cluster-to-region assignment (e.g., MB vs HIP) unjustifiable

## Inputs

- Feature matrix CSV with pixel coordinates (X, Y) in columns 1-2 and ion intensities at each m/z in subsequent columns (e.g., 14,260 pixels x 3,044 ion peaks, AFAD-ESI (+), 100 um pixel size)
- Raw data from NGDC OMIX Repository ID: OMIX009541 or SMAnalyst Tutorial Panel Example Data

## Outputs

- Pixel-to-cluster assignments (25 spatial categories from Cardinal spatial shrunken centroids)
- Region-group labels (e.g., midbrain MB vs hippocampal HIP) per pixel for differential analysis
- Differential metabolite counts per direction (e.g., 90 upregulated in MB, 94 downregulated in HIP after FC > 2 or < 0.5 and FDR-adjusted p < 0.01)

## How to apply

Load the feature matrix (e.g., the 14,260 pixels x 3,044 ion peaks mouse brain dataset, AFAD-ESI positive mode, 100 um pixel size, from SMAnalyst's Tutorial Panel example data or NGDC OMIX OMIX009541) into R. Run Cardinal's spatial shrunken centroids method to assign pixels into 25 spatial categories that produce anatomically recognizable clusters. Assign Cardinal-defined clusters to the comparison regions — here the midbrain (MB) group and the hippocampal (HIP) group. Treat every pixel within a group as a sample for the subsequent Wilcoxon rank-sum test with Benjamini-Hochberg FDR correction. The rationale is that clustering-derived ROIs provide an objective, tool-external definition of regions that can be cross-checked against SMAnalyst's auto-clustered or manually drawn ROIs.

## Related tools

- **Cardinal** (Performs spatial shrunken centroids clustering to assign pixels into 25 spatial categories used to define the MB and HIP comparison regions)
- **SMAnalyst** (Source of the mouse brain feature matrix (Tutorial Panel Example Data) and reference differential-analysis results (89% overlap) against which the Cardinal-based grouping is benchmarked) — mzlab-research/SManalyst
- **R (>= v4.1.0)** (Execution environment for Cardinal clustering and downstream Wilcoxon differential analysis) — mzlab-research/SManalyst

## Examples

```
library(Cardinal); ms <- readImzML("mouse_brain"); clusters <- spatialShrunkenCentroids(ms, r=2, k=25); groups <- ifelse(clusters %in% mb_clusters, "MB", "HIP")
```

## Evaluation signals

- Clustering yields 25 spatial categories that map to anatomically recognizable brain structures (midbrain, hippocampus)
- Wilcoxon + BH FDR differential analysis with FC > 2 or < 0.5 and FDR-adjusted p-value < 0.01 returns 90 upregulated (MB) and 94 downregulated (HIP) metabolites
- Overlap between the Cardinal-derived differential metabolite set and SMAnalyst's differential results reaches 89%
- Pixel count consistency: all 14,260 pixels of the input feature matrix receive exactly one region-group label
- Results reproduce Supplementary Figure S16B,C of the paper

## Limitations

- Pixel-level clustering algorithms are tool-dependent; Cardinal's 25-cluster solution must align with SMAnalyst's Seurat-LV/LM/SLM or UMAP-kmeans outputs only at the differential-metabolite overlap level (89%), not necessarily per-pixel
- Treating all pixels in a group as independent samples ignores spatial autocorrelation, which may inflate significance in the Wilcoxon rank-sum test
- Cluster-to-region assignment relies on anatomical recognizability; ambiguous clusters require manual judgment about which pixels belong to MB vs HIP

## Evidence

- [other] run Cardinal's spatial shrunken centroids method on the feature matrix to assign the pixels into 25 spatial categories, producing anatomically recognizable clusters: [evidence span withheld — anchor status: not_found]
- [other] treating all pixels in a group as samples: [evidence span withheld — anchor status: not_found]
- [readme] Pixel-level clustering: Identify tissue regions using 4 algorithms: "**Pixel-level clustering**: Identify tissue regions using 4 algorithms:

    *   Seurat-LV, Seurat-LM, Seurat-SLM, UMAP-kmeans"
- [readme] SMAnalyst requires a feature matrix in CSV format: "SManalyst requires a **feature matrix** in **CSV format**:

*   **Columns 1-2**: Pixel coordinates (X, Y)"
- [abstract] flexible differential analysis (cluster- or user-defined regions): "It also offers flexible differential analysis (cluster- or user-defined regions)."
