---
name: quality-control-sample-correlation-analysis
description: Use when comparing batch-effect or run-order effect removal methods on
  metabolomics peak matrices that include pooled/quality control samples, and you
  need a quantitative, reference-free similarity metric to rank corrected datasets.
when_to_use_negative:
- The dataset contains no pooled quality control samples, so there are no replicate
  pairs to correlate.
- You want to assess between-batch effects on multi-batch data where batch labels
  are available — use PCA score plots, cumulative RSD curves, or batch-label-aware
  methods instead of QCS-pair PCC alone.
edam_operation: http://edamontology.org/operation_2930
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_3520
tools:
- name: R
  role: Implementation language in which QCS-pair Pearson correlation coefficients
    were computed on original and corrected metabolomics peak matrices
- name: WaveICA_2.0
  role: Batch/run-order effect removal method whose output QCS similarity is evaluated
    by this correlation analysis; R package for metabolomics data
  repo: https://github.com/dengkuistat/WaveICA_2.0
- name: QC-RLSC
  role: Comparison method (loess-style smoothing curve fitted in QCSs) evaluated against
    WaveICA 2.0 using QCS PCC
- name: QC-SVRC
  role: Comparison method (SVM regression with radial basis kernel, gamma grid 2^-3
    to 2^6 selected by ten-fold cross-validation) evaluated against WaveICA 2.0 using
    QCS PCC
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1007/s11306-021-01839-7
    title: 'WaveICA 2.0: a novel batch effect removal method for untargeted metabolomics
      data without using batch information'
  source_content_hash: sha256:ad92b543c853142b0b444aaf01ae00ffcbce0c794287ca7bf9d50bca51ecb076
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/dengkuistat/WaveICA_2.0
  tools:
  - QC-RLSC
  - QC-SVRC
  - R
  - WaveICA_2.0
  edam_operation: http://edamontology.org/operation_2930
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# quality-control-sample-correlation-analysis

## Summary

Quantifies batch/run-order effect removal in untargeted metabolomics by computing Pearson correlation coefficients (PCCs) between all pairs of quality control samples (QCSs), where PCCs approaching 1 indicate effective drift removal. Used in WaveICA 2.0 to show that only WaveICA 2.0 increased QCS similarity on single-batch data (PCC 0.973) versus QC-RLSC (0.941) and QC-SVRC (0.934).

## When to use

Use when comparing batch-effect or run-order effect removal methods on metabolomics peak matrices that include pooled/quality control samples, and you need a quantitative, reference-free similarity metric to rank corrected datasets.

## When NOT to use

- The dataset contains no pooled quality control samples, so there are no replicate pairs to correlate.
- You want to assess between-batch effects on multi-batch data where batch labels are available — use PCA score plots, cumulative RSD curves, or batch-label-aware methods instead of QCS-pair PCC alone.

## Inputs

- Corrected (or original) metabolomics peak intensity matrix with a sample-type annotation distinguishing QCSs from subject samples (e.g., batch1_comparison_results.csv with PCC and differential peak count columns)
- Quality control sample (QCS) row indices/labels within the peak matrix
- Optionally, injection order vector used to generate the corrected datasets being compared

## Outputs

- Pearson correlation coefficients between all pairs of QCSs for each corrected dataset (summary statistic such as mean PCC per method)
- Method comparison table (e.g., batch1_comparison_results.csv) ranking methods by QCS PCC alongside differential peak counts

## How to apply

Take the peak intensity matrix (e.g., 6,461 peaks from 192 subject samples and 25 QCSs in batch 1 of the plasma metabolomics dataset) for the original and each corrected dataset. Extract the QCS submatrix and compute Pearson correlation coefficients between every pair of QCSs. High PCCs approaching 1 indicate that injection-order or batch drift has been removed, since replicate QCSs should be biologically identical. Compile mean/summary PCCs per method and compare: in the paper's validation, WaveICA 2.0 yielded PCC near 0.973 versus 0.941 (QC-RLSC) and 0.934 (QC-SVRC), so only methods that raise PCC relative to the original data are considered to have efficiently removed run order effects. Pair this metric with downstream differential peak counts (PLS-DA with FDR < 0.05 and VIP > 1) to confirm biological signal is not lost.

## Related tools

- **R** (Implementation language in which QCS-pair Pearson correlation coefficients were computed on original and corrected metabolomics peak matrices)
- **WaveICA_2.0** (Batch/run-order effect removal method whose output QCS similarity is evaluated by this correlation analysis; R package for metabolomics data) — https://github.com/dengkuistat/WaveICA_2.0
- **QC-RLSC** (Comparison method (loess-style smoothing curve fitted in QCSs) evaluated against WaveICA 2.0 using QCS PCC)
- **QC-SVRC** (Comparison method (SVM regression with radial basis kernel, gamma grid 2^-3 to 2^6 selected by ten-fold cross-validation) evaluated against WaveICA 2.0 using QCS PCC)

## Examples

```
# in R, after extracting the QCS submatrix for original and corrected datasets
cors <- cor(t(qcs_batch1), method="pearson"); mean(cors[upper.tri(cors)])  # WaveICA 2.0 expected ~0.973, QC-RLSC ~0.941, QC-SVRC ~0.934
```

## Evaluation signals

- Mean PCC between QCS pairs should increase after effective batch/run-order effect removal and approach 1; WaveICA 2.0 on batch 1 should yield PCC near 0.973.
- Methods that fail to remove run order effects (QC-RLSC 0.941, QC-SVRC 0.934 on batch 1) show lower PCC than WaveICA 2.0; only WaveICA 2.0 increased QCS similarity in the paper's single-batch comparison.
- The correlation analysis should be run identically on the original and all corrected datasets so PCC values are directly comparable.
- Cross-check with differential peak counts under PLS-DA (FDR < 0.05, VIP > 1): WaveICA 2.0 gave 172 differential peaks versus 56 (QC-RLSC) and 68 (QC-SVRC), and improved SVM AUC (0.8831 ± 0.0170).

## Limitations

- PCC between QCSs alone does not confirm that biological variation among subject samples is preserved; pair it with downstream biomarker discovery (PLS-DA differential peaks, SVM five-fold cross-validated AUC).
- The comparison in this paper used batch 1 of a plasma metabolomics dataset (192 subjects, 25 QCSs, 6,461 peaks); results may not generalize to data without QCSs or with different drift structure.
- On multi-batch data, WaveICA 2.0's average QCS distance in PCA was slightly longer than batch-label-aware WaveICA, so PCC comparisons should be interpreted per design (batch-aware vs. batch-unaware).

## Evidence

- [results] only WaveICA 2.0 efficiently improved the Pearson correlation coefficients between pairs of QCSs: [evidence span withheld — anchor status: not_found]
- [supplementary] Compute Pearson correlation coefficients between all pairs of QCSs: [evidence span withheld — anchor status: not_found]
