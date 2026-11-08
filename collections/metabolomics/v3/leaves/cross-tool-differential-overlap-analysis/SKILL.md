---
name: cross-tool-differential-overlap-analysis
description: Use when a pixel × ion feature matrix (e.g., 14,260 pixels × 3,044 ion
  peaks, AFAD-ESI positive mode) has differential analysis results from one spatial
  metabolomics tool and you need to cross-validate them against an independent tool's
  clustering and differential calls.
when_to_use_negative:
- Input is raw imaging MS spectra rather than a pre-extracted pixel × ion feature
  matrix — SMAnalyst requires the CSV feature-matrix format (X, Y, m/z intensity columns),
  not raw instrument files
- The goal is de novo marker ion discovery or peak picking (e.g., via SmartGate's
  graph-attention auto-encoder) rather than benchmarking consistency of differential
  calls between two tools
- No independent tool's spatial clusters or manually drawn ROI labels are available
  to define the two comparison regions needed for the cross-tool contrast
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: Cardinal
  role: 'Independent R tool used for benchmarking: spatial shrunken centroids clustering
    to define anatomical regions for the cross-tool differential comparison'
- name: SMAnalyst
  role: Reference platform whose differential analysis results on the mouse brain
    dataset are compared against Cardinal's calls to compute overlap
  repo: mzlab-research/SManalyst
- name: R
  role: Scripting environment (>= v4.1.0) for running Cardinal clustering and the
    Wilcoxon rank-sum differential test with BH correction
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.3390/biom15111562
    title: 'SMAnalyst: A Web Server for Spatial Metabolomic Data Analysis and Annotation'
  source_content_hash: sha256:5f88edf0b9e2dfcd77f73549f13bdcebfe1e062f2d00b143b79d82ea9ceec789
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mzlab-research/SManalyst
  tools:
  - Cardinal
  - R
  - SMAnalyst
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# cross-tool-differential-overlap-analysis

## Summary

Benchmark the consistency of differential metabolite calls from SMAnalyst against an independent tool (Cardinal) on the same spatial metabolomics feature matrix, quantifying overlap of differential metabolite sets. This validates that region-detection differences do not propagate into inconsistent biology.

## When to use

Use when a pixel × ion feature matrix (e.g., 14,260 pixels × 3,044 ion peaks, AFAD-ESI positive mode) has differential analysis results from one spatial metabolomics tool and you need to cross-validate them against an independent tool's clustering and differential calls.

## When NOT to use

- Input is raw imaging MS spectra rather than a pre-extracted pixel × ion feature matrix — SMAnalyst requires the CSV feature-matrix format (X, Y, m/z intensity columns), not raw instrument files
- The goal is de novo marker ion discovery or peak picking (e.g., via SmartGate's graph-attention auto-encoder) rather than benchmarking consistency of differential calls between two tools
- No independent tool's spatial clusters or manually drawn ROI labels are available to define the two comparison regions needed for the cross-tool contrast

## Inputs

- Pixel × ion feature matrix CSV (columns 1-2 pixel X/Y coordinates, subsequent columns ion intensities per m/z; 14,260 pixels × 3,044 ions for the mouse brain test dataset)
- SMAnalyst differential metabolite results from the same dataset (cluster- or ROI-defined regions)
- Raw data accession NGDC OMIX OMIX009541 as an alternative source of the test dataset

## Outputs

- Cardinal spatial shrunken centroids cluster labels for all 14,260 pixels (25 categories)
- Table of differential metabolites between MB and HIP pixel groups with Wilcoxon statistics, BH-adjusted p-values, and fold changes (reported: 90 upregulated in MB, 94 downregulated)
- Overlap percentage of differential metabolite sets between Cardinal and SMAnalyst (reported: 89%)

## How to apply

Load the same feature matrix into both tools — here the mouse brain coronal section dataset (14,260 pixels × 3,044 ion peaks, AFAD-ESI (+) mode, 100 μm pixel size, NGDC OMIX009541 or the SMAnalyst Tutorial Panel example data). In R, run Cardinal's spatial shrunken centroids to assign pixels into 25 spatial categories, then map the Cardinal-defined clusters onto anatomical comparison regions (midbrain vs hippocampal). Run differential analysis between the two pixel groups using the Wilcoxon rank-sum test with Benjamini-Hochberg FDR correction, treating all pixels in a group as samples. Filter with Fold Change > 2 or < 0.5 and FDR-adjusted p-value < 0.01, then compute the set overlap between the Cardinal differential metabolites and the SMAnalyst differential results from the same dataset. The rationale is that a high overlap (reported at 89%) demonstrates the tools' region definitions and differential statistics converge, so conclusions are tool-independent.

## Related tools

- **Cardinal** (Independent R tool used for benchmarking: spatial shrunken centroids clustering to define anatomical regions for the cross-tool differential comparison)
- **SMAnalyst** (Reference platform whose differential analysis results on the mouse brain dataset are compared against Cardinal's calls to compute overlap) — mzlab-research/SManalyst
- **R** (Scripting environment (>= v4.1.0) for running Cardinal clustering and the Wilcoxon rank-sum differential test with BH correction)

## Examples

```
Rscript -e 'wt <- wilcox.test(intensity ~ region, data=mb_hip); padj <- p.adjust(wt$p.value, method="BH"); sig <- fc > 2 | fc < 0.5 & padj < 0.01'
```

## Evaluation signals

- Cardinal's spatial shrunken centroids on 25 categories yields clusters that map cleanly onto anatomically recognizable regions (midbrain vs hippocampal)
- Differential counts under Fold Change > 2 or < 0.5 and FDR-adjusted p < 0.01 match the reported 90 upregulated (MB) and 94 downregulated (HIP) metabolites
- Set overlap between Cardinal and SMAnalyst differential metabolites reaches the reported 89% (Supplementary Figure S16B,C)
- Both tools were run on the identical feature matrix (14,260 pixels × 3,044 ion peaks) so differences are attributable to method, not data
- Wilcoxon p-values are BH-corrected across all ions before the FDR < 0.01 filter is applied

## Limitations

- Pixel groups are treated as samples in the Wilcoxon test; spatial autocorrelation between adjacent 100 μm pixels may inflate significance
- Overlap depends on the Cardinal clustering resolution (25 categories) and the choice of which clusters map to midbrain vs hippocampal regions
- SMAnalyst's differential analysis supports only cluster- or user-defined ROI contrasts; results vary if regions are defined differently across tools

## Evidence

- [intro] Differential metabolite detection with fold-change/significance filters: "Differential metabolite detection with fold-change/significance filters"
- [abstract] Flexible differential analysis scope: "It also offers flexible differential analysis (cluster- or user-defined regions)."
- [readme] Test dataset acquisition parameters: "Acquisition: AFAD-ESI (+) mode, 100μm resolution"
- [readme] Pixel clustering algorithms available for ROI definition: "Pixel-level clustering: Identify tissue regions using 4 algorithms"
