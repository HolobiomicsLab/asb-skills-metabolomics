---
name: lcms-feature-intensity-ttesting
description: Use when you have a feature-intensity matrix from LC/MS preprocessing
  (apLCMS or XCMS) plus batch and phenotype labels, and need to test each feature
  for between-group differences after batch-effect correction.
when_to_use_negative:
- Raw LC/MS data files still need peak detection, RT correction, alignment, and weak
  signal recovery — run cdf.to.ftr / the preprocessing pipeline first; post hoc correction
  cannot fix misaligned features or unrecovered weak signals
- Data come from a single batch with no batch structure, making ComBat correction
  unnecessary
- The analysis requires multi-group or covariate-adjusted modeling rather than a two-sample
  comparison between exactly two cell types
edam_operation: http://edamontology.org/operation_2262
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3520
tools:
- name: R
  role: Environment in which ComBat correction, per-feature t-tests, and BH FDR adjustment
    are scripted; apLCMS dependencies (MASS, mzR, doParallel, foreach) must be attached
    explicitly with library()
- name: ComBat (sva)
  role: Empirical-Bayes batch-effect correction of the feature-intensity matrix, with
    batch as the batch covariate and cell type preserved as a biological covariate
- name: apLCMS
  role: Source of the feature-intensity matrices being compared; the two-stage batch-aware
    variant produced more significant features at FDR <= 0.2 after ComBat
  repo: https://github.com/tianwei-yu/apLCMS
- name: XCMS
  role: Alternative preprocessing pipeline whose feature matrix serves as a comparator
    for significant-feature counts
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41598-020-70850-0
    title: Addressing the batch effect issue for LC/MS metabolomics data in data preprocessing
  source_content_hash: sha256:540722b94191093d47342f79a68afb571a2a29a5af5f27ea88dc366566ccfd0a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/tianwei-yu/apLCMS
  tools:
  - ComBat (sva)
  - R
  - XCMS
  - apLCMS
  edam_operation: http://edamontology.org/operation_2262
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6153846153846154
n_claims: 13
---

# lcms-feature-intensity-ttesting

## Summary

Per-feature two-sample t-testing on LC/MS metabolomics feature-intensity matrices after ComBat batch correction, followed by Benjamini-Hochberg FDR adjustment, to count significant metabolic features between sample groups (here, two yeast cell types in ST000868) at FDR <= 0.2.

## When to use

Use when you have a feature-intensity matrix from LC/MS preprocessing (apLCMS or XCMS) plus batch and phenotype labels, and need to test each feature for between-group differences after batch-effect correction.

## When NOT to use

- Raw LC/MS data files still need peak detection, RT correction, alignment, and weak signal recovery — run cdf.to.ftr / the preprocessing pipeline first; post hoc correction cannot fix misaligned features or unrecovered weak signals
- Data come from a single batch with no batch structure, making ComBat correction unnecessary
- The analysis requires multi-group or covariate-adjusted modeling rather than a two-sample comparison between exactly two cell types

## Inputs

- Feature-intensity matrix (features x samples) from apLCMS, two-stage apLCMS, or XCMS preprocessing
- Batch annotation table mapping each sample to its acquisition batch
- Cell-type / phenotype annotation table for the two compared groups (e.g. yeast cell types in ST000868)

## Outputs

- Per-feature t-test p-values on ComBat-corrected intensities
- Benjamini-Hochberg FDR-adjusted p-values
- Table of significant-feature counts at FDR <= 0.2 per preprocessing parameter setting (e.g. fig3e_fdr_significant_counts.csv)

## How to apply

Load the feature-intensity matrix together with batch and cell-type annotation tables into R. Apply ComBat empirical-Bayes batch correction (sva package) to the intensity matrix using batch labels as the batch covariate and cell type as the preserved biological covariate — this post hoc correction adjusts between-batch intensity differences but cannot fix peak alignment errors, so the upstream feature table must already be consistent. For each feature, run a two-sample t-test comparing the two cell types on the ComBat-corrected intensities. Adjust t-test p-values with Benjamini-Hochberg FDR and count features with FDR <= 0.2 per parameter setting. Assemble per-setting counts into a table (e.g. reproducing Fig. 3e of the batch-effect paper) and export as CSV; note that in the ST000868 comparison the two-stage apLCMS approach yielded more significant features than traditional apLCMS and XCMS after ComBat.

## Related tools

- **R** (Environment in which ComBat correction, per-feature t-tests, and BH FDR adjustment are scripted; apLCMS dependencies (MASS, mzR, doParallel, foreach) must be attached explicitly with library())
- **ComBat (sva)** (Empirical-Bayes batch-effect correction of the feature-intensity matrix, with batch as the batch covariate and cell type preserved as a biological covariate)
- **apLCMS** (Source of the feature-intensity matrices being compared; the two-stage batch-aware variant produced more significant features at FDR <= 0.2 after ComBat) — https://github.com/tianwei-yu/apLCMS
- **XCMS** (Alternative preprocessing pipeline whose feature matrix serves as a comparator for significant-feature counts)

## Examples

```
library(sva); corrected <- ComBat(ftr[, samples], batch=ann$batch, mod=model.matrix(~cell_type, ann)); pval <- apply(corrected, 1, function(x) t.test(x ~ ann$cell_type)$p.value); fdr <- p.adjust(pval, "BH"); sum(fdr <= 0.2)
```

## Evaluation signals

- The output table contains a significant-feature count for every preprocessing parameter setting
- Each reported count equals the number of features with Benjamini-Hochberg FDR <= 0.2, reproducible from the t-test p-values
- Counts match the bar values shown in Fig. 3e of the batch-effect paper when the ST000868 dataset and the same parameter settings are used
- ComBat run uses batch labels as the batch covariate while preserving cell type as a biological covariate (verify with model.matrix design)
- The two-stage apLCMS count should exceed traditional apLCMS and XCMS counts under ComBat correction, per the published finding

## Limitations

- Post hoc batch-effect correction can only adjust signal intensity; it cannot correct peak misalignment across batches or incomplete weak-signal recovery from the original data
- Traditional preprocessing that treats all samples as a single group can produce alignment errors that cannot be fixed by downstream correction, biasing downstream feature counts
- The article's methods do not state an apLCMS package version, so exact reproduction may depend on the package build used

## Evidence

- [abstract] The new method addresses batch effects in the preprocessing stage and can be combined with down-stream batch effect correction methods: [evidence span withheld — anchor status: not_found]
- [intro] Post hoc correction limitation: "However, such approaches can only adjust signal intensity. They cannot address issues such as misalignment of features across batches22, or the incomplete weak signal recovery"
- [methods] R dependency loading requirement: "Packages listed under Dependencies are NOT attached by loading this package (only Depends is); call library(<dep>) explicitly"
- [discussion] Improved consistency of feature detection and quantification: "we showed that the new method improved the consistency of feature detection and quantification"
