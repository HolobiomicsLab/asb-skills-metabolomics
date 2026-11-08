---
name: cell-type-differential-abundance-testing
description: Use when a samples-by-features LC/MS intensity matrix (raw or batch-corrected)
  has group labels and the goal is to rank or count metabolomic features that differ
  between groups at a relaxed FDR threshold such as ≤ 0.2.
when_to_use_negative:
- Input is already a table of adjusted p-values or significant-feature counts rather
  than a raw feature intensity matrix requiring testing
- Samples lack group labels (no cell-type or condition annotation to define the two
  groups for the t-test)
- The goal is batch effect correction itself (WaveICA/ComBat) rather than downstream
  differential testing of the resulting matrices
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0203
tools:
- name: R
  role: Environment in which the per-feature two-sample t-tests, Benjamini-Hochberg
    FDR adjustment, and significant-feature counting are implemented
- name: apLCMS
  role: Produces the feature tables (two-stage and traditional preprocessing variants)
    whose significant-feature counts at FDR ≤ 0.2 are compared
  repo: https://github.com/tianwei-yu/apLCMS
provenance:
  source_task_ids:
  - task_005
  - task_003
  source_papers:
  - doi: 10.1038/s41598-020-70850-0
    title: Addressing the batch effect issue for LC/MS metabolomics data in data preprocessing
  source_content_hash: sha256:b16d10a801067e23a0f083325ca980c80489e551d16e844b996f4aaf97d4c826
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/tianwei-yu/apLCMS
  tools:
  - R
  - WaveICA
  - apLCMS
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# cell-type-differential-abundance-testing

## Summary

Per-feature two-sample t-testing across pre-defined sample groups (e.g., oak vs. wine yeast cell types in ST000868 LC/MS feature matrices), with Benjamini-Hochberg FDR correction and counting of significant features at FDR ≤ 0.2 to compare preprocessing methods (two-stage apLCMS vs. traditional apLCMS/XCMS).

## When to use

Use when a samples-by-features LC/MS intensity matrix (raw or batch-corrected) has group labels and the goal is to rank or count metabolomic features that differ between groups at a relaxed FDR threshold such as ≤ 0.2.

## When NOT to use

- Input is already a table of adjusted p-values or significant-feature counts rather than a raw feature intensity matrix requiring testing
- Samples lack group labels (no cell-type or condition annotation to define the two groups for the t-test)
- The goal is batch effect correction itself (WaveICA/ComBat) rather than downstream differential testing of the resulting matrices

## Inputs

- Samples-by-features LC/MS feature intensity matrix (e.g., ST000868) with cell-type group annotations (oak vs. wine yeast)
- Batch-labeled corrected matrices from WaveICA or ComBat (for the batch-corrected comparison)
- Uncorrected feature tables produced across apLCMS parameter settings

## Outputs

- Per-parameter-setting counts of features with BH-adjusted p-value (FDR) ≤ 0.2 (e.g., fig3d_significant_feature_counts.csv, fig3f_significant_feature_counts.csv)
- Per-feature t-test p-values and BH-adjusted FDR values

## How to apply

Load the ST000868 feature intensity matrix (samples x features) with cell-type annotations separating oak and wine yeast samples. Optionally filter features to retain only those with ≤ 33% zero (missing) values within at least one of the two cell types, as in the uncorrected Fig. 3d analysis; when comparing post-WaveICA/ComBat corrected matrices (Fig. 3f), apply the t-test directly to each corrected matrix across the parameter settings being swept. For every retained feature run a two-sample t-test between the cell-type groups, then apply Benjamini-Hochberg FDR correction across all features within each matrix/parameter setting. Count features with adjusted p-value (FDR) ≤ 0.2 per setting and compile the counts into a summary CSV mirroring Fig. 3d/3f. The rationale is that a preprocessing method that improves peak detection, alignment, and quantification should surface more differentially abundant features, so significant-feature counts at FDR ≤ 0.2 serve as the downstream metric for judging preprocessing quality.

## Related tools

- **R** (Environment in which the per-feature two-sample t-tests, Benjamini-Hochberg FDR adjustment, and significant-feature counting are implemented)
- **apLCMS** (Produces the feature tables (two-stage and traditional preprocessing variants) whose significant-feature counts at FDR ≤ 0.2 are compared) — https://github.com/tianwei-yu/apLCMS

## Examples

```
pvals <- apply(ftr_matrix, 2, function(x) t.test(x ~ cell_type)$p.value); fdr <- p.adjust(pvals, method="BH"); sum(fdr <= 0.2)
```

## Evaluation signals

- Each feature's t-test p-value is BH-adjusted within its parameter setting; the significant count equals the number of adjusted p-values ≤ 0.2
- Reproduces the qualitative pattern of Fig. 3d: without batch correction, all methods yield relatively few significant metabolites at FDR ≤ 0.2, while the two-stage method detects more
- After WaveICA correction (Fig. 3f), the two-stage approach again reports more significant features at FDR ≤ 0.2 than traditional apLCMS and XCMS, with results similar to ComBat
- Counts table is emitted as a per-setting CSV (e.g., fig3d_significant_feature_counts.csv) with one row per apLCMS/WaveICA parameter setting
- In the uncorrected analysis, filtered features satisfy ≤ 33% zero values within at least one cell type before testing

## Limitations

- Post hoc batch effect correction methods can only adjust signal intensity; they cannot fix feature misalignment or incomplete weak signal recovery, so differential testing on corrected matrices cannot rescue preprocessing-stage errors
- Traditional preprocessing treating all samples as a single group can introduce peak alignment errors that cannot be corrected post hoc, capping the significant features recoverable by downstream testing
- Without batch effect correction, all methods yielded relatively few significant metabolites at FDR ≤ 0.2 on ST000868, so the uncorrected comparison is a weak-signal regime

## Evidence

- [abstract] new method resulted in feature tables with better consistency, as well as better down-stream analysis results: "the new method resulted in feature tables with better consistency, as well as better down-stream analysis results"
- [intro] such approaches can only adjust signal intensity: "However, such approaches can only adjust signal intensity. They cannot address issues such as misalignment of features across batches22, or the incomplete weak signal recovery from the original data."
- [abstract] Such practice can result in errors in the alignment of peaks, which cannot be corrected by post hoc application of batch effect correction methods: "Such practice can result in errors in the alignment of peaks, which cannot be corrected by post hoc application of batch effect correction methods"
- [discussion] two-stage approach for LC/MS metabolomics data generated in multiple batches: "We presented a two-stage approach for LC/MS metabolomics data generated in multiple batches."
- [methods] cdf.to.ftr: "`cdf.to.ftr(folder, file.pattern, n.nodes, min.exp, min.pres, min.run, mz.tol, baseline.correct.noise.percentile, shape.model"
