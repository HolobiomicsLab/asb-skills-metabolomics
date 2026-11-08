---
name: kmeans-cluster-partitioning
description: Use when you have a spatial metabolomics feature matrix (pixels x m/z
  ion intensities) and need to segment tissue into anatomical or spatially coherent
  regions via unsupervised kmeans partitioning on UMAP embeddings.
when_to_use_negative:
- Input is already a segmented or annotated ROI table and no unsupervised pixel partitioning
  is needed
- The goal is metabolite-level (ion-by-spatial-pattern) clustering rather than pixel-level
  tissue-region clustering — SMAnalyst provides a separate metabolite co-expression
  clustering mode
- Data is raw mass-spectrometry imaging rather than a pre-built feature matrix; SMAnalyst
  expects CSV feature matrices, not mzML/IMS vendor formats
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: SMAnalyst
  role: Web platform hosting the UMAP-kmeans pixel clustering method alongside Seurat-LV,
    Seurat-LM, and Seurat-SLM options
  repo: mzlab-research/SManalyst
- name: Seurat
  role: R package (5.1.0) underpinning SMAnalyst's Seurat-based pixel clustering algorithms
    including UMAP-kmeans
- name: R
  role: Runtime (>= v4.1.0) for local installation of the SManalyst Shiny platform
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.3390/biom15111562
    title: 'SMAnalyst: A Web Server for Spatial Metabolomic Data Analysis and Annotation'
  source_content_hash: sha256:72fd154c38e131017145eb0e13e4db79d6f8abe2ef705d02907d731c5533102c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mzlab-research/SManalyst
  tools:
  - R
  - SMAnalyst
  - Seurat
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# kmeans-cluster-partitioning

## Summary

Pixel-level clustering of spatial metabolomics feature matrices using SMAnalyst's UMAP-kmeans option (one of four Seurat 5.1.0-based algorithms) to partition tissue pixels into spatially coherent clusters, e.g. k=25 clusters recovering cortex, hippocampus, midbrain, hindbrain, and fiber tracts in mouse brain sections.

## When to use

Use when you have a spatial metabolomics feature matrix (pixels x m/z ion intensities) and need to segment tissue into anatomical or spatially coherent regions via unsupervised kmeans partitioning on UMAP embeddings.

## When NOT to use

- Input is already a segmented or annotated ROI table and no unsupervised pixel partitioning is needed
- The goal is metabolite-level (ion-by-spatial-pattern) clustering rather than pixel-level tissue-region clustering — SMAnalyst provides a separate metabolite co-expression clustering mode
- Data is raw mass-spectrometry imaging rather than a pre-built feature matrix; SMAnalyst expects CSV feature matrices, not mzML/IMS vendor formats

## Inputs

- Feature matrix CSV with pixel coordinates (X, Y) in columns 1-2 and ion intensity columns per m/z (e.g. 14,260 pixels x 3,044 unique ion peaks from a mouse brain coronal section, AFAD-ESI (+) mode, 100 um resolution)
- Number of clusters k (e.g. k=25)
- Optional: noise-filtered ion subset defined by noise score >= 30 (spatstat quadrat test)

## Outputs

- Cluster label assignment for every pixel across k=25 cluster labels (verifiable as a table covering all 14,260 pixels)
- Spatial pixel map of cluster assignments over the tissue section (PNG export, analogous to Figure 5B)
- Cluster annotations matching major anatomical divisions (cerebral cortex, hippocampus, midbrain, hindbrain, fiber tracts)

## How to apply

Load the feature matrix (columns 1-2 are X/Y pixel coordinates, subsequent columns are ion intensities at each m/z value; each row is one spatial pixel). Optionally restrict to ions passing the noise filter, where the noise score is -log10 of the spatstat quadrat-test p-value with default threshold score >= 30 (this removed ~39% of ions and improved cluster coherence). Run the UMAP-kmeans pixel clustering method from SMAnalyst's four Seurat 5.1.0-based options (Seurat-LV, Seurat-LM, Seurat-SLM, UMAP-kmeans), setting k=25 to match the reported 25 spatially heterogeneous categories. Render cluster assignments as a spatial pixel map and compare against a reference atlas (e.g. Allen Mouse Brain Atlas) to annotate major anatomical divisions.

## Related tools

- **SMAnalyst** (Web platform hosting the UMAP-kmeans pixel clustering method alongside Seurat-LV, Seurat-LM, and Seurat-SLM options) — mzlab-research/SManalyst
- **Seurat** (R package (5.1.0) underpinning SMAnalyst's Seurat-based pixel clustering algorithms including UMAP-kmeans)
- **R** (Runtime (>= v4.1.0) for local installation of the SManalyst Shiny platform)

## Examples

```
install.packages(c("shinydashboard","ggplot2","shiny","shinythemes","tidyverse","reshape2","pheatmap","shinyjs","plotly","ggnewscale","cowplot","spatstat","Seurat"))  # then run UMAP-kmeans clustering via SManalyst app (Tutorial Panel -> Example Data, k=25)
```

## Evaluation signals

- Output table contains assignments for all 14,260 pixels across exactly 25 cluster labels (k=25)
- Spatial cluster map shows recognizable major anatomical divisions — cerebral cortex, hippocampus, midbrain, hindbrain, fiber tracts — consistent with the Allen Mouse Brain Atlas
- Applying the noise filter (spatstat quadrat-test score >= 30, removing ~39% of ions) should improve cluster spatial coherence relative to unfiltered ions
- Platform handles the full-scale input (>14,000 pixels, >3,000 ion peaks) without failure, matching the reported benchmark on the mouse brain dataset

## Limitations

- Cluster number k must be chosen by the user (k=25 was tuned to this mouse brain dataset); other tissues may need different k
- Clusters are spatially heterogeneous categories, not guaranteed 1:1 with anatomical regions; atlas concordance is qualitative rather than quantitative in the reported workflow
- Noise filtering is optional but skipping it can degrade cluster coherence; the default threshold (score >= 30) is dataset-dependent

## Evidence

- [readme] Pixel-level clustering: "Pixel-level clustering: Identify tissue regions using 4 algorithms: Seurat-LV, Seurat-LM, Seurat-SLM, UMAP-kmeans"
- [readme] dataset scale: "Content: 14,260 pixels × 3,044 ions"
- [readme] noise threshold: "Adjust noise ion threshold (default: score ≥30)"
- [abstract] benchmark: "Tested with a mouse brain dataset, SMAnalyst efficiently handles large-scale data (e.g., >14,000 pixels, >3000 ion peaks)"
