---
name: umap-dimensionality-reduction
description: Use when you have a spatial metabolomics feature matrix (CSV with X/Y
  pixel coordinates plus m/z intensity columns) and need pixel-level clustering to
  identify tissue regions, e.g. k=25 clusters over a mouse brain coronal section.
when_to_use_negative:
- Input is raw IMS spectra or imzML rather than the required pre-extracted feature
  matrix CSV (X/Y columns + m/z intensity columns)
- The goal is metabolite-level (ion) clustering by spatial co-expression, not pixel-level
  tissue-region clustering
- The tissue region number is known in advance to differ substantially from k=25,
  or clusters must be smaller than pixel-scale anatomical divisions — SMAnalyst also
  offers Seurat-LV, Seurat-LM, and Seurat-SLM as alternative pixel clustering algorithms
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: SMAnalyst
  role: Web server / R Shiny platform that implements UMAP-kmeans as one of four Seurat-based
    pixel-level clustering algorithms and renders the spatial cluster map
  repo: https://metax.genomics.cn/app/SManalyst
- name: Seurat
  role: R dependency providing the Seurat-based clustering framework from which the
    UMAP-kmeans pixel clustering option derives
- name: spatstat
  role: Computes the quadrat-test p-values whose -log10 values define the noise score
    used to pre-filter ions (default threshold ≥30) before clustering
- name: R
  role: Runtime (≥ v4.1.0) for local installation of the SMAnalyst Shiny application
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.3390/biom15111562
    title: 'SMAnalyst: A Web Server for Spatial Metabolomic Data Analysis and Annotation'
  source_content_hash: sha256:d1102a97e16167c636816bf26c0c2c7997350b8049fcf319c415f9918e0c54db
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - SMAnalyst
  - Seurat
  - spatstat
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# umap-dimensionality-reduction

## Summary

UMAP-based dimensionality reduction embedded in SMAnalyst's UMAP-kmeans pixel-clustering option, which projects high-dimensional spatial metabolomics feature matrices (e.g., 14,260 pixels × 3,044 m/z ion columns) into a low-dimensional space so that k-means can delineate tissue regions, reproducing anatomical divisions such as cortex, hippocampus, midbrain, hindbrain, and fiber tracts.

## When to use

Use when you have a spatial metabolomics feature matrix (CSV with X/Y pixel coordinates plus m/z intensity columns) and need pixel-level clustering to identify tissue regions, e.g. k=25 clusters over a mouse brain coronal section.

## When NOT to use

- Input is raw IMS spectra or imzML rather than the required pre-extracted feature matrix CSV (X/Y columns + m/z intensity columns)
- The goal is metabolite-level (ion) clustering by spatial co-expression, not pixel-level tissue-region clustering
- The tissue region number is known in advance to differ substantially from k=25, or clusters must be smaller than pixel-scale anatomical divisions — SMAnalyst also offers Seurat-LV, Seurat-LM, and Seurat-SLM as alternative pixel clustering algorithms

## Inputs

- Feature matrix CSV: two pixel-coordinate columns (X, Y) followed by m/z ion-intensity columns, one row per spatial pixel (e.g., 14,260 pixels × 3,044 ions)
- Mouse brain coronal section test dataset (AFAD-ESI (+) mode, 100 μm resolution; NGDC OMIX accession OMIX009541)
- Optional: noise-filtered ion subset (noise score = -log10 spatstat quadrat-test p-value, threshold ≥30)
- Number of clusters k (25 for the reported mouse brain analysis)

## Outputs

- Per-pixel cluster assignment table (all 14,260 pixels labeled across 25 clusters)
- Spatial pixel map of cluster assignments over the tissue section
- Annotated anatomical divisions (cortex, hippocampus, midbrain, hindbrain, fiber tracts) mapped onto clusters for comparison with the Allen Mouse Brain Atlas
- Downloadable CSV tables and PNG images via the SMAnalyst Visualize & Export step

## How to apply

Load the feature matrix (columns 1-2 = pixel X/Y coordinates; remaining columns = ion intensities at each m/z value; one row per spatial pixel). Optionally restrict to ions passing the noise filter, where the noise score is -log10 of the spatstat quadrat-test p-value with default threshold score ≥30 (this filters ~39% of ions in the test dataset and improves cluster coherence). Choose UMAP-kmeans, one of the four Seurat-based pixel clustering algorithms in SMAnalyst (alongside Seurat-LV, Seurat-LM, Seurat-SLM), and set the number of clusters — k=25 was used to recover the 25 spatially heterogeneous categories in the mouse brain Dataset 1. Run UMAP to embed the pixels, then k-means on the UMAP embedding to assign cluster labels. Render the assignments as a spatial pixel map over the tissue section and compare against the Allen Mouse Brain Atlas to annotate major divisions (cerebral cortex, hippocampus, midbrain, hindbrain, fiber tracts). The rationale for UMAP here is that graph-based latent structure preserves spatial-metabolic neighborhoods so that k-means partitions correspond to anatomically coherent regions rather than noisy intensity gradients.

## Related tools

- **SMAnalyst** (Web server / R Shiny platform that implements UMAP-kmeans as one of four Seurat-based pixel-level clustering algorithms and renders the spatial cluster map) — https://metax.genomics.cn/app/SManalyst
- **Seurat** (R dependency providing the Seurat-based clustering framework from which the UMAP-kmeans pixel clustering option derives)
- **spatstat** (Computes the quadrat-test p-values whose -log10 values define the noise score used to pre-filter ions (default threshold ≥30) before clustering)
- **R** (Runtime (≥ v4.1.0) for local installation of the SMAnalyst Shiny application)

## Examples

```
R: seu <- RunUMAP(seu, dims = 1:30); cl <- kmeans(seu@reductions$umap@cell.embeddings, centers = 25)  # UMAP-kmeans pixel clustering via SMAnalyst/Seurat (k=25)
```

## Evaluation signals

- Output table contains assignments for all 14,260 pixels across exactly 25 cluster labels (k as specified)
- Cluster map shows spatially coherent, contiguous regions rather than salt-and-pepper noise; noise filtering (score ≥30, ~39% of ions removed) should improve coherence
- Major anatomical divisions — cerebral cortex, hippocampus, midbrain, hindbrain, and fiber tracts — are recognizable in the cluster map, consistent with the Allen Mouse Brain Atlas
- Input CSV conforms to the required schema: columns 1-2 are pixel coordinates, remaining columns are m/z intensities, one row per pixel
- Result should be comparable to the paper's Figure 5B-style spatial pixel map for Dataset 1

## Limitations

- The number of clusters k must be chosen a priori (k=25 in the reported mouse brain analysis); a different k may fragment or merge anatomical divisions
- Noise filtering with the default threshold ≥30 removes ~39% of ions; filtering is shown to improve cluster coherence but discards low-quadrat-score ions
- UMAP-kmeans is one of four pixel-clustering options (with Seurat-LV, Seurat-LM, Seurat-SLM); relative algorithmic trade-offs are not fully characterized in the provided context
- Performance claims are demonstrated on large-scale data (>14,000 pixels, >3000 ion peaks); behavior on much smaller or larger sections is not evidenced here

## Evidence

- [readme] Pixel-level clustering: Identify tissue regions using 4 algorithms: Seurat-LV, Seurat-LM, Seurat-SLM, UMAP-kmeans: "Pixel-level clustering: Identify tissue regions using 4 algorithms:

    *   Seurat-LV, Seurat-LM, Seurat-SLM, UMAP-kmeans"
- [abstract] Tested with a mouse brain dataset, SMAnalyst efficiently handles large-scale data: "Tested with a mouse brain dataset, SMAnalyst efficiently handles large-scale data (e.g., >14,000 pixels, >3000 ion peaks)"
- [readme] Content: 14,260 pixels × 3,044 ions: "Content: 14,260 pixels × 3,044 ions"
- [readme] SManalyst requires a feature matrix in CSV format: Columns 1-2: Pixel coordinates (X, Y): "SManalyst requires a **feature matrix** in CSV format:

*   **Columns 1-2**: Pixel coordinates (X, Y)"
- [readme] Run metabolite/pixel clustering: "4.  **Spatial Analysis**:

    *   Run metabolite/pixel clustering"
