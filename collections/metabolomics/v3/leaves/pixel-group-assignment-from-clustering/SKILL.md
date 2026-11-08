---
name: pixel-group-assignment-from-clustering
description: Use when a spatial metabolomics dataset (e.g., ~14,260 pixels x ~3,044
  ion peaks) has been pixel-clustered (UMAP-kmeans or Seurat-LV/LM/SLM) and cluster
  IDs must be merged into two biological groups for comparison.
when_to_use_negative:
- When the user already has manually drawn ROIs or externally defined biological regions
  — use manual ROI selection instead of cluster-derived groups
- When the input is raw mass-spectrometry imaging data not yet converted to a pixel-by-ion
  feature matrix — quality control and preprocessing must come first
- When comparing more than two regions or treating clusters individually rather than
  merging them into biological groups
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: SMAnalyst
  role: Web platform used to run pixel-level clustering (UMAP-kmeans) on the feature
    matrix and generate the cluster assignments that are merged into biological pixel
    groups
  repo: mzlab-research/SManalyst
- name: Seurat
  role: Provides the FindMarkers function used to run the univariate Wilcoxon rank-sum
    test on every metabolic ion between the pixel groups
- name: R
  role: Runtime environment (>= v4.1.0) for local installation of SMAnalyst and execution
    of the group assignment and differential analysis
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.3390/biom15111562
    title: 'SMAnalyst: A Web Server for Spatial Metabolomic Data Analysis and Annotation'
  source_content_hash: sha256:5b4c6dc938f0cd6d535054f9f2f7b0d3a15e78254ca62e66294faea653de9510
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mzlab-research/SManalyst
  tools:
  - R
  - R (>=)
  - SMAnalyst
  - Seurat
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# Pixel group assignment from clustering

## Summary

Maps pixel-level clustering results from spatial metabolomics feature matrices onto biologically defined regions (e.g., midbrain vs hippocampal pixel sets) so that every pixel in a group is treated as a sample for downstream differential metabolite testing.

## When to use

Use when a spatial metabolomics dataset (e.g., ~14,260 pixels x ~3,044 ion peaks) has been pixel-clustered (UMAP-kmeans or Seurat-LV/LM/SLM) and cluster IDs must be merged into two biological groups for comparison.

## When NOT to use

- When the user already has manually drawn ROIs or externally defined biological regions — use manual ROI selection instead of cluster-derived groups
- When the input is raw mass-spectrometry imaging data not yet converted to a pixel-by-ion feature matrix — quality control and preprocessing must come first
- When comparing more than two regions or treating clusters individually rather than merging them into biological groups

## Inputs

- Feature matrix CSV with pixel coordinates (X, Y) in columns 1-2 and ion intensities per m/z (e.g., 14,260 pixels x 3,044 ion peaks, NGDC OMIX ID OMIX009541)
- Pixel cluster assignments from UMAP-kmeans (25 clusters) or other SMAnalyst pixel-clustering algorithms (Seurat-LV, Seurat-LM, Seurat-SLM)
- Mapping of cluster IDs to biological group labels (e.g., clusters 6/1 -> MB; clusters 23/12 -> HIP)

## Outputs

- Two-group pixel assignment table labeling each spatial pixel as MB or HIP
- Differential metabolites table (differential_metabolites_MB_vs_HIP.csv) with average fold change, significance, and annotation status per ion

## How to apply

Load the feature matrix (pixel coordinates x m/z ion intensities, e.g., 14,260 pixels x 3,044 ion peaks) together with the pixel cluster assignments from the chosen clustering algorithm (here UMAP-kmeans with 25 clusters). Merge designated cluster IDs into named biological groups — e.g., midbrain (MB) = clusters 6 and 1, hippocampal (HIP) = clusters 23 and 12 — treating all pixels of a group as that group's samples. Use SMAnalyst's spatial analysis module to run metabolite/pixel clustering and define regions, either auto-clustered or manually drawn ROIs, then feed the two pixel groups into differential analysis (Wilcoxon rank-sum via Seurat's FindMarkers with Benjamini-Hochberg FDR adjustment). The rationale is that pixel clustering captures metabolic heterogeneity across the tissue section, and grouping clusters by anatomical identity converts a 2D pixel grid into the two-sample contrast needed for region-level statistics. Retain metabolites with Fold Change > 2 or < 0.5 and FDR-adjusted p-value < 0.01 before tabulating fold change, significance, and annotation status per ion.

## Related tools

- **SMAnalyst** (Web platform used to run pixel-level clustering (UMAP-kmeans) on the feature matrix and generate the cluster assignments that are merged into biological pixel groups) — mzlab-research/SManalyst
- **Seurat** (Provides the FindMarkers function used to run the univariate Wilcoxon rank-sum test on every metabolic ion between the pixel groups)
- **R** (Runtime environment (>= v4.1.0) for local installation of SMAnalyst and execution of the group assignment and differential analysis)

## Examples

```
markers <- Seurat::FindMarkers(seurat_obj, ident.1 = c(6, 1), ident.2 = c(23, 12), test.use = "wilcox")
```

## Evaluation signals

- The differential table contains exactly 76 metabolites upregulated in MB and 82 in HIP under FC > 2 or < 0.5 and FDR < 0.01 filters
- Annotated subset counts match: 27 annotated (17 uniquely annotated) among MB metabolites and 29 annotated (16 uniquely annotated) among HIP metabolites
- Every pixel in the feature matrix receives exactly one group label, and group sizes correspond to the union of the assigned cluster memberships
- Output file differential_metabolites_MB_vs_HIP.csv is produced with columns for average fold change, significance, and annotation status

## Limitations

- Cluster-to-region mapping (e.g., clusters 6/1 = MB) must be validated against anatomical context; incorrect cluster-to-region assignment propagates directly into the differential results
- The reported counts (76 MB / 82 HIP) are specific to the 14,260-pixel mouse brain dataset under the stated filters; other datasets or thresholds will yield different counts
- Noise ion filtering with default threshold score >= 30 should precede differential analysis, or low-quality ions may inflate the significant metabolite list

## Evidence

- [readme] Pixel-level clustering: Identify tissue regions using 4 algorithms: Seurat-LV, Seurat-LM, Seurat-SLM, UMAP-kmeans: "Pixel-level clustering: Identify tissue regions using 4 algorithms:

*   Seurat-LV, Seurat-LM, Seurat-SLM, UMAP-kmeans"
- [readme] Differential metabolite detection with fold-change/significance filters: "Differential metabolite detection with fold-change/significance filters"
- [intro] Compare regions: auto-clustered or manually drawn ROIs: "**Spatial Analysis**:
    *   Run metabolite/pixel clustering
    *   Compare regions: auto-clustered or manually drawn ROIs"
- [abstract] It also offers flexible differential analysis (cluster- or user-defined regions).: "It also offers flexible differential analysis (cluster- or user-defined regions)."
