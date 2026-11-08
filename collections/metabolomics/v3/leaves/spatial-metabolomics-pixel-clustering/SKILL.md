---
name: spatial-metabolomics-pixel-clustering
description: Use when you have a spatial metabolomics feature matrix (rows = pixels
  with X/Y coordinates, columns = m/z ion intensities) and need to segment the tissue
  section into spatial regions or compare them to anatomical references.
when_to_use_negative:
- Input is raw IMS spectral data rather than an already-extracted feature matrix of
  X/Y coordinates and m/z intensities (SMAnalyst requires the feature matrix CSV format)
- The goal is metabolite-level (ion co-expression) clustering or marker ion discovery
  rather than pixel-level tissue segmentation
- The tissue is expected to have far fewer anatomical domains than the default demonstration
  (k=25 was tuned to mouse brain divisions); an inappropriate k can force spurious
  spatial categories
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: SMAnalyst
  role: Web/Shiny platform that implements the four pixel-level clustering algorithms
    (Seurat-LV, Seurat-LM, Seurat-SLM, UMAP-kmeans) and renders the spatial cluster
    map
  repo: https://github.com/mzlab-research/SManalyst
- name: Seurat
  role: R package (Seurat 5.1.0) underlying the Seurat-based pixel clustering options
    within SMAnalyst
- name: spatstat
  role: R package whose quadrat test supplies the noise score (-log10 p-value) used
    to filter ions before clustering
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.3390/biom15111562
    title: 'SMAnalyst: A Web Server for Spatial Metabolomic Data Analysis and Annotation'
  source_content_hash: sha256:86c80304cd99b73dfed3bb478a22b8e4946c5f1fb01367a4d2cb73f0f34c8190
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mzlab-research/SManalyst
  tools:
  - SMAnalyst
  - Seurat
  - spatstat
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# spatial-metabolomics-pixel-clustering

## Summary

Pixel-level clustering of imaging-MS spatial metabolomics feature matrices (e.g., via SMAnalyst's Seurat-based or UMAP-kmeans algorithms) to segment tissue into anatomical regions, as demonstrated by reproducing 25 spatially heterogeneous clusters on a mouse brain coronal section concordant with the Allen Mouse Brain Atlas.

## When to use

Use when you have a spatial metabolomics feature matrix (rows = pixels with X/Y coordinates, columns = m/z ion intensities) and need to segment the tissue section into spatial regions or compare them to anatomical references.

## When NOT to use

- Input is raw IMS spectral data rather than an already-extracted feature matrix of X/Y coordinates and m/z intensities (SMAnalyst requires the feature matrix CSV format)
- The goal is metabolite-level (ion co-expression) clustering or marker ion discovery rather than pixel-level tissue segmentation
- The tissue is expected to have far fewer anatomical domains than the default demonstration (k=25 was tuned to mouse brain divisions); an inappropriate k can force spurious spatial categories

## Inputs

- Feature matrix CSV with columns 1-2 as pixel X/Y coordinates and subsequent columns as ion intensities at each m/z value (one row per spatial pixel), e.g. 14,260 pixels x 3,044 ions from AFAD-ESI (+) mode at 100 um resolution
- Noise-filtered ion subset (optional): ions with spatstat quadrat-test noise score >= 30
- Reference atlas for annotation (e.g., Allen Mouse Brain Atlas)

## Outputs

- Per-pixel cluster label table (assignments for all 14,260 pixels across 25 cluster labels)
- Spatial pixel map of cluster assignments over the tissue section (e.g., Figure 5B-style plot, exportable as PNG)
- Auto-generated ROIs from clustering results usable for downstream differential analysis

## How to apply

Load the feature matrix (e.g., Dataset 1: 14,260 spatial pixels x 3,044 unique ion peaks with X/Y coordinate columns plus m/z intensity columns) into SMAnalyst (Shiny/R, Seurat 5.1.0-based). Optionally first apply noise ion filtering using the spatstat quadrat-test-derived noise score, defined as -log10 of the quadrat-test p-value, with the default threshold score >= 30 (this removes ~39% of ions and improves cluster coherence). Then run pixel-level clustering, choosing one of the four algorithms (Seurat-LV, Seurat-LM, Seurat-SLM, or UMAP-kmeans); for reproducing the mouse brain result, use UMAP-kmeans with k=25. Render the cluster assignments as a spatial pixel map over the tissue section and annotate the identified regions against a reference atlas such as the Allen Mouse Brain Atlas, checking for cerebral cortex, hippocampus, midbrain, hindbrain, and fiber tracts.

## Related tools

- **SMAnalyst** (Web/Shiny platform that implements the four pixel-level clustering algorithms (Seurat-LV, Seurat-LM, Seurat-SLM, UMAP-kmeans) and renders the spatial cluster map) — https://github.com/mzlab-research/SManalyst
- **Seurat** (R package (Seurat 5.1.0) underlying the Seurat-based pixel clustering options within SMAnalyst)
- **spatstat** (R package whose quadrat test supplies the noise score (-log10 p-value) used to filter ions before clustering)

## Evaluation signals

- Output table contains assignments for all 14,260 pixels across exactly 25 cluster labels (when reproducing Dataset 1 with k=25)
- Cluster map qualitatively shows the major anatomical divisions — cerebral cortex, hippocampus, midbrain, hindbrain, and fiber tracts — consistent with the Allen Mouse Brain Atlas
- Applying the default noise threshold (score >= 30) removes approximately 39% of ions, and clusters are visibly more spatially coherent after filtering
- Clusters are spatially contiguous/heterogeneous across the section rather than salt-and-pepper noise, matching the reported 25 spatially heterogeneous categories

## Limitations

- The noise filter is optional but recommended: noise filtering is shown to improve cluster coherence, and the default threshold (score >= 30) discards ~39% of ions, which may drop low-abundance marker ions
- UMAP-kmeans produced 25 spatially heterogeneous categories for the mouse brain dataset; the choice of k and algorithm (of the four available) strongly shapes the spatial segmentation and must be justified per tissue
- Validation is qualitative (atlas concordance of major divisions); cluster identity must be manually annotated against a reference atlas
- The demonstration dataset is a single mouse brain coronal section (AFAD-ESI (+), 100 um); performance on other tissues, polarities, or resolutions is not established by this evidence

## Evidence

- [readme] Pixel-level clustering: "Pixel-level clustering: Identify tissue regions using 4 algorithms: Seurat-LV, Seurat-LM, Seurat-SLM, UMAP-kmeans"
- [readme] input format: "Columns 1-2: Pixel coordinates (X, Y); Subsequent columns: Ion intensities at each m/z value"
- [readme] noise threshold: "Select tissue/background regions (interactive lasso tool); Adjust noise ion threshold (default: score ≥30)"
- [intro] spatial analysis workflow: "Run metabolite/pixel clustering; Compare regions: auto-clustered or manually drawn ROIs"
- [abstract] scale capability: "Tested with a mouse brain dataset, SMAnalyst efficiently handles large-scale data (e.g., >14,000 pixels, >3000 ion peaks)"
