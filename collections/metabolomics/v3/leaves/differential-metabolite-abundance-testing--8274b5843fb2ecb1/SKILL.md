---
name: differential-metabolite-abundance-testing
description: Use when a pixel x m/z feature matrix (e.g., 14,260 pixels x 3,044 ions
  from AFAD-ESI positive-mode mouse brain IMS at 100 um) has region assignments (auto-clusters
  or manual ROIs) and you need up/downregulated differential metabolites between two
  regions.
when_to_use_negative:
- Input is raw imaging MS data (not yet peak-picked into a pixel x m/z feature matrix
  CSV) — SMAnalyst requires the processed feature matrix, not instrument files
- No spatial region definition exists (no clusters, no manual ROIs), so the two comparison
  groups cannot be formed
- The question requires true biological-replicate statistics across tissue sections
  rather than within-section pixel-level comparisons, since this method treats pixels
  in a region as samples
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_0091
tools:
- name: SMAnalyst
  role: 'Web platform performing the differential analysis: automatic ROI generation
    from clustering or manual ROI selection, differential metabolite detection with
    fold-change/significance filters, and CSV/PNG export'
  repo: https://github.com/mzlab-research/SManalyst
- name: Cardinal
  role: R package whose spatial shrunken centroids clustering defines the comparison
    regions (MB, HIP) used as the benchmark pipeline for the differential test
- name: R (>= v4.1.0)
  role: Runtime environment for the local SMAnalyst installation and for the R-based
    Wilcoxon/BH differential testing in the Cardinal benchmark
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.3390/biom15111562
    title: 'SMAnalyst: A Web Server for Spatial Metabolomic Data Analysis and Annotation'
  source_content_hash: sha256:d85a6fe056640a02a31952809d1ceded3f25852198f4d6560ff54efcecd20e19
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mzlab-research/SManalyst
  tools:
  - Cardinal
  - R (>= v4.1.0)
  - SMAnalyst
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8421052631578947
n_claims: 19
---

# differential-metabolite-abundance-testing

## Summary

Identifies metabolites (ion peaks) whose abundances differ between spatially defined regions of a tissue, using the Wilcoxon rank-sum test with Benjamini-Hochberg FDR correction and fold-change filters, as implemented in SMAnalyst's flexible differential analysis module and benchmarked against Cardinal.

## When to use

Use when a pixel x m/z feature matrix (e.g., 14,260 pixels x 3,044 ions from AFAD-ESI positive-mode mouse brain IMS at 100 um) has region assignments (auto-clusters or manual ROIs) and you need up/downregulated differential metabolites between two regions.

## When NOT to use

- Input is raw imaging MS data (not yet peak-picked into a pixel x m/z feature matrix CSV) — SMAnalyst requires the processed feature matrix, not instrument files
- No spatial region definition exists (no clusters, no manual ROIs), so the two comparison groups cannot be formed
- The question requires true biological-replicate statistics across tissue sections rather than within-section pixel-level comparisons, since this method treats pixels in a region as samples

## Inputs

- Feature matrix in CSV format: columns 1-2 are pixel coordinates (X, Y), subsequent columns are ion intensities at each m/z (e.g., m/z_100.002, m/z_101.005), one row per spatial pixel
- Mouse brain example dataset: 14,260 pixels x 3,044 ion peaks, AFAD-ESI (+) mode, 100 um pixel size (SMAnalyst Tutorial Panel example data or NGDC OMIX OMIX009541)
- Region/group labels per pixel, from SMAnalyst clustering, manually drawn ROIs, or external clustering (e.g., Cardinal spatial shrunken centroids, 25 categories)

## Outputs

- Table of differential metabolites per region comparison with Wilcoxon p-values, BH-adjusted FDR, and fold change (e.g., 90 upregulated in MB, 94 downregulated vs. HIP at FC > 2 / < 0.5, FDR < 0.01)
- Downloadable CSV result tables and PNG images from SMAnalyst's Visualize & Export step
- Overlap assessment between differential metabolite sets (e.g., 89% SMAnalyst-Cardinal overlap)

## How to apply

First define the two regions to compare: either use SMAnalyst's pixel clustering (Seurat-LV, Seurat-LM, Seurat-SLM, or UMAP-kmeans) to auto-generate ROIs, draw manual ROIs via the interactive tissue imaging, or import cluster labels from an external tool (in the Cardinal benchmark, Cardinal's spatial shrunken centroids assigned pixels into 25 spatial categories, from which midbrain (MB) and hippocampal (HIP) groups were defined). Then perform differential analysis treating all pixels in each region group as samples, testing each ion with the Wilcoxon rank-sum test and correcting with Benjamini-Hochberg FDR. Apply the filters Fold Change > 2 or < 0.5 together with FDR-adjusted p-value < 0.01 to call differential metabolites, separating upregulated (FC > 2) and downregulated (FC < 0.5) sets per region. The rationale for pixel-level testing is that spatial regions provide large within-region sample counts; cross-tool consistency can be assessed by overlapping the differential metabolite sets from SMAnalyst and the Cardinal-based pipeline.

## Related tools

- **SMAnalyst** (Web platform performing the differential analysis: automatic ROI generation from clustering or manual ROI selection, differential metabolite detection with fold-change/significance filters, and CSV/PNG export) — https://github.com/mzlab-research/SManalyst
- **Cardinal** (R package whose spatial shrunken centroids clustering defines the comparison regions (MB, HIP) used as the benchmark pipeline for the differential test)
- **R (>= v4.1.0)** (Runtime environment for the local SMAnalyst installation and for the R-based Wilcoxon/BH differential testing in the Cardinal benchmark)

## Examples

```
Rscript -e 'p <- apply(fm, 1, function(x) wilcox.test(x ~ group)$p.value); padj <- p.adjust(p, method="BH"); diff <- subset(res, (FC > 2 | FC < 0.5) & padj < 0.01)'
```

## Evaluation signals

- On the mouse brain benchmark (MB vs. HIP), the filtered result set matches the reported counts: 90 upregulated (MB) and 94 downregulated (HIP) metabolites at FC > 2 or < 0.5 and FDR-adjusted p < 0.01
- The overlap of the Cardinal-based differential metabolite set with SMAnalyst's differential results reaches 89% (Supplementary Figure S16B,C)
- FDR correction uses Benjamini-Hochberg and the p-value column is adjusted (qvalue/BH), not raw Wilcoxon p-values, before the < 0.01 cutoff is applied
- Region groups derive from anatomically recognizable clusters (midbrain and hippocampus), not arbitrary pixel partitions
- Exported result table is a CSV with per-ion statistics and passes the fold-change/FC-direction sanity check (upregulated FC > 2, downregulated FC < 0.5)

## Limitations

- Pixels are treated as samples for the Wilcoxon test, which inflates the effective sample size and does not model biological-replicate variability across sections
- Results depend on upstream region definition: Cardinal's 25-cluster spatial shrunken centroids output vs. SMAnalyst's four pixel-clustering algorithms (Seurat-LV, Seurat-LM, Seurat-SLM, UMAP-kmeans) or manual ROIs can change the differential calls
- Ions entering the test depend on prior noise-ion filtering (default threshold score >= 30) during QC, so overly strict/loose QC changes the candidate metabolite set

## Evidence

- [other] Perform differential analysis between the MB and HIP pixel groups using the Wilcoxon rank-sum test with Benjamini-Hochberg FDR correction: [evidence span withheld — anchor status: not_found]
- [readme] Differential metabolite detection with fold-change/significance filters: "Differential metabolite detection with fold-change/significance filters"
- [readme] It integrates quality control, metabolite annotation, spatial pattern discovery, and differential analysis into a unified workflow.: "It integrates quality control, metabolite annotation, spatial pattern discovery, and differential analysis into a unified workflow."
- [other] Filter differential metabolites using Fold Change > 2 or < 0.5 and FDR-adjusted p-value < 0.01: [evidence span withheld — anchor status: not_found]
