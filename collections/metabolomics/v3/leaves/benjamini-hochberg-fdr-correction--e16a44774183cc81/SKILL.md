---
name: benjamini-hochberg-fdr-correction
description: Use when a spatial metabolomics feature matrix (e.g., 14,260 pixels x
  3,044 ion peaks) has been grouped into two regions (e.g., pixel clusters MB vs HIP)
  and every ion's raw p-value from a univariate test must be multiplicity-corrected
  before calling differential metabolites.
when_to_use_negative:
- The input is not a two-group spatial metabolomics contrast (e.g., no cluster- or
  ROI-defined regions to compare), so there is no per-ion p-value family to correct
- Only a small pre-selected list of metabolites is being tested individually, where
  BH correction across thousands of ion peaks is not the intended multiplicity framework
- The analysis goal is annotation or QC (noise-ion filtering, metabolite identification
  scoring) rather than differential metabolite detection
edam_operation: http://edamontology.org/operation_3660
edam_topics:
- http://edamontology.org/topic_3517
tools:
- name: Seurat
  role: FindMarkers runs the Wilcoxon rank-sum test per ion and applies the Benjamini–Hochberg
    p-value adjustment on the MB vs HIP comparison
- name: SMAnalyst
  role: Provides the feature matrix, UMAP-kmeans pixel clusters, and the differential-analysis
    module with fold-change/significance filters that the corrected p-values feed
    into
  repo: https://github.com/mzlab-research/SManalyst
- name: qvalue
  role: R package installed alongside SMAnalyst for p-value/FDR-type estimation in
    the local deployment
  repo: https://github.com/mzlab-research/SManalyst
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.3390/biom15111562
    title: 'SMAnalyst: A Web Server for Spatial Metabolomic Data Analysis and Annotation'
  source_content_hash: sha256:134dd181026cd345cbbad604e22f698c165438dcd0b0428cc59759a1ea5cb151
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mzlab-research/SManalyst
  tools:
  - R (>=)
  - SMAnalyst
  - Seurat
  - qvalue
  edam_operation: http://edamontology.org/operation_3660
  edam_topics:
  - http://edamontology.org/topic_3517
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7857142857142857
n_claims: 14
---

# benjamini-hochberg-fdr-correction

## Summary

Applies Benjamini–Hochberg false-discovery-rate correction to p-values from a per-ion univariate test (Wilcoxon rank-sum via Seurat's FindMarkers) across thousands of spatial-metabolomics ion peaks, then filters differential metabolites by FC > 2 or < 0.5 with FDR < 0.01.

## When to use

Use when a spatial metabolomics feature matrix (e.g., 14,260 pixels x 3,044 ion peaks) has been grouped into two regions (e.g., pixel clusters MB vs HIP) and every ion's raw p-value from a univariate test must be multiplicity-corrected before calling differential metabolites.

## When NOT to use

- The input is not a two-group spatial metabolomics contrast (e.g., no cluster- or ROI-defined regions to compare), so there is no per-ion p-value family to correct
- Only a small pre-selected list of metabolites is being tested individually, where BH correction across thousands of ion peaks is not the intended multiplicity framework
- The analysis goal is annotation or QC (noise-ion filtering, metabolite identification scoring) rather than differential metabolite detection

## Inputs

- Feature matrix CSV (pixel coordinates X,Y in columns 1-2; ion intensities per m/z column; e.g., 14,260 pixels x 3,044 ion peaks from AFAD-ESI (+) mode, 100um resolution)
- Pixel cluster assignments (UMAP-kmeans, 25 clusters) mapping each pixel to a region group
- Raw per-ion p-values from Seurat FindMarkers (test.use = 'wilcox') comparing MB vs HIP pixels

## Outputs

- differential_metabolites_MB_vs_HIP.csv with per-ion average fold change, raw p-value, BH-adjusted FDR, and annotation status
- Filtered differential metabolite table (76 MB-upregulated with 27 annotated/17 uniquely annotated; 82 HIP-upregulated with 29 annotated/16 uniquely annotated) feeding pathway enrichment

## How to apply

Run the univariate Wilcoxon rank-sum test for every metabolic ion between the two pixel-defined groups (e.g., MB = clusters 6/1 vs HIP = clusters 23/12) using Seurat's FindMarkers function, which returns raw p-values per ion. Adjust all p-values with the Benjamini–Hochberg method to control FDR — BH is chosen because thousands of ion peaks are tested simultaneously and per-ion p-values alone would yield many false positives. Then retain metabolites with Fold Change > 2 or < 0.5 AND FDR-adjusted p-value < 0.01. Tabulate average fold change, significance, and annotation status per ion, and verify the filtered table reproduces the reference counts (76 upregulated in MB, 82 in HIP for the mouse brain OMIX009541 dataset). Save the result as differential_metabolites_MB_vs_HIP.csv for downstream pathway enrichment.

## Related tools

- **Seurat** (FindMarkers runs the Wilcoxon rank-sum test per ion and applies the Benjamini–Hochberg p-value adjustment on the MB vs HIP comparison)
- **SMAnalyst** (Provides the feature matrix, UMAP-kmeans pixel clusters, and the differential-analysis module with fold-change/significance filters that the corrected p-values feed into) — https://github.com/mzlab-research/SManalyst
- **qvalue** (R package installed alongside SMAnalyst for p-value/FDR-type estimation in the local deployment) — https://github.com/mzlab-research/SManalyst

## Examples

```
FindMarkers(object = mb_hip_seurat_obj, ident.1 = "MB", ident.2 = "HIP", test.use = "wilcox", logfc.threshold = 1)
```

## Evaluation signals

- The differential table contains exactly 76 significant metabolites upregulated in MB and 82 in HIP under FC > 2 or < 0.5 and FDR < 0.01, matching Section 3.4 (Figure 5D)
- Every retained ion has FDR-adjusted p-value < 0.01 and fold change either > 2 or < 0.5 — no ion violating either threshold appears in the filtered table
- The adjusted p-value column is monotone-consistent with raw p-values and never smaller than them (BH adjustment property)
- Counts of annotated rows are reproducible: 27 annotated (17 uniquely annotated) among the 76 MB metabolites and 29 annotated (16 uniquely annotated) among the 82 HIP metabolites
- The output CSV is saved under the expected name differential_metabolites_MB_vs_HIP.csv and downstream pathway enrichment reproduces the reported HIP (taurine/sphingolipid/glycerophospholipid) and MB (glycerophospholipid/linoleic acid/pyruvate) enrichments

## Limitations

- The thresholds (FC > 2 or < 0.5, FDR < 0.01) and resulting counts (76/82) were tuned to the mouse brain OMIX009541 dataset; other tissues or clustering resolutions will yield different counts
- The group definitions depend on the UMAP-kmeans pixel clustering (MB = clusters 6/1, HIP = clusters 23/12); different clustering algorithms or cluster choices change the tested contrast
- Most significant ions remain unannotated (e.g., only 27/76 MB and 29/82 HIP metabolites annotated), so corrected significance alone does not guarantee interpretable pathway hits

## Evidence

- [full_text] BH thresholds and filtering: [evidence span withheld — anchor status: not_found]
- [full_text] test that generates p-value family: [evidence span withheld — anchor status: not_found]
- [full_text] validation counts: [evidence span withheld — anchor status: not_found]
- [readme] differential module scope: "Differential metabolite detection with fold-change/significance filters"
