---
name: lc-ms-batch-effect-correction
description: Use when multi-batch LC/MS metabolomics raw files must be preprocessed
  so that between-batch differences do not corrupt peak detection, RT correction,
  feature alignment, or weak signal recovery, and when post-hoc correction will be
  applied to intensities.
when_to_use_negative:
- Data are from a single batch with no batch structure, so batch-aware preprocessing
  and correction have nothing to adjust
- You already have an aligned feature table and only need intensity adjustment — post-hoc
  correction can adjust signal intensity but cannot fix misalignment or incomplete
  weak signal recovery
- You cannot reprocess raw data and only have a legacy feature table whose alignment
  errors were introduced by single-group preprocessing
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: apLCMS
  role: Implements the batch-aware two-stage preprocessing (peak detection, RT correction,
    alignment, weak signal recovery) via cdf.to.ftr
  repo: https://github.com/tianwei-yu/apLCMS
- name: R
  role: Environment in which the apLCMS package runs and where downstream WaveICA/ComBat
    correction, t-tests, and FDR counting are performed
- name: WaveICA
  role: Downstream batch effect correction applied to the feature intensity matrices
    before differential testing (combined with ComBat comparison)
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41598-020-70850-0
    title: Addressing the batch effect issue for LC/MS metabolomics data in data preprocessing
  source_content_hash: sha256:afaaeffdc6a2aaac425582cd5d6bbdaa441787fe348ded2cac32cbbdf0f272aa
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/tianwei-yu/apLCMS
  tools:
  - R
  - WaveICA
  - apLCMS
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# lc-ms-batch-effect-correction

## Summary

A two-stage LC/MS metabolomics preprocessing skill that addresses batch effects at the peak detection/alignment stage (via batch-aware apLCMS) and can be combined with downstream batch effect correction (e.g., WaveICA, ComBat) for between-batch intensity differences.

## When to use

Use when multi-batch LC/MS metabolomics raw files must be preprocessed so that between-batch differences do not corrupt peak detection, RT correction, feature alignment, or weak signal recovery, and when post-hoc correction will be applied to intensities.

## When NOT to use

- Data are from a single batch with no batch structure, so batch-aware preprocessing and correction have nothing to adjust
- You already have an aligned feature table and only need intensity adjustment — post-hoc correction can adjust signal intensity but cannot fix misalignment or incomplete weak signal recovery
- You cannot reprocess raw data and only have a legacy feature table whose alignment errors were introduced by single-group preprocessing

## Inputs

- Raw LC/MS data files (e.g., NetCDF/cdf) organized in a folder with batch labels per sample
- apLCMS parameters: min.exp, min.pres, min.run, mz.tol, baseline.correct.noise.percentile, shape.model
- Sample-level batch annotations and group labels (e.g., cell-type annotations for ST000868)

## Outputs

- Batch-aware sample-by-feature intensity matrix (feature table)
- Per-feature two-sample t-test p-values with Benjamini-Hochberg FDR-adjusted values
- Summary table of significant feature counts at FDR <= 0.2 (e.g., fig3f_significant_feature_counts.csv) and comparison bar-chart

## How to apply

Run the two-stage apLCMS workflow: first preprocess each batch separately with cdf.to.ftr (proc.cdf/adaptive.bin for peak detection, RT correction via find.tol.time, within-batch alignment), then align features across batches with cumulative nonlinear RT correction and recover weak signals across batches. Rationale: treating all samples as a single group causes alignment errors that post-hoc correction cannot fix, while batch-aware preprocessing improves feature consistency before intensity adjustment. After building the batch-labeled sample-by-feature intensity matrix, optionally apply downstream correction such as WaveICA (sweeping its parameters) or ComBat. Finally, evaluate by two-sample t-tests per feature, Benjamini-Hochberg FDR adjustment, and counting features with FDR <= 0.2 to confirm the two-stage pipeline detects more significant features than traditional apLCMS/XCMS.

## Related tools

- **apLCMS** (Implements the batch-aware two-stage preprocessing (peak detection, RT correction, alignment, weak signal recovery) via cdf.to.ftr) — https://github.com/tianwei-yu/apLCMS
- **R** (Environment in which the apLCMS package runs and where downstream WaveICA/ComBat correction, t-tests, and FDR counting are performed)
- **WaveICA** (Downstream batch effect correction applied to the feature intensity matrices before differential testing (combined with ComBat comparison))

## Examples

```
library(apLCMS); ftrs <- cdf.to.ftr(folder='raw/', file.pattern='\.cdf', n.nodes=4, min.exp=10, min.pres=0.5, min.run=12, mz.tol=1e-5)
```

## Evaluation signals

- Two-stage preprocessing yields feature tables with better consistency of feature detection and quantification across batches than single-group preprocessing
- After WaveICA or ComBat correction, the two-stage approach detects more significant features at FDR <= 0.2 in group t-tests than traditional apLCMS and XCMS preprocessing
- Peak alignment errors present in single-group preprocessing are absent — features are correctly matched across batches before any intensity adjustment
- Weak signals recovered in the cross-batch recovery step appear in the final feature table rather than being dropped after preprocessing

## Limitations

- Post-hoc batch effect correction applied after traditional preprocessing can only adjust signal intensity; it cannot fix misalignment of features across batches or incomplete weak signal recovery from the original data
- Treating all samples as a single batch can result in peak alignment errors that cannot be corrected by post hoc application of batch effect correction methods
- The apLCMS method requires batch information, and no specific package version is stated in the article

## Evidence

- [abstract] Two-stage preprocessing improves peak steps before intensity correction: "we developed a new approach that address the batch effect issue in the preprocessing stage, resulting in better peak detection, alignment and quantification."
- [abstract] Post-hoc correction can be combined: "It can be combined with down-stream batch effect correction methods to further correct for between-batch intensity differences"
- [intro] Limitation of post-hoc-only correction: "They cannot address issues such as misalignment of features across batches22, or the incomplete weak signal recovery from the original data."
- [methods] Entry-point function for raw data processing: "`cdf.to.ftr(folder, file.pattern, n.nodes, min.exp, min.pres, min.run, mz.tol, baseline.correct.noise.percentile, shape.model"
