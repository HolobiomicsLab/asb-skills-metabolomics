---
name: metabolomics-significant-feature-counting
description: Use when you have an LC/MS feature-intensity matrix plus batch and phenotype
  annotations and need per-preprocessing-setting counts of differentially abundant
  features at FDR <= 0.2 after empirical-Bayes batch correction.
when_to_use_negative:
- Input is raw LC/MS data that still needs peak detection, RT correction, and feature
  alignment (use the apLCMS two-stage preprocessing pipeline such as cdf.to.ftr first)
- Goal is to fix feature misalignment or incomplete weak signal recovery across batches
  — the article notes batch correction 'can only adjust signal intensity. They cannot
  address issues such as misalignment of features across batches'
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: ComBat (sva)
  role: Empirical-Bayes batch correction of the feature-intensity matrix, preserving
    cell type as the biological covariate
- name: R
  role: Runtime environment for ComBat correction, two-sample t-tests, Benjamini-Hochberg
    FDR adjustment, and count table assembly
- name: apLCMS
  role: Produces the feature tables (traditional and two-stage) whose significant-feature
    counts are compared; the two-stage approach is implemented in the apLCMS workflow
  repo: https://github.com/tianwei-yu/apLCMS
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41598-020-70850-0
    title: Addressing the batch effect issue for LC/MS metabolomics data in data preprocessing
  source_content_hash: sha256:224750eba8d853af546ddf7b7e6552f4c5a240070917d01f348a6a9f593c1bc9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/tianwei-yu/apLCMS
  tools:
  - ComBat (sva)
  - R
  - apLCMS
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 4
---

# metabolomics-significant-feature-counting

## Summary

Counts significant LC/MS metabolic features between two biological groups (e.g., yeast cell types in ST000868) by running two-sample t-tests on ComBat batch-corrected feature-intensity matrices and tallying features passing Benjamini-Hochberg FDR <= 0.2 per preprocessing parameter setting, as in Fig. 3e of the apLCMS batch-effect paper.

## When to use

Use when you have an LC/MS feature-intensity matrix plus batch and phenotype annotations and need per-preprocessing-setting counts of differentially abundant features at FDR <= 0.2 after empirical-Bayes batch correction.

## When NOT to use

- Input is raw LC/MS data that still needs peak detection, RT correction, and feature alignment (use the apLCMS two-stage preprocessing pipeline such as cdf.to.ftr first)
- Goal is to fix feature misalignment or incomplete weak signal recovery across batches — the article notes batch correction 'can only adjust signal intensity. They cannot address issues such as misalignment of features across batches'

## Inputs

- LC/MS feature-intensity matrix (samples x features) from apLCMS or XCMS preprocessing of the ST000868 dataset
- Sample-to-batch annotation table (batch labels used as the ComBat batch covariate)
- Sample-to-cell-type annotation table (preserved biological covariate and t-test grouping variable)

## Outputs

- Per-parameter-setting table of significant-feature counts at FDR <= 0.2 (reproducing Fig. 3e), written to fig3e_fdr_significant_counts.csv
- Intermediate per-feature t-test p-values and Benjamini-Hochberg-adjusted FDR values on ComBat-corrected intensities

## How to apply

Load the feature-intensity matrix (e.g., produced by apLCMS via cdf.to.ftr) together with batch labels and cell-type annotations into R. Apply ComBat from the sva package using batch labels as the batch covariate and cell type as the preserved biological covariate, so between-batch intensity differences are adjusted without erasing the biology of interest. For each feature, run a two-sample t-test comparing the two cell types on the ComBat-corrected intensities, then adjust p-values with Benjamini-Hochberg FDR correction and count features with FDR <= 0.2. Repeat for each preprocessing method/parameter setting (e.g., traditional apLCMS, two-stage apLCMS, XCMS) so the per-setting counts can be assembled into a table mirroring the bar values of Fig. 3e. The rationale, per the article, is that batch-aware preprocessing tables 'can be combined with down-stream batch effect correction methods to further correct for between-batch intensity differences', yielding better downstream analysis results.

## Related tools

- **ComBat (sva)** (Empirical-Bayes batch correction of the feature-intensity matrix, preserving cell type as the biological covariate)
- **R** (Runtime environment for ComBat correction, two-sample t-tests, Benjamini-Hochberg FDR adjustment, and count table assembly)
- **apLCMS** (Produces the feature tables (traditional and two-stage) whose significant-feature counts are compared; the two-stage approach is implemented in the apLCMS workflow) — https://github.com/tianwei-yu/apLCMS

## Examples

```
library(sva); combat_ftrs <- ComBat(t(ftr_matrix), batch=batch_labels, mod=model.matrix(~cell_type)); pvals <- apply(combat_ftrs, 1, function(x) t.test(x ~ cell_type)$p.value)
```

## Evaluation signals

- The output table contains a significant-feature count for every preprocessing parameter setting represented in the input matrices
- Each reported count equals the exact number of features with Benjamini-Hochberg FDR <= 0.2 on ComBat-corrected intensities
- Counts are reproducible when ComBat is rerun with the same batch covariate and cell type as the preserved biological covariate
- The assembled per-setting counts match the relative ordering in Fig. 3e, where two-stage apLCMS yields more significant features than traditional apLCMS and XCMS

## Limitations

- Post hoc batch correction such as ComBat can only adjust signal intensity; it cannot fix peak misalignment or incomplete weak signal recovery introduced during preprocessing
- Preprocessing all samples as a single group can cause peak alignment errors that no downstream batch correction can repair, so results depend on upstream preprocessing quality
- Counts are dataset- and threshold-specific (ST000868, FDR <= 0.2); they do not generalize directly to other datasets or cutoffs

## Evidence

- [abstract] Batch correction combined with downstream testing: "It can be combined with down-stream batch effect correction methods to further correct for between-batch intensity differences"
- [abstract] Downstream results quality: "the new method resulted in feature tables with better consistency, as well as better down-stream analysis results"
- [intro] Post hoc correction cannot fix alignment: "However, such approaches can only adjust signal intensity. They cannot address issues such as misalignment of features across batches"
- [methods] Feature table generation entry point: "cdf.to.ftr(folder, file.pattern, n.nodes, min.exp, min.pres, min.run, mz.tol, baseline.correct.noise.percentile, shape.model"
