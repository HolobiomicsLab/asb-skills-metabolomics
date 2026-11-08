---
name: wavelet-ica-intensity-normalization
description: Use when a samples x features LC/MS intensity matrix carries batch labels
  plus a two-group annotation (e.g., cell type), and you need post-hoc WaveICA batch
  correction followed by per-feature differential testing and FDR-binned significant-feature
  counts.
when_to_use_negative:
- The goal is to fix peak misalignment or incomplete weak signal recovery after preprocessing
  — post-hoc batch correction can only adjust signal intensity and cannot correct
  alignment errors
- The data are single-batch or lack batch labels, so no between-batch intensity correction
  is applicable
- A batch-aware two-stage preprocessing method is already applied in the preprocessing
  stage and no post-hoc correction comparison (e.g., vs ComBat) is needed
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: WaveICA
  role: Wavelet-based independent component analysis method applied post hoc to correct
    between-batch intensity differences in the feature matrix before testing
- name: R
  role: Analysis environment used to run WaveICA correction, per-feature two-sample
    t-tests, Benjamini-Hochberg FDR adjustment, and count/tabulate results
- name: apLCMS
  role: Upstream platform whose workflow produced the preprocessed batch-labeled feature
    tables corrected here; its two-stage variant is the benchmark whose significant-feature
    counts are compared
  repo: https://github.com/tianwei-yu/apLCMS
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41598-020-70850-0
    title: Addressing the batch effect issue for LC/MS metabolomics data in data preprocessing
  source_content_hash: sha256:71af87bc17dea02788eb092063843ed09fe81e977f2e6e23f7a140e4f718518c
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
yes_rate: 1.0
n_claims: 9
---

# wavelet-ica-intensity-normalization

## Summary

Applies WaveICA batch-effect correction to LC/MS metabolomics feature intensity matrices (e.g., the ST000868 cell-type dataset), then evaluates detection of significant metabolic features via two-sample t-tests with Benjamini-Hochberg FDR ≤ 0.2, as in Fig. 3f of the apLCMS two-stage preprocessing paper.

## When to use

Use when a samples x features LC/MS intensity matrix carries batch labels plus a two-group annotation (e.g., cell type), and you need post-hoc WaveICA batch correction followed by per-feature differential testing and FDR-binned significant-feature counts.

## When NOT to use

- The goal is to fix peak misalignment or incomplete weak signal recovery after preprocessing — post-hoc batch correction can only adjust signal intensity and cannot correct alignment errors
- The data are single-batch or lack batch labels, so no between-batch intensity correction is applicable
- A batch-aware two-stage preprocessing method is already applied in the preprocessing stage and no post-hoc correction comparison (e.g., vs ComBat) is needed

## Inputs

- ST000868 feature intensity matrix (samples x features) with batch labels
- Cell-type group annotations for each sample
- WaveICA parameter settings to sweep (e.g., different values of the compared settings)

## Outputs

- WaveICA-corrected intensity matrices (one per parameter setting)
- fig3f_significant_feature_counts.csv: per-setting counts of features with BH FDR <= 0.2
- Bar-chart figure mirroring Fig. 3f

## How to apply

Load the preprocessed ST000868 batch-labeled feature intensity matrix (samples as rows, features as columns, with cell-type annotations) into R. Apply WaveICA batch correction to the matrix, sweeping the WaveICA parameter settings under comparison; the paper found WaveICA results were similar to ComBat after correction. For each corrected matrix, run a two-sample t-test on every feature between the cell-type groups. Adjust p-values with Benjamini-Hochberg FDR correction and count features with FDR <= 0.2 per parameter setting. Compile the per-setting counts into a summary table and bar chart mirroring Fig. 3f, saved as fig3f_significant_feature_counts.csv. Rationale: the paper's two-stage preprocessing (peak detection, RT correction, feature alignment, weak signal recovery using batch information) can be combined with down-stream batch correction methods such as WaveICA to further correct between-batch intensity differences, and the corrected tables improved downstream testing relative to traditional apLCMS and XCMS.

## Related tools

- **WaveICA** (Wavelet-based independent component analysis method applied post hoc to correct between-batch intensity differences in the feature matrix before testing)
- **R** (Analysis environment used to run WaveICA correction, per-feature two-sample t-tests, Benjamini-Hochberg FDR adjustment, and count/tabulate results)
- **apLCMS** (Upstream platform whose workflow produced the preprocessed batch-labeled feature tables corrected here; its two-stage variant is the benchmark whose significant-feature counts are compared) — https://github.com/tianwei-yu/apLCMS

## Examples

```
Rscript waveica_fdr_counts.R --input ST000868_ftrs_matrix.csv --batches batch_labels.csv --fdr 0.2 --output fig3f_significant_feature_counts.csv
```

## Evaluation signals

- Corrected matrices preserve the original samples x features dimensions of the input intensity table
- WaveICA-corrected FDR <= 0.2 counts should be broadly similar to ComBat-corrected counts on the same ST000868 matrices, per the paper's Fig. 3f comparison
- The two-stage apLCMS pipeline's significant-feature count should exceed those of traditional apLCMS and XCMS after WaveICA correction
- The output CSV matches the expected schema: one row per WaveICA parameter setting with its FDR <= 0.2 significant-feature count
- BH-adjusted p-values are monotone with raw p-values and every reported count corresponds to adjusted p <= 0.2

## Limitations

- Post-hoc batch effect removal can only adjust signal intensity; it cannot address misalignment of features across batches or incomplete weak signal recovery from the original data
- Treating all samples as a single group during preprocessing can cause peak alignment errors that cannot be corrected by post hoc application of batch effect correction methods
- The article provides no apLCMS package version, no explicit limitations of the new method, and supplementary section content was unavailable
- WaveICA results were reported as similar to ComBat rather than superior, so the choice of correction method may not change conclusions on comparable datasets

## Evidence

- [abstract] It can be combined with down-stream batch effect correction methods to further correct for between-batch intensity differences: "It can be combined with down-stream batch effect correction methods to further correct for between-batch intensity differences"
- [intro] Limitation of post hoc correction: "However, such approaches can only adjust signal intensity. They cannot address issues such as misalignment of features across batches22, or the incomplete weak signal recovery from the original data."
- [abstract] Two-stage preprocessing improves downstream results: "the new method resulted in feature tables with better consistency, as well as better down-stream analysis results"
- [discussion] apLCMS implementation and availability: "The package can be downloaded at https://github.com/tianwei-yu/apLCMS."
