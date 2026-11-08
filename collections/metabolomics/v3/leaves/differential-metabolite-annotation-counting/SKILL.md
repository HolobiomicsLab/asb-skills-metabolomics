---
name: differential-metabolite-annotation-counting
description: Use when a spatial metabolomics feature matrix (pixels × m/z ion peaks)
  has been pixel-clustered (e.g., UMAP-kmeans) and you must identify metabolites significantly
  upregulated in one anatomical region versus another, with per-region annotation
  counts.
when_to_use_negative:
- Input is not a spatial pixel-level feature matrix (e.g., bulk LC-MS peak tables)
  or lacks pixel-cluster assignments needed to define the two compared regions
- Region boundaries are already known and manually drawn as ROIs rather than derived
  from UMAP-kmeans pixel clustering, where SMAnalyst's manual ROI differential mode
  may be simpler
- The dataset is small (< thousands of pixels) such that a simple t-test or limma
  on ROIs suffices without the Seurat/SMAnalyst pipeline
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3334
tools:
- name: SMAnalyst
  role: Web platform providing the differential analysis module (cluster- or user-defined
    ROIs) with fold-change/significance filters and metabolite annotation for tabulating
    annotated counts
  repo: https://github.com/mzlab-research/SManalyst
- name: Seurat
  role: Provides the FindMarkers function used to run the univariate Wilcoxon rank-sum
    test on every metabolic ion between the MB and HIP pixel groups
- name: R
  role: Runtime environment (>= v4.1.0) for local execution of the SMAnalyst/Seurat
    differential workflow
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.3390/biom15111562
    title: 'SMAnalyst: A Web Server for Spatial Metabolomic Data Analysis and Annotation'
  source_content_hash: sha256:a7e6a5c907e2f55d22ea8051f3bf754e6b35a64799cfd17f947a2663d9ad1d85
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mzlab-research/SManalyst
  tools:
  - R
  - R (>=)
  - SMAnalyst
  - Seurat
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3334
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# differential-metabolite-annotation-counting

## Summary

Performs region-wise differential metabolite testing on spatial metabolomics pixel data (e.g., midbrain clusters 6/1 vs hippocampal clusters 23/12 from a 14,260-pixel × 3,044-ion mouse brain feature matrix) using Seurat's Wilcoxon rank-sum FindMarkers with FC > 2 or < 0.5 and FDR < 0.01 filters, then tabulates annotated metabolite counts per region.

## When to use

Use when a spatial metabolomics feature matrix (pixels × m/z ion peaks) has been pixel-clustered (e.g., UMAP-kmeans) and you must identify metabolites significantly upregulated in one anatomical region versus another, with per-region annotation counts.

## When NOT to use

- Input is not a spatial pixel-level feature matrix (e.g., bulk LC-MS peak tables) or lacks pixel-cluster assignments needed to define the two compared regions
- Region boundaries are already known and manually drawn as ROIs rather than derived from UMAP-kmeans pixel clustering, where SMAnalyst's manual ROI differential mode may be simpler
- The dataset is small (< thousands of pixels) such that a simple t-test or limma on ROIs suffices without the Seurat/SMAnalyst pipeline

## Inputs

- Feature matrix CSV: columns 1-2 pixel coordinates (X, Y), subsequent columns ion intensities per m/z; 14,260 pixels × 3,044 ion peaks (mouse brain coronal section, AFAD-ESI (+), 100 μm resolution)
- UMAP-kmeans pixel cluster assignments (25 clusters) from SMAnalyst spatial analysis
- Metabolite annotation table (HMDB/KEGG/LIPIDMAPS or custom database) for annotation-status counting

## Outputs

- differential_metabolites_MB_vs_HIP.csv with average fold change, significance, and annotation status per ion (76 MB-upregulated, 82 HIP-upregulated entries)
- Per-region annotated metabolite counts (MB: 27 annotated / 17 uniquely annotated; HIP: 29 annotated / 16 uniquely annotated) feeding pathway enrichment

## How to apply

Load the feature matrix (e.g., Dataset 1: 14,260 pixels × 3,044 ion peaks from NGDC OMIX OMIX009541 or the SMAnalyst Tutorial Panel) together with UMAP-kmeans pixel cluster assignments (25 clusters). Define biological groups by mapping clusters to regions — midbrain (MB) = clusters 6 and 1, hippocampal (HIP) = clusters 23 and 12 — treating all pixels of a group as that group's samples. Run the univariate Wilcoxon rank-sum test for every metabolic ion between MB and HIP using Seurat's FindMarkers function (exposed through SMAnalyst's flexible differential analysis module). Adjust p-values with Benjamini–Hochberg FDR, then retain metabolites with Fold Change > 2 or < 0.5 and FDR-adjusted p-value < 0.01. Tabulate average fold change, significance, and annotation status per ion, and verify the reported counts: 76 metabolites upregulated in MB (27 annotated, 17 uniquely annotated) and 82 in HIP (29 annotated, 16 uniquely annotated), whose pathway enrichments align with regional functions (taurine/sphingolipid/glycerophospholipid in HIP; glycerophospholipid/linoleic acid/pyruvate in MB).

## Related tools

- **SMAnalyst** (Web platform providing the differential analysis module (cluster- or user-defined ROIs) with fold-change/significance filters and metabolite annotation for tabulating annotated counts) — https://github.com/mzlab-research/SManalyst
- **Seurat** (Provides the FindMarkers function used to run the univariate Wilcoxon rank-sum test on every metabolic ion between the MB and HIP pixel groups)
- **R** (Runtime environment (>= v4.1.0) for local execution of the SMAnalyst/Seurat differential workflow)

## Examples

```
Idents(obj) <- "cluster"; markers <- FindMarkers(obj, ident.1 = c(6,1), ident.2 = c(23,12), test.use = "wilcox", logfc.threshold = log(2))
```

## Evaluation signals

- Differential table contains exactly 76 significant metabolites upregulated in MB and 82 in HIP under FC > 2 or < 0.5 and FDR < 0.01, matching Section 3.4 (Figure 5D)
- MB annotated counts are 27 (17 uniquely annotated) and HIP annotated counts are 29 (16 uniquely annotated)
- HIP metabolites include taurine and sphingolipids whose enrichments map to taurine/sphingolipid/glycerophospholipid pathways; MB to glycerophospholipid/linoleic acid/pyruvate pathways
- All p-values in the output table are BH-adjusted (FDR-controlled) rather than raw
- Every ion in the feature matrix (3,044 peaks) was tested, not a pre-filtered subset

## Limitations

- Cluster-to-region mapping is dataset-specific: the MB (clusters 6/1) and HIP (clusters 23/12) assignments come from the 25-cluster UMAP-kmeans solution on this particular mouse brain section and must be re-derived for other tissues or cluster resolutions
- SMAnalyst's noise-ion filtering (default score ≥ 30) and QC steps (QC1–QC4) should precede differential testing, so counts depend on upstream QC thresholds
- Differential results depend on annotation coverage; only 27/76 MB and 29/82 HIP hits were annotated against the chosen database

## Evidence

- [readme] Flexible differential analysis with cluster- or user-defined ROI differential testing and fold-change/significance filters: "Differential metabolite detection with fold-change/significance filters"
- [intro] SMAnalyst's flexible differential analysis accepts cluster-defined regions: "It also offers flexible differential analysis (cluster- or user-defined regions)."
- [abstract] Scale of the tested mouse brain dataset: "Tested with a mouse brain dataset, SMAnalyst efficiently handles large-scale data (e.g., >14,000 pixels, >3000 ion peaks)"
- [readme] Input data specification for the feature matrix: "SManalyst requires a feature matrix in CSV format: Columns 1-2: Pixel coordinates (X, Y)"
- [readme] Reference dataset access route: "Built-in dataset in SManalyst (Tutorial Panel → Example Data) Raw data: NGDC OMIX Repository ID: OMIX009541"
