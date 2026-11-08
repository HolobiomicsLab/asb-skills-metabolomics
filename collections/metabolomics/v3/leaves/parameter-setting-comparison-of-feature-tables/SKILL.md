---
name: parameter-setting-comparison-of-feature-tables
description: Use when multiple apLCMS parameter settings (or apLCMS/XCMS vs a two-stage
  batch-aware pipeline) have produced feature tables for a two-group design, and you
  need to count features significant at FDR <= 0.2 per setting.
when_to_use_negative:
- The input feature tables have already been batch-effect corrected — post hoc correction
  changes intensity structure and defeats the purpose of comparing uncorrected preprocessing
  outputs
- There is only a single feature table with no parameter-setting variation to compare
  across
- The design is not a two-group comparison suitable for a two-sample t-test (e.g.
  multi-class or time-course labels)
tools:
- name: apLCMS
  role: Generates the feature tables across parameter settings via cdf.to.ftr (peak
    detection, RT correction, alignment, weak signal recovery); the two-stage batch-aware
    method is implemented in this package
  repo: https://github.com/tianwei-yu/apLCMS
- name: R
  role: Used to filter features by missingness (<= 33% zeros per cell type), run two-sample
    t-tests, apply Benjamini-Hochberg FDR correction, and count significant features
    per setting
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41598-020-70850-0
    title: Addressing the batch effect issue for LC/MS metabolomics data in data preprocessing
  source_content_hash: sha256:8d2981626242922cbf528a257466875cf71b484ac8ee72250eadbf3563c6752b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/tianwei-yu/apLCMS
  tools:
  - R
  - apLCMS
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# parameter-setting-comparison-of-feature-tables

## Summary

Generates and statistically compares uncorrected LC/MS feature tables across multiple apLCMS parameter settings to test whether the two-stage batch-aware preprocessing detects more significant features (two-sample t-test, Benjamini-Hochberg FDR <= 0.2) than traditional preprocessing, e.g. on the ST000868 oak vs wine yeast metabolomics dataset.

## When to use

Use when multiple apLCMS parameter settings (or apLCMS/XCMS vs a two-stage batch-aware pipeline) have produced feature tables for a two-group design, and you need to count features significant at FDR <= 0.2 per setting.

## When NOT to use

- The input feature tables have already been batch-effect corrected — post hoc correction changes intensity structure and defeats the purpose of comparing uncorrected preprocessing outputs
- There is only a single feature table with no parameter-setting variation to compare across
- The design is not a two-group comparison suitable for a two-sample t-test (e.g. multi-class or time-course labels)

## Inputs

- Uncorrected ST000868 feature data matrices (one per apLCMS parameter setting, including the two-stage method, apLCMS, and XCMS)
- Sample-to-cell-type label table separating oak and wine yeast samples

## Outputs

- fig3d_significant_feature_counts.csv: counts of features with BH-adjusted p-value <= 0.2 per parameter setting
- Per-setting vectors of raw and FDR-adjusted t-test p-values

## How to apply

Load each uncorrected feature data matrix produced across the apLCMS parameter settings (e.g. via cdf.to.ftr with varying min.exp/min.pres/min.run/mz.tol), and use the cell-type labels to separate oak and wine yeast samples. Retain only features with <= 33% zero (missing) values within at least one of the two cell types, since pervasive missingness invalidates the t-test. For every retained feature run a two-sample t-test comparing oak versus wine yeast intensity values, then apply Benjamini-Hochberg FDR correction across features within each parameter setting separately. Count features with adjusted p-value <= 0.2 per parameter setting and write the counts table to fig3d_significant_feature_counts.csv. The rationale is that traditional preprocessing treating all samples as a single group can misalign peaks, an error that post hoc batch effect correction cannot fix, so the comparison is run on uncorrected tables to expose the preprocessing advantage.

## Related tools

- **apLCMS** (Generates the feature tables across parameter settings via cdf.to.ftr (peak detection, RT correction, alignment, weak signal recovery); the two-stage batch-aware method is implemented in this package) — https://github.com/tianwei-yu/apLCMS
- **R** (Used to filter features by missingness (<= 33% zeros per cell type), run two-sample t-tests, apply Benjamini-Hochberg FDR correction, and count significant features per setting)

## Examples

```
Rscript -e 'p<-apply(ftr,1,function(x)t.test(x~celltype)$p.value); fdr<-p.adjust(p,"BH"); sum(fdr<=0.2)'
```

## Evaluation signals

- fig3d_significant_feature_counts.csv contains one count row per parameter setting, each computed after Benjamini-Hochberg correction within that setting
- Every tested feature has <= 33% zero values within at least one of the two cell types (missingness filter applied before testing)
- Adjusted p-values are bounded in [0,1] and the <= 0.2 cutoff is applied to adjusted, not raw, p-values
- The per-setting counts qualitatively reproduce Fig. 3d: the two-stage method tends to detect more significant features (FDR <= 0.2) than traditional apLCMS or XCMS without batch correction
- Counts remain relatively low overall, consistent with the reported finding of few significant metabolites at FDR <= 0.2 without batch effect correction

## Limitations

- Traditional preprocessing treating all samples as a single group can cause peak alignment errors that cannot be corrected by post hoc batch effect correction, so results are specific to uncorrected tables
- Without batch effect correction, all methods yielded relatively few significant metabolites at FDR <= 0.2, so differences between settings may be modest and only qualitative
- No apLCMS package version is stated in the article or repository context, so exact reproducibility of parameter settings is not guaranteed

## Evidence

- [abstract] misalignment motivates preprocessing-stage comparison: "Such practice can result in errors in the alignment of peaks, which cannot be corrected by post hoc application of batch effect correction methods"
- [intro] two-stage approach rationale: "we propose a new approach that preprocess the data in a two-stage manner"
- [methods] feature table generation entry point: "cdf.to.ftr(folder, file.pattern, n.nodes, min.exp, min.pres, min.run, mz.tol, baseline.correct.noise.percentile, shape.model"
- [discussion] expected outcome pattern: "we showed that the new method improved the consistency of feature detection and quantification"
- [abstract] implementation platform: "The method is implemented in the existing workflow of the apLCMS platform."
