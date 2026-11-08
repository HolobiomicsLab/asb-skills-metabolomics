---
name: wilcoxon-rank-sum-differential-testing
description: Use when pixels from an MSI feature matrix (e.g., 14,260 pixels × 3,044
  ion peaks) have been assigned to biological regions via UMAP-kmeans clustering and
  you need a non-parametric per-ion differential test between two region groups (e.g.,
  MB clusters 6/1 vs HIP clusters 23/12).
when_to_use_negative:
- When regions have not been defined — if no UMAP-kmeans or manual ROI assignment
  exists yet, run SMAnalyst's pixel clustering or ROI drawing first rather than testing
  on arbitrary groupings
- When the comparison requires parametric modeling of covariates or replicate-level
  design rather than pixel-level univariate ranking
- When input is raw IMS mass-spec data rather than an already-picked, QC-filtered
  feature matrix of ion intensities per pixel
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0091
tools:
- name: SMAnalyst
  role: Web platform that hosts the feature matrix (Tutorial Panel example data),
    provides UMAP-kmeans pixel clustering to define the MB/HIP regions, and performs
    differential metabolite detection with fold-change/significance filters
  repo: https://github.com/mzlab-research/SManalyst
- name: Seurat
  role: Provides the FindMarkers function used to run the univariate Wilcoxon rank-sum
    test on every metabolic ion between the MB and HIP pixel groups
- name: R (>= v4.1.0)
  role: Runtime environment for local installation of SManalyst and execution of the
    FindMarkers-based differential test
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.3390/biom15111562
    title: 'SMAnalyst: A Web Server for Spatial Metabolomic Data Analysis and Annotation'
  source_content_hash: sha256:4acfb10239751938024215fd70b5b36b05a5a62fae9800b2a1483dc97037d52a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mzlab-research/SManalyst
  tools:
  - R (>= v4.1.0)
  - R (>=)
  - SMAnalyst
  - Seurat
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8461538461538461
n_claims: 13
---

# wilcoxon-rank-sum-differential-testing

## Summary

Univariate Wilcoxon rank-sum differential testing between pixel clusters defined on spatial metabolomics (MSI) data, used in SMAnalyst's workflow to identify metabolites significantly upregulated in midbrain (MB) vs hippocampal (HIP) regions of a mouse brain DESI section, followed by Benjamini–Hochberg FDR correction and fold-change filtering.

## When to use

Use when pixels from an MSI feature matrix (e.g., 14,260 pixels × 3,044 ion peaks) have been assigned to biological regions via UMAP-kmeans clustering and you need a non-parametric per-ion differential test between two region groups (e.g., MB clusters 6/1 vs HIP clusters 23/12).

## When NOT to use

- When regions have not been defined — if no UMAP-kmeans or manual ROI assignment exists yet, run SMAnalyst's pixel clustering or ROI drawing first rather than testing on arbitrary groupings
- When the comparison requires parametric modeling of covariates or replicate-level design rather than pixel-level univariate ranking
- When input is raw IMS mass-spec data rather than an already-picked, QC-filtered feature matrix of ion intensities per pixel

## Inputs

- SMAnalyst feature matrix CSV (columns 1-2: pixel X,Y coordinates; remaining columns: ion intensities per m/z; one row per spatial pixel)
- UMAP-kmeans pixel cluster assignments (25 clusters) from SMAnalyst's spatial analysis module
- Pixel-to-group mapping: MB = clusters 6 and 1, HIP = clusters 23 and 12

## Outputs

- differential_metabolites_MB_vs_HIP.csv — filtered differential metabolites table with average fold change, significance (BH-adjusted p-value), and annotation status per ion
- Per-region upregulated metabolite lists: 76 metabolites up in MB (27 annotated, 17 uniquely annotated) and 82 up in HIP (29 annotated, 16 uniquely annotated)

## How to apply

Load the SMAnalyst feature matrix (14,260 pixels × 3,044 ion peaks, from the Tutorial Panel example data or NGDC OMIX OMIX009541) together with the 25-cluster UMAP-kmeans pixel assignments, and map clusters 6 and 1 to the midbrain (MB) group and clusters 23 and 12 to the hippocampal (HIP) group, treating every pixel as an individual sample. Run a univariate Wilcoxon rank-sum test on each ion between MB and HIP using Seurat's FindMarkers function (the non-parametric default of SMAnalyst's differential analysis module, chosen because ion intensities are not normally distributed). Adjust p-values with the Benjamini–Hochberg method to control FDR, then retain metabolites with Fold Change > 2 or < 0.5 and FDR-adjusted p-value < 0.01. Tabulate average expression fold change, significance, and annotation status per ion and export the filtered table as differential_metabolites_MB_vs_HIP.csv; downregulation in one group is equivalently upregulation in the other, so both regional lists come from the single MB-vs-HIP comparison.

## Related tools

- **SMAnalyst** (Web platform that hosts the feature matrix (Tutorial Panel example data), provides UMAP-kmeans pixel clustering to define the MB/HIP regions, and performs differential metabolite detection with fold-change/significance filters) — https://github.com/mzlab-research/SManalyst
- **Seurat** (Provides the FindMarkers function used to run the univariate Wilcoxon rank-sum test on every metabolic ion between the MB and HIP pixel groups)
- **R (>= v4.1.0)** (Runtime environment for local installation of SManalyst and execution of the FindMarkers-based differential test)

## Examples

```
markers <- FindMarkers(seurat_obj, ident.1 = c("6", "1"), ident.2 = c("23", "12"), test.use = "wilcox", logfc.threshold = 1)
```

## Evaluation signals

- The filtered differential table contains exactly 76 significant metabolites upregulated in MB and 82 in HIP under FC > 2 or < 0.5 and FDR < 0.01, matching Section 3.4 (Figure 5D)
- Annotated-subset counts reproduce: 27 annotated / 17 uniquely annotated for MB and 29 annotated / 16 uniquely annotated for HIP
- Output file differential_metabolites_MB_vs_HIP.csv exists with columns for average fold change, significance, and annotation status per ion
- All reported p-values are BH-adjusted (FDR-controlled), and no ion passes filtering without meeting both the fold-change and FDR < 0.01 criteria
- Every pixel in the feature matrix is assigned to exactly one of the two groups (MB = clusters 6/1, HIP = clusters 23/12) before testing

## Limitations

- Treating each of the >14,000 pixels as an independent sample (pseudo-replication) means significance reflects spatial-region pixel differences, not biological replicate statistics
- Results depend on the upstream 25-cluster UMAP-kmeans assignment; the MB/HIP region definition (clusters 6/1 vs 23/12) is dataset-specific to the mouse brain coronal section
- SMAnalyst is an R/Shiny app requiring R >= v4.1.0 and many CRAN/Bioc packages (including Seurat), so local installation can be heavy; the online web app is the lighter alternative

## Evidence

- [readme] Differential metabolite detection with fold-change/significance filters: "Differential metabolite detection with fold-change/significance filters"
- [abstract] It also offers flexible differential analysis (cluster- or user-defined regions).: "It also offers flexible differential analysis (cluster- or user-defined regions)."
- [readme] Run metabolite/pixel clustering; Compare regions: auto-clustered or manually drawn ROIs: "Run metabolite/pixel clustering
    *   Compare regions: auto-clustered or manually drawn ROIs"
- [readme] Seurat is listed as a required package for SManalyst local installation: ""MetaboCoreUtils", "Seurat""
- [intro] Quality control precedes annotation and differential analysis in the SMAnalyst workflow: "It integrates quality control, metabolite annotation, spatial pattern discovery, and differential analysis into a unified workflow."
