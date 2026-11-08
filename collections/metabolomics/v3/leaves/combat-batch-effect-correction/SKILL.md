---
name: combat-batch-effect-correction
description: Use when a feature-intensity matrix from multi-batch LC/MS metabolomics
  data (samples processed in batches) shows between-batch intensity differences that
  confound two-group differential testing, and you need batch-corrected intensities
  before per-feature t-tests.
when_to_use_negative:
- The input is already a batch-corrected matrix or the raw data were processed with
  the article's batch-aware two-stage preprocessing alone without downstream intensity
  correction
- Samples were all acquired in a single batch — between-batch intensity differences
  the method targets do not exist
- The goal is to fix peak misalignment or incomplete weak signal recovery; ComBat
  can only adjust signal intensity and cannot correct alignment errors post hoc
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: ComBat (sva)
  role: Empirical-Bayes batch correction applied to the feature-intensity matrix using
    batch labels as the batch covariate and cell type as the preserved biological
    covariate
- name: R
  role: Runtime environment for running ComBat, per-feature two-sample t-tests, and
    Benjamini-Hochberg FDR adjustment
- name: apLCMS
  role: Upstream producer of the feature-intensity matrix (two-stage batch-aware and
    traditional modes) whose ComBat-corrected outputs are compared
  repo: https://github.com/tianwei-yu/apLCMS
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41598-020-70850-0
    title: Addressing the batch effect issue for LC/MS metabolomics data in data preprocessing
  source_content_hash: sha256:73114746eb83c3d474730c8268a3e77bdcc83b07c9c7c74c844d9e529ac44011
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/tianwei-yu/apLCMS
  tools:
  - ComBat (sva)
  - R
  - apLCMS
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 5
---

# combat-batch-effect-correction

## Summary

Applies ComBat (sva) empirical-Bayes batch correction to LC/MS metabolomics feature-intensity matrices (e.g. from apLCMS two-stage or XCMS preprocessing on ST000868) so that between-batch intensity differences are removed while a biological covariate such as yeast cell type is preserved, enabling fair comparison of significant-feature counts at FDR <= 0.2 across preprocessing methods.

## When to use

Use when a feature-intensity matrix from multi-batch LC/MS metabolomics data (samples processed in batches) shows between-batch intensity differences that confound two-group differential testing, and you need batch-corrected intensities before per-feature t-tests.

## When NOT to use

- The input is already a batch-corrected matrix or the raw data were processed with the article's batch-aware two-stage preprocessing alone without downstream intensity correction
- Samples were all acquired in a single batch — between-batch intensity differences the method targets do not exist
- The goal is to fix peak misalignment or incomplete weak signal recovery; ComBat can only adjust signal intensity and cannot correct alignment errors post hoc

## Inputs

- Feature-intensity matrix (features x samples) from apLCMS two-stage apLCMS, traditional apLCMS, or XCMS preprocessing of LC/MS data (e.g. ST000868)
- Batch annotation table mapping each sample to its acquisition batch
- Biological covariate table (e.g. the two yeast cell-type labels per sample)

## Outputs

- ComBat-corrected intensity matrix with the same features and samples
- Per-feature two-sample t-test p-values and Benjamini-Hochberg FDR values
- Table of significant-feature counts at FDR <= 0.2 per parameter setting (e.g. fig3e_fdr_significant_counts.csv reproducing Fig. 3e)

## How to apply

Load the feature-intensity matrix (e.g. apLCMS cdf.to.ftr output) together with the batch annotation table and the biological covariate table (e.g. cell type). Transpose so features are rows and samples are columns, then call ComBat with the batch labels as the batch covariate and a model matrix built from the biological covariate to preserve it. Because post-hoc correction can only adjust signal intensity — it cannot fix peak misalignment or incomplete weak signal recovery from preprocessing — apply ComBat downstream of, not instead of, batch-aware preprocessing. On the corrected matrix, run a two-sample t-test per feature comparing the two groups, adjust p-values with Benjamini-Hochberg FDR, and count features with FDR <= 0.2 per parameter setting. Rationale: the article shows batch correction at the preprocessing stage improves consistency, and ComBat further corrects between-batch intensity differences to yield better downstream analysis results.

## Related tools

- **ComBat (sva)** (Empirical-Bayes batch correction applied to the feature-intensity matrix using batch labels as the batch covariate and cell type as the preserved biological covariate)
- **R** (Runtime environment for running ComBat, per-feature two-sample t-tests, and Benjamini-Hochberg FDR adjustment)
- **apLCMS** (Upstream producer of the feature-intensity matrix (two-stage batch-aware and traditional modes) whose ComBat-corrected outputs are compared) — https://github.com/tianwei-yu/apLCMS

## Examples

```
library(sva); corrected <- ComBat(t(ftr_matrix), batch = batch_labels, mod = model.matrix(~cell_type, data = annot))
```

## Evaluation signals

- The output table contains a significant-feature count for every preprocessing parameter setting, with no missing settings
- Each reported count equals the number of features with Benjamini-Hochberg FDR <= 0.2 on the ComBat-corrected intensities, matching the bar values in Fig. 3e
- The ComBat call used batch labels as the batch covariate and preserved the biological covariate (cell type) via the mod argument
- Reproduced counts show the two-stage apLCMS approach yielding more significant features at FDR <= 0.2 than traditional apLCMS and XCMS
- Corrected matrix has the same dimensions (features x samples) as the input matrix

## Limitations

- Post-hoc batch effect correction can only adjust signal intensity; it cannot address misalignment of features across batches or incomplete weak signal recovery from the original data
- Traditional preprocessing that treats all samples as a single group can produce peak alignment errors that cannot be corrected by post hoc application of batch effect correction methods
- No apLCMS package version is stated in the article, so reproducibility of the upstream feature table depends on the package commit used

## Evidence

- [abstract] ComBat as downstream intensity correction: "It can be combined with down-stream batch effect correction methods to further correct for between-batch intensity differences"
- [intro] Scope limitation of post-hoc correction: "However, such approaches can only adjust signal intensity. They cannot address issues such as misalignment of features across batches22, or the incomplete weak signal"
- [intro] Multi-batch setting motivating correction: "When the sample size is large, it is often necessary for the samples to be processed in batches."
- [abstract] Downstream benefit of correcting batch effects: "the new method resulted in feature tables with better consistency, as well as better down-stream analysis results"
