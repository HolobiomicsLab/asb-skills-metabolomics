---
name: pca-quality-control-tightness-evaluation
description: Use when a corrected metabolomics feature matrix (e.g., post-WaveICA
  or WaveICA 2.0) must be scored for batch effect removal quality and pooled QCS injections
  are available to benchmark clustering tightness.
when_to_use_negative:
- The dataset contains no pooled quality control sample (QCS) injections, so QCS gathering
  tightness and pairwise QCS PCCs cannot be computed.
- The input is raw MS data (e.g., .d or mzXML files) rather than a preprocessed peak
  intensity matrix from XCMS/CAMERA.
- The goal is batch effect correction itself rather than evaluation of an already-corrected
  dataset.
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: R
  role: Environment in which the PCA score plots, average QCS/subject-sample distances,
    and pairwise QCS Pearson correlations are computed
- name: WaveICA_2.0
  role: Provides the Amide_data feature matrix, injection order, and batch labels
    used to generate corrected datasets for the QCS tightness evaluation
  repo: https://github.com/dengkuistat/WaveICA_2.0
- name: MetaboAnalystR
  role: Metabolomics pipeline into which WaveICA is incorporated, supporting downstream
    PCA-based quality assessment of corrected data
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1007/s11306-021-01839-7
    title: 'WaveICA 2.0: a novel batch effect removal method for untargeted metabolomics
      data without using batch information'
  source_content_hash: sha256:5f2db89dbdb663f6b6b4a61909ec550b62943252fe0b8535e72a344a8cdb552b
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/dengkuistat/WaveICA_2.0
  tools:
  - MetaboAnalystR
  - R
  - WaveICA_2.0
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# pca-quality-control-tightness-evaluation

## Summary

Evaluates batch effect removal in untargeted metabolomics by measuring how tightly quality control samples (QCSs) gather in PCA score plots, using the average distance of QCSs and subject samples plus the average Pearson correlation coefficient (aPCC) between QCS pairs.

## When to use

Use when a corrected metabolomics feature matrix (e.g., post-WaveICA or WaveICA 2.0) must be scored for batch effect removal quality and pooled QCS injections are available to benchmark clustering tightness.

## When NOT to use

- The dataset contains no pooled quality control sample (QCS) injections, so QCS gathering tightness and pairwise QCS PCCs cannot be computed.
- The input is raw MS data (e.g., .d or mzXML files) rather than a preprocessed peak intensity matrix from XCMS/CAMERA.
- The goal is batch effect correction itself rather than evaluation of an already-corrected dataset.

## Inputs

- Feature matrix of metabolite peak intensities (samples x peaks, e.g., 642 samples x 6,461 peaks) before batch effect removal
- Corrected feature matrix after applying a batch effect removal method (WaveICA, WaveICA 2.0, QC-RLSC, QC-SVRC)
- Sample annotation marking which injections are QCSs vs subject samples (e.g., 25/25/24 QCSs across three batches)

## Outputs

- Average distance of QCSs in the PCA score plot
- Average distance of subject samples in the PCA score plot
- Average Pearson correlation coefficient (aPCC) between pairs of QCSs
- Results table compiling the metrics per method (e.g., results_table.csv rows such as 'WaveICA 2.0 9.154 34.261 0.973 ...')

## How to apply

Load the corrected feature matrix (e.g., the Amide_data matrix of 642 samples and 6,461 metabolite peaks with 74 QCSs across three batches) and QCS/subject sample labels. Compute a PCA score plot for the samples and, per the article's protocol, calculate the average distance among QCSs and among subject samples in the score plot, plus the average Pearson correlation coefficient between all pairs of QCSs. Run the same metrics on the original (uncorrected) data so the reduction in QCS distance and rise in aPCC quantify batch effect removal; a well-corrected dataset drives pairwise QCS PCCs toward 1. Compile the metrics for each condition (original, WaveICA 2.0, WaveICA, and optionally QC-RLSC/QC-SVRC) into a results table to compare methods side by side.

## Related tools

- **R** (Environment in which the PCA score plots, average QCS/subject-sample distances, and pairwise QCS Pearson correlations are computed)
- **WaveICA_2.0** (Provides the Amide_data feature matrix, injection order, and batch labels used to generate corrected datasets for the QCS tightness evaluation) — https://github.com/dengkuistat/WaveICA_2.0
- **MetaboAnalystR** (Metabolomics pipeline into which WaveICA is incorporated, supporting downstream PCA-based quality assessment of corrected data)

## Examples

```
aPCC <- mean(cor(t(qcs_corrected)))  # average Pearson correlation between pairs of QCSs, per article step 3
```

## Evaluation signals

- Pairwise QCS Pearson correlation coefficients should approach 1 after effective correction (article reports PCCs between any two QCSs approaching 1).
- Average QCS distance in the PCA score plot should be substantially reduced versus the original data (e.g., WaveICA 2.0 at 9.154 vs a much larger original value).
- Average distance of subject samples should decrease and be similar between WaveICA 2.0 and WaveICA, since batch labels are not used by WaveICA 2.0.
- Compiled metrics should reproduce the Table 1/supplementary pattern, e.g., row 'WaveICA 2.0 9.154 34.261 0.973 172 0.8831 ± 0.0170'.
- The corrected-condition aPCC should exceed the aPCC of competing correction methods (e.g., QC-RLSC 0.941 and QC-SVRC 0.934 vs WaveICA 2.0 0.973).

## Limitations

- The average distance in QCSs for batch-label-dependent WaveICA was slightly shorter than that for WaveICA 2.0, so this metric alone can favor methods that use batch information.
- Metric ordering does not always track downstream utility: a method with a lower aPCC (e.g., QC-SVRC, 0.8476 ± 0.0203 AUC) can still be compared only in combination with biomarker count and SVM cross-validation AUC.
- The evaluation depends on pooled QCS injections being present in every batch; it was validated on a single three-batch plasma dataset (568 patients, 6,461 peaks) from the WaveICA_2.0 package.

## Evidence
