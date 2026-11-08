---
name: fold-change-significance-filtering
description: Use when two spatially defined pixel groups (e.g., UMAP-kmeans clusters
  6/1 vs 23/12) must be compared per ion peak in a 14,260-pixel × 3,044-ion spatial
  metabolomics feature matrix, and you need fold-change/significance-filtered differential
  metabolites to feed pathway enrichment.
when_to_use_negative:
- Input is raw mass-spectrometry imaging data rather than a pre-built pixel × ion
  feature matrix — run SMAnalyst QC (background consistency QC1, noise ion filtering
  QC2 with score ≥ 30) and annotation first
- Regions are not defined by pixel clusters but by manually drawn ROIs of different
  biological origin where a Wilcoxon rank-sum on pixel-level samples is inappropriate
- The goal is peak picking or marker-ion discovery from latent features rather than
  two-group differential testing — SmartGate's graph-attention autoencoder covers
  that case instead
edam_operation: http://edamontology.org/operation_3662
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: SMAnalyst
  role: Web platform providing the feature matrix, UMAP-kmeans pixel clustering used
    to define the compared regions, and export of the filtered differential table
  repo: https://github.com/mzlab-research/SManalyst
- name: Seurat
  role: FindMarkers function executes the per-ion Wilcoxon rank-sum test between the
    MB and HIP pixel groups
- name: R
  role: Runtime environment (≥ v4.1.0) hosting Seurat and the differential filtering
    script
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.3390/biom15111562
    title: 'SMAnalyst: A Web Server for Spatial Metabolomic Data Analysis and Annotation'
  source_content_hash: sha256:c62b02b72f39b7475a45e65cb7465e088afe7c8e45fdfcabb5d363ef070244ad
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mzlab-research/SManalyst
  tools:
  - R
  - R (>=)
  - SMAnalyst
  - Seurat
  edam_operation: http://edamontology.org/operation_3662
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8461538461538461
n_claims: 13
---

# fold-change-significance-filtering

## Summary

A statistical filtering skill that identifies metabolite ions differentially abundant between two spatial regions by combining a non-parametric Wilcoxon rank-sum test with Benjamini–Hochberg FDR control and a dual fold-change cutoff (FC > 2 or < 0.5). In the SMAnalyst mouse-brain workflow it separates midbrain (MB) from hippocampal (HIP) metabolites for downstream pathway enrichment.

## When to use

Use when two spatially defined pixel groups (e.g., UMAP-kmeans clusters 6/1 vs 23/12) must be compared per ion peak in a 14,260-pixel × 3,044-ion spatial metabolomics feature matrix, and you need fold-change/significance-filtered differential metabolites to feed pathway enrichment.

## When NOT to use

- Input is raw mass-spectrometry imaging data rather than a pre-built pixel × ion feature matrix — run SMAnalyst QC (background consistency QC1, noise ion filtering QC2 with score ≥ 30) and annotation first
- Regions are not defined by pixel clusters but by manually drawn ROIs of different biological origin where a Wilcoxon rank-sum on pixel-level samples is inappropriate
- The goal is peak picking or marker-ion discovery from latent features rather than two-group differential testing — SmartGate's graph-attention autoencoder covers that case instead

## Inputs

- Feature matrix CSV (pixel X,Y coordinates in columns 1-2; ion intensities per m/z column; 14,260 pixels × 3,044 ion peaks for the SMAnalyst mouse brain dataset, OMIX009541)
- Pixel-cluster assignment table mapping each pixel to one of 25 UMAP-kmeans clusters
- Group definition mapping cluster IDs to biological regions (MB: clusters 6/1; HIP: clusters 23/12)

## Outputs

- Differential metabolites table (e.g., differential_metabolites_MB_vs_HIP.csv) with per-ion fold change, significance, and annotation status
- Counts of significant metabolites per region (76 upregulated in MB, 82 in HIP; 27 annotated/17 uniquely annotated for MB, 29 annotated/16 uniquely annotated for HIP)

## How to apply

First assign every pixel of the feature matrix to one of two biological groups using the pixel-cluster assignments (in the SMAnalyst mouse-brain case: midbrain = clusters 6 and 1, hippocampal = clusters 23 and 12). For each of the 3,044 ion peaks, run a univariate Wilcoxon rank-sum test between the groups with Seurat's FindMarkers function, treating all pixels of a group as that group's samples. Adjust raw p-values with the Benjamini–Hochberg method to control the false discovery rate, then retain ions satisfying Fold Change > 2 or < 0.5 AND FDR-adjusted p-value < 0.01. Tabulate average expression fold change, significance, and annotation status per ion (annotated and uniquely annotated counts), then export the table (e.g., differential_metabolites_MB_vs_HIP.csv) for pathway enrichment. The dual cutoff on fold change plus the 0.01 FDR threshold is the decision point: ions passing only one criterion are dropped.

## Related tools

- **SMAnalyst** (Web platform providing the feature matrix, UMAP-kmeans pixel clustering used to define the compared regions, and export of the filtered differential table) — https://github.com/mzlab-research/SManalyst
- **Seurat** (FindMarkers function executes the per-ion Wilcoxon rank-sum test between the MB and HIP pixel groups)
- **R** (Runtime environment (≥ v4.1.0) hosting Seurat and the differential filtering script)

## Examples

```
FindMarkers(seurat_obj, ident.1 = c(6,1), ident.2 = c(23,12), test.use = "wilcox")
```

## Evaluation signals

- Output table contains exactly 76 metabolites upregulated in MB and 82 in HIP under FC > 2 or < 0.5 and FDR < 0.01, matching Section 3.4 (Figure 5D)
- Annotated subset counts match: 27 annotated / 17 uniquely annotated for MB; 29 annotated / 16 uniquely annotated for HIP
- Every retained ion satisfies both criteria simultaneously (fold-change cutoff AND Benjamini–Hochberg-adjusted p < 0.01)
- FDR adjustment used BH method; test used is Wilcoxon rank-sum (Seurat FindMarkers), not a parametric t-test
- Output is a valid CSV with average expression fold change, significance, and annotation status per ion

## Limitations

- Region definition depends entirely on the chosen UMAP-kmeans clusters (6/1 vs 23/12 out of 25); different cluster assignments change which pixels enter each group and thus the differential counts
- Only ions passing the strict dual filter (FC > 2 or < 0.5, FDR < 0.01) are carried forward, so borderline metabolites are discarded before pathway enrichment
- SMAnalyst's differential analysis offers cluster-defined or user-defined regions; results are specific to the selected ROI strategy and the 100 μm-resolution AFAD-ESI (+) mouse brain test dataset

## Evidence

- [readme] Differential metabolite detection with fold-change/significance filters: "Differential metabolite detection with fold-change/significance filters"
- [intro] Flexible differential analysis scope: "It also offers flexible differential analysis (cluster- or user-defined regions)."
- [abstract] Full workflow placement of differential analysis: "covering the full analytical workflow (quality control, preprocessing, identification, pattern, and differential analysis)"
