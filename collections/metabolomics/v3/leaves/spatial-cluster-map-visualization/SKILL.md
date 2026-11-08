---
name: spatial-cluster-map-visualization
description: Use when a spatial metabolomics feature matrix (pixels x m/z intensities)
  has been clustered at the pixel level and you need to visualize cluster assignments
  as a tissue map and validate segmentation against anatomical divisions (cortex,
  hippocampus, midbrain, hindbrain, fiber tracts).
when_to_use_negative:
- Input is raw mass-spec imaging data (e.g., imzML) rather than the pre-extracted
  feature matrix CSV that SMAnalyst requires
- The goal is marker ion discovery or peak picking rather than tissue-region segmentation
  (e.g., SmartGate's iterative graph-attention autoencoder peak picking)
- Only single-ion distribution images or RGB overlays are needed, without multi-cluster
  region segmentation
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: SMAnalyst
  role: Web platform that runs the four pixel-level clustering algorithms (Seurat-LV,
    Seurat-LM, Seurat-SLM, UMAP-kmeans) and renders/exportspatial cluster maps (CSV
    tables, PNG images)
  repo: https://github.com/mzlab-research/SManalyst
- name: Seurat
  role: R package providing the underlying clustering engines (including UMAP-kmeans
    option) used by SMAnalyst for pixel-level clustering
- name: ggplot2
  role: R plotting dependency used by SMAnalyst for rendering spatial visualization
    outputs
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.3390/biom15111562
    title: 'SMAnalyst: A Web Server for Spatial Metabolomic Data Analysis and Annotation'
  source_content_hash: sha256:99925d946967ea136224d3e46cef344a847e804a5c727854b542a67a76d3ec4b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mzlab-research/SManalyst
  tools:
  - SMAnalyst
  - Seurat
  - ggplot2
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# spatial-cluster-map-visualization

## Summary

Render pixel-level clustering results from spatial metabolomics data as a spatial map over the tissue section (e.g., SMAnalyst's UMAP-kmeans with k=25 on a 14,260-pixel mouse brain section) and annotate regions against a reference anatomical atlas such as the Allen Mouse Brain Atlas.

## When to use

Use when a spatial metabolomics feature matrix (pixels x m/z intensities) has been clustered at the pixel level and you need to visualize cluster assignments as a tissue map and validate segmentation against anatomical divisions (cortex, hippocampus, midbrain, hindbrain, fiber tracts).

## When NOT to use

- Input is raw mass-spec imaging data (e.g., imzML) rather than the pre-extracted feature matrix CSV that SMAnalyst requires
- The goal is marker ion discovery or peak picking rather than tissue-region segmentation (e.g., SmartGate's iterative graph-attention autoencoder peak picking)
- Only single-ion distribution images or RGB overlays are needed, without multi-cluster region segmentation

## Inputs

- Feature matrix CSV with pixel X/Y coordinate columns followed by m/z intensity columns (one row per spatial pixel, e.g., 14,260 pixels x 3,044 ions)
- Chosen pixel clustering method and cluster count k (e.g., UMAP-kmeans, k=25)
- Optional: ion list passing the noise filter (score >= 30 from spatstat quadrat test)
- Reference anatomical atlas (e.g., Allen Mouse Brain Atlas)

## Outputs

- Per-pixel cluster assignment table covering all 14,260 pixels across 25 cluster labels
- Spatial cluster map rendered over the tissue section (PNG image)
- Annotated anatomical division labels (cerebral cortex, hippocampus, midbrain, hindbrain, fiber tracts) mapped onto clusters
- Exportable CSV table of cluster assignments

## How to apply

Load the feature matrix CSV (columns 1-2 are pixel X/Y coordinates, subsequent columns are ion intensities at each m/z value), optionally restricting to ions passing the noise filter (spatstat quadrat-test score >= 30, which removes ~39% of ions and improves cluster coherence). Run pixel-level clustering in SMAnalyst using one of the four Seurat-based algorithms — Seurat-LV, Seurat-LM, Seurat-SLM, or UMAP-kmeans — choosing k to match expected tissue heterogeneity (k=25 for the mouse brain Dataset 1). Render the per-pixel cluster labels as a spatial map over the tissue section (as in Figure 5B), then compare the clusters with the Allen Mouse Brain Atlas to annotate major anatomical divisions. Rationale: spatially coherent cluster maps that recapitulate known anatomy are evidence the segmentation reflects real metabolic heterogeneity rather than noise.

## Related tools

- **SMAnalyst** (Web platform that runs the four pixel-level clustering algorithms (Seurat-LV, Seurat-LM, Seurat-SLM, UMAP-kmeans) and renders/exportspatial cluster maps (CSV tables, PNG images)) — https://github.com/mzlab-research/SManalyst
- **Seurat** (R package providing the underlying clustering engines (including UMAP-kmeans option) used by SMAnalyst for pixel-level clustering)
- **ggplot2** (R plotting dependency used by SMAnalyst for rendering spatial visualization outputs)

## Evaluation signals

- Output table contains assignments for all 14,260 pixels across exactly 25 cluster labels (no missing/unassigned pixels)
- Cluster map is spatially coherent and major anatomical divisions (cortex, hippocampus, midbrain, hindbrain, fiber tracts) are qualitatively recognizable, consistent with the Allen Mouse Brain Atlas
- Clustering performed after noise filtering (score >= 30) yields more coherent clusters than on unfiltered ions
- Exported results include downloadable CSV cluster tables and PNG images from the Visualize & Export step

## Limitations

- Cluster count k is dataset-specific; k=25 reproduces the mouse brain Dataset 1 result but other tissues require re-tuning
- Noise filtering (score >= 30) removes ~39% of ions, so cluster maps depend on the filtering threshold choice
- Comparisons with the Allen Mouse Brain Atlas are qualitative in the reported workflow, not a quantitative concordance metric

## Evidence

- [readme] Pixel-level clustering: Identify tissue regions using 4 algorithms: Seurat-LV, Seurat-LM, Seurat-SLM, UMAP-kmeans: "Pixel-level clustering: Identify tissue regions using 4 algorithms:

    *   Seurat-LV, Seurat-LM, Seurat-SLM, UMAP-kmeans"
- [intro] Run metabolite/pixel clustering and compare regions: "**Spatial Analysis**:
    *   Run metabolite/pixel clustering
    *   Compare regions: auto-clustered or manually drawn ROIs"
- [other] noise score is defined as -log10 of the spatstat quadrat-test p-value and the default threshold is 30: [evidence span withheld — anchor status: not_found]
- [readme] Explore ion distribution images and download results: "**Visualize & Export**:
    *   Explore ion distribution images
    *   Download results (CSV tables, PNG images)"
- [other] UMAP-kmeans clustering identified 25 spatially heterogeneous categories in mouse brain tissue: [evidence span withheld — anchor status: not_found]
