---
name: wilcoxon-rank-sum-testing-with-fdr-correction
description: Use when comparing ion intensities between two spatial metabolomics regions
  (auto-clustered or manually drawn ROIs) on a feature matrix, and metabolite intensity
  distributions cannot be assumed normal.
when_to_use_negative:
- Input is raw imaging MS data (e.g., imzML) rather than a processed pixel-by-m/z
  feature matrix — peak picking and QC must come first
- Comparison requires more than two regions or repeated-measures design — Wilcoxon
  rank-sum is a two-group test
- Pixels are spatially autocorrelated and must be treated as independent samples —
  the benchmark treats all pixels in a group as samples, which can inflate significance
edam_operation: http://edamontology.org/operation_3661
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3170
tools:
- name: SMAnalyst
  role: Web platform providing flexible differential analysis with fold-change/significance
    filters; supplies the example feature matrix and reference differential results
  repo: https://github.com/mzlab-research/SManalyst
- name: R (>= v4.1.0)
  role: Runtime for running wilcox.test and p.adjust (method='BH') on pixel-grouped
    ion intensities
- name: Cardinal
  role: Used in the benchmark to define the comparison regions (spatial shrunken centroids,
    25 clusters) whose pixels feed the Wilcoxon test
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.3390/biom15111562
    title: 'SMAnalyst: A Web Server for Spatial Metabolomic Data Analysis and Annotation'
  source_content_hash: sha256:81e00d84100119125e299422c1a038402adc5e76e9e189b9c92f6822f964a328
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mzlab-research/SManalyst
  tools:
  - Cardinal
  - R (>= v4.1.0)
  - SMAnalyst
  edam_operation: http://edamontology.org/operation_3661
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8461538461538461
n_claims: 13
---

# Wilcoxon rank-sum testing with FDR correction

## Summary

A non-parametric differential analysis for spatial metabolomics pixels: run the Wilcoxon rank-sum test on ion intensities between two regions of interest (e.g., midbrain vs hippocampal pixel groups), then correct p-values with Benjamini-Hochberg FDR and filter by fold-change/significance.

## When to use

Use when comparing ion intensities between two spatial metabolomics regions (auto-clustered or manually drawn ROIs) on a feature matrix, and metabolite intensity distributions cannot be assumed normal.

## When NOT to use

- Input is raw imaging MS data (e.g., imzML) rather than a processed pixel-by-m/z feature matrix — peak picking and QC must come first
- Comparison requires more than two regions or repeated-measures design — Wilcoxon rank-sum is a two-group test
- Pixels are spatially autocorrelated and must be treated as independent samples — the benchmark treats all pixels in a group as samples, which can inflate significance

## Inputs

- Feature matrix CSV (columns 1-2: X/Y pixel coordinates; remaining columns: ion intensities per m/z; one row per pixel)
- Pixel-to-region assignment table (cluster or ROI labels for each pixel, e.g., MB vs HIP groups from Cardinal spatial shrunken centroids)

## Outputs

- Per-ion Wilcoxon rank-sum p-values and BH FDR-adjusted p-values
- Differential metabolite table with fold-change, FDR-adjusted p-value, up/down direction
- Filtered counts of upregulated and downregulated metabolites (benchmark: 90 up in MB / 94 down in HIP)

## How to apply

Treat each spatial pixel as an independent sample: for each of the ion peak columns (e.g., the 3,044 m/z columns of the mouse brain AFAD-ESI(+) feature matrix), run the Wilcoxon rank-sum test between the pixel groups of the two ROIs (e.g., Cardinal-defined midbrain vs hippocampus clusters). Convert raw p-values to Benjamini-Hochberg FDR-adjusted p-values across all ions. Filter differential metabolites with Fold Change > 2 or < 0.5 and FDR-adjusted p-value < 0.01. On the SMAnalyst mouse brain benchmark this yields 90 upregulated and 94 downregulated metabolites and ~89% overlap with SMAnalyst's own differential results, which serves as the reproduction target. The rationale for the rank-sum test is that pixel-level ion intensities in imaging MS are non-normal and pixel counts are large (e.g., >14,000 pixels), making a robust non-parametric test appropriate.

## Related tools

- **SMAnalyst** (Web platform providing flexible differential analysis with fold-change/significance filters; supplies the example feature matrix and reference differential results) — https://github.com/mzlab-research/SManalyst
- **R (>= v4.1.0)** (Runtime for running wilcox.test and p.adjust (method='BH') on pixel-grouped ion intensities)
- **Cardinal** (Used in the benchmark to define the comparison regions (spatial shrunken centroids, 25 clusters) whose pixels feed the Wilcoxon test)

## Examples

```
p <- apply(X, 1, function(v) wilcox.test(v[grp=="MB"], v[grp=="HIP"])$p.value); padj <- p.adjust(p, method="BH"); sig <- fc > 2 | fc < 0.5 & padj < 0.01
```

## Evaluation signals

- Filtered differential metabolite counts match the reported 90 upregulated (MB) and 94 downregulated (HIP) at FC > 2 or < 0.5 and FDR < 0.01
- Overlap between the Wilcoxon/Cardinal differential set and SMAnalyst's differential results reaches ~89%
- All reported FDR values are <= the corresponding raw p-values and are monotone under Benjamini-Hochberg ordering
- Result table contains one row per tested ion peak (3,044 ions in the benchmark dataset) with no missing p-values
- Clusters used as regions are anatomically recognizable (midbrain and hippocampus) before testing

## Limitations

- Treating all pixels in a region as samples ignores spatial autocorrelation, which can inflate significance; counts are only validated against this specific mouse brain benchmark (Supplementary Figure S16B,C)
- Requires prior region definition (auto-clustered or manually drawn ROIs); results depend on the clustering method (e.g., Cardinal 25-cluster solution vs SMAnalyst Seurat-LV/LM/SLM or UMAP-kmeans)
- The 90/94 metabolite counts and 89% overlap were demonstrated on AFAD-ESI(+) 100 um mouse brain data and may not transfer to other tissues or acquisition modes

## Evidence

- [readme] Differential metabolite detection with fold-change/significance filters: "Differential metabolite detection with fold-change/significance filters"
- [intro] Flexible differential analysis regions: "It also offers flexible differential analysis (cluster- or user-defined regions)."
- [supplementary] Benchmark workflow with thresholds and counts: [evidence span withheld — anchor status: not_found]
- [readme] Example dataset scale: "Content: 14,260 pixels × 3,044 ions"
- [supplementary] Overlap validation target: [evidence span withheld — anchor status: not_found]
