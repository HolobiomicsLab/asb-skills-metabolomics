---
name: figure-reproduction-from-corrected-data
description: Use when a target figure reports per-setting counts or effect sizes (e.g.
  features passing FDR <= 0.2) that must be regenerated from feature-intensity matrices
  plus batch and phenotype annotation tables.
when_to_use_negative:
- The figure reports raw peak-detection or alignment performance (e.g. number of features
  detected) rather than downstream statistical significance counts — batch correction
  and t-testing are not needed
- Only a single batch is present, so ComBat empirical-Bayes between-batch correction
  has no batch covariate to adjust
- The target figure is about misalignment or weak-signal recovery in preprocessing
  itself — the paper notes post-hoc correction can only adjust signal intensity and
  cannot fix misaligned features
edam_operation: http://edamontology.org/operation_3664
edam_topics:
- http://edamontology.org/topic_3517
tools:
- name: apLCMS
  role: Source of the feature-intensity matrices (two-stage and traditional preprocessing
    pipelines) whose ComBat-corrected significant-feature counts are tabulated
  repo: https://github.com/tianwei-yu/apLCMS
- name: ComBat (sva)
  role: Empirical-Bayes batch correction applied to the intensity matrix with batch
    labels as batch covariate and cell type preserved as the biological covariate
- name: R
  role: Analysis environment for ComBat correction, per-feature two-sample t-tests,
    Benjamini-Hochberg FDR adjustment, and CSV export
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41598-020-70850-0
    title: Addressing the batch effect issue for LC/MS metabolomics data in data preprocessing
  source_content_hash: sha256:b3d98fa08545044ace1e533d322e6cb59f84105ee73da55d304b07dfd44903fc
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/tianwei-yu/apLCMS
  tools:
  - ComBat (sva)
  - R
  - apLCMS
  edam_operation: http://edamontology.org/operation_3664
  edam_topics:
  - http://edamontology.org/topic_3517
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 5
---

# figure-reproduction-from-corrected-data

## Summary

Reproduces published figure values (e.g. Fig. 3e bar counts of significant metabolic features in ST000868) by re-running the statistical test — a two-sample t-test with Benjamini-Hochberg FDR — on each preprocessing method's feature matrix after ComBat empirical-Bayes batch correction, then comparing per-setting counts to the figure.

## When to use

Use when a target figure reports per-setting counts or effect sizes (e.g. features passing FDR <= 0.2) that must be regenerated from feature-intensity matrices plus batch and phenotype annotation tables.

## When NOT to use

- The figure reports raw peak-detection or alignment performance (e.g. number of features detected) rather than downstream statistical significance counts — batch correction and t-testing are not needed
- Only a single batch is present, so ComBat empirical-Bayes between-batch correction has no batch covariate to adjust
- The target figure is about misalignment or weak-signal recovery in preprocessing itself — the paper notes post-hoc correction can only adjust signal intensity and cannot fix misaligned features

## Inputs

- Feature-intensity matrix (samples x features) for each preprocessing method/parameter setting (apLCMS feature table, XCMS feature table) on the ST000868 metabolomics dataset
- Batch annotation table mapping samples to processing batches
- Cell-type annotation table labeling the two yeast cell types being compared

## Outputs

- CSV table of significant-feature counts at FDR <= 0.2 per parameter setting and preprocessing method (fig3e_fdr_significant_counts.csv)
- Per-feature t-test p-values and Benjamini-Hochberg-adjusted FDR values on ComBat-corrected intensities

## How to apply

Load the feature-intensity matrix (apLCMS feature table) for the dataset — here ST000868 — together with the batch labels and the cell-type annotation table. Apply ComBat empirical-Bayes batch correction to the intensity matrix using batch labels as the batch covariate and cell type as the preserved biological covariate, repeating for each preprocessing method's matrix (two-stage apLCMS, traditional apLCMS, XCMS). For every feature, run a two-sample t-test comparing the two yeast cell types on the ComBat-corrected intensities, adjust p-values with Benjamini-Hochberg FDR, and count features with FDR <= 0.2 per parameter setting. The rationale, per the article, is that post-hoc batch correction adjusts signal intensity between batches and can be combined with preprocessing-stage alignment — the paper shows the two-stage apLCMS approach yielded more significant features at FDR <= 0.2 than traditional apLCMS or XCMS after ComBat. Assemble counts into a table matching the figure and write it (e.g. fig3e_fdr_significant_counts.csv).

## Related tools

- **apLCMS** (Source of the feature-intensity matrices (two-stage and traditional preprocessing pipelines) whose ComBat-corrected significant-feature counts are tabulated) — https://github.com/tianwei-yu/apLCMS
- **ComBat (sva)** (Empirical-Bayes batch correction applied to the intensity matrix with batch labels as batch covariate and cell type preserved as the biological covariate)
- **R** (Analysis environment for ComBat correction, per-feature two-sample t-tests, Benjamini-Hochberg FDR adjustment, and CSV export)

## Examples

```
library(sva); corrected <- t(ComBat(t(ftr_matrix), batch=batch_labels, mod=model.matrix(~cell_type, data=ann))); p <- apply(corrected, 2, function(x) t.test(x ~ ann$cell_type)$p.value); fdr <- p.adjust(p, method="BH")
```

## Evaluation signals

- Output table contains one significant-feature count for every preprocessing method/parameter setting shown in the figure (Fig. 3e)
- Each reported count equals the number of features whose Benjamini-Hochberg FDR on ComBat-corrected intensities is <= 0.2
- Reproduced counts match the bar values in Fig. 3e, with two-stage apLCMS yielding more significant features than traditional apLCMS and XCMS
- Same features are compared across methods only where the t-test was run on identically ComBat-corrected matrices (batch covariate = batch labels, mod preserved covariate = cell type)

## Limitations

- Post-hoc batch effect correction can only adjust signal intensity; it cannot fix feature misalignment across batches or incomplete weak-signal recovery from the original raw data
- Treating all samples as a single group in preprocessing can cause peak alignment errors that no downstream correction method can repair, so counts depend on the upstream alignment quality
- No apLCMS package version is stated in the article, and supplementary section content is unavailable, so exact software-version-dependent counts may be hard to reproduce bit-for-bit

## Evidence

- [abstract] It can be combined with down-stream batch effect correction methods to further correct for between-batch intensity differences: "It can be combined with down-stream batch effect correction methods to further correct for between-batch intensity differences"
- [intro] However, such approaches can only adjust signal intensity. They cannot address issues such as misalignment of features across batches22, or the incomplete weak signal recovery from the original data.: "However, such approaches can only adjust signal intensity. They cannot address issues such as misalignment of features across batches22, or the incomplete weak signal recovery from the original data."
- [discussion] we showed that the new method improved the consistency of feature detection and quantification: "we showed that the new method improved the consistency of feature detection and quantification"
