---
name: batch-correction-parameter-sensitivity-analysis
description: Use when comparing batch-correction configurations (e.g. WaveICA vs ComBat
  vs none) on a batch-labeled metabolomics feature table, where the outcome metric
  is the count of two-sample t-test features passing FDR <= 0.2 between biological
  groups.
when_to_use_negative:
- Raw spectral data files (e.g. NetCDF/mzML) that still need peak detection, RT correction,
  alignment, and weak-signal recovery — use the two-stage apLCMS cdf.to.ftr preprocessing
  first; batch correction methods cannot fix alignment errors post hoc.
- Single-batch datasets with no between-batch intensity differences, where correction
  parameters have nothing to correct and the sweep adds no information.
- Analyses where the outcome must be intensity-consistency metrics rather than group-differential
  significance counts.
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: WaveICA
  role: Performs the batch effect correction on the feature intensity matrix; its
    parameters are swept to generate each corrected matrix tested.
- name: R
  role: Runtime environment for applying correction settings, running per-feature
    two-sample t-tests, Benjamini-Hochberg FDR adjustment, and compiling counts/figures.
- name: apLCMS
  role: Source of the two-stage preprocessing pipeline whose feature tables serve
    as the comparison benchmark; the method is implemented in the existing apLCMS
    workflow.
  repo: https://github.com/tianwei-yu/apLCMS
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41598-020-70850-0
    title: Addressing the batch effect issue for LC/MS metabolomics data in data preprocessing
  source_content_hash: sha256:39cd3f9e089d638f0db2846b23027e5021aa3d6c6590706f7f0052d0da591aa6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/tianwei-yu/apLCMS
  tools:
  - R
  - WaveICA
  - apLCMS
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# batch-correction-parameter-sensitivity-analysis

## Summary

A workflow for sweeping batch-effect correction settings (e.g. WaveICA) over an LC/MS feature intensity matrix and scoring each setting by the number of features significant after Benjamini-Hochberg FDR correction at FDR <= 0.2, mirroring Fig. 3f of the two-stage apLCMS preprocessing paper (Scientific Reports, 2020).

## When to use

Use when comparing batch-correction configurations (e.g. WaveICA vs ComBat vs none) on a batch-labeled metabolomics feature table, where the outcome metric is the count of two-sample t-test features passing FDR <= 0.2 between biological groups.

## When NOT to use

- Raw spectral data files (e.g. NetCDF/mzML) that still need peak detection, RT correction, alignment, and weak-signal recovery — use the two-stage apLCMS cdf.to.ftr preprocessing first; batch correction methods cannot fix alignment errors post hoc.
- Single-batch datasets with no between-batch intensity differences, where correction parameters have nothing to correct and the sweep adds no information.
- Analyses where the outcome must be intensity-consistency metrics rather than group-differential significance counts.

## Inputs

- Batch-labeled LC/MS feature intensity matrix (samples x features) with group annotations, e.g. the ST000868 cell-type dataset
- Batch identifiers and cell-type group labels per sample
- Sweep of WaveICA parameter settings to compare
- Baseline feature tables from traditional apLCMS and XCMS preprocessing for comparison

## Outputs

- Per-setting count of significant features at FDR <= 0.2 (summary table, e.g. fig3f_significant_feature_counts.csv)
- Bar-chart figure mirroring Fig. 3f comparing significant-feature counts across correction settings and preprocessing pipelines

## How to apply

Load a preprocessed, batch-labeled feature intensity matrix (samples x features, e.g. ST000868 with cell-type annotations) into R. For each WaveICA parameter setting under comparison, apply the batch correction to the full intensity matrix, then run a two-sample t-test on every feature between the cell-type groups. Adjust the t-test p-values with Benjamini-Hochberg FDR correction and count features with FDR <= 0.2 per setting. Compile the per-setting counts into a summary table (fig3f_significant_feature_counts.csv) and a bar-chart figure, comparing against the two-stage apLCMS preprocessing result as well as traditional apLCMS and XCMS pipelines, which the paper shows detect fewer significant features. The rationale is that downstream batch correction (WaveICA, ComBat) only adjusts signal intensity and cannot fix feature misalignment or incomplete weak-signal recovery, so correction settings must be evaluated on their downstream detection power, not just visually.

## Related tools

- **WaveICA** (Performs the batch effect correction on the feature intensity matrix; its parameters are swept to generate each corrected matrix tested.)
- **R** (Runtime environment for applying correction settings, running per-feature two-sample t-tests, Benjamini-Hochberg FDR adjustment, and compiling counts/figures.)
- **apLCMS** (Source of the two-stage preprocessing pipeline whose feature tables serve as the comparison benchmark; the method is implemented in the existing apLCMS workflow.) — https://github.com/tianwei-yu/apLCMS

## Examples

```
p <- apply(mat, 2, function(x) t.test(x ~ celltype)$p.value); fdr <- p.adjust(p, "BH"); counts <- sum(fdr <= 0.2)
```

## Evaluation signals

- For every corrected matrix, the number of features with Benjamini-Hochberg FDR <= 0.2 from per-feature two-sample t-tests is recorded, not eyeballed from volcano plots.
- The two-stage apLCMS preprocessing shows more significant features at FDR <= 0.2 than traditional apLCMS and XCMS, consistent with Fig. 3f.
- WaveICA-corrected results are comparable to ComBat-corrected results on the same ST000868 matrices, as reported in the paper.
- The summary table contains one significant-feature count row per swept WaveICA parameter setting and exports cleanly to fig3f_significant_feature_counts.csv.
- Adjusted p-values derive from BH correction over all features within each corrected matrix, so the FDR count depends on the full feature set size.

## Limitations

- Post hoc batch-effect correction can only adjust signal intensity; it cannot address misalignment of features across batches or incomplete weak-signal recovery from the original data.
- Preprocessing all samples as a single group can cause peak alignment errors that cannot be corrected post hoc by batch effect correction methods.
- The paper does not state an apLCMS package version or explicit limitations of the new method in the discussion.

## Evidence

- [results] After applying WaveICA batch correction to all ST000868 data matrices, results were similar to ComBat, and the two-stage approach again detected more significant features at FDR ≤ 0.2 than traditional apLCMS and XCMS: [evidence span withheld — anchor status: not_found]
- [intro] However, such approaches can only adjust signal intensity.: "However, such approaches can only adjust signal intensity. They cannot address issues such as misalignment of features across batches22, or the incomplete weak signal recovery from the original data."
- [abstract] Such practice can result in errors in the alignment of peaks, which cannot be corrected by post hoc application of batch effect correction methods: "Such practice can result in errors in the alignment of peaks, which cannot be corrected by post hoc application of batch effect correction methods"
- [methods] Packages listed under Dependencies are NOT attached by loading this package (only Depends is); call library(<dep>) explicitly: "Packages listed under Dependencies are NOT attached by loading this package (only Depends is); call library(<dep>) explicitly"
