---
name: plsda-differential-peak-identification
description: Use when you have a batch-corrected metabolomics peak table (e.g., 6,461
  peaks from XCMS/CAMERA) with two-class labels (CRC vs CE) and need to call differential
  peaks and estimate their predictive AUC.
when_to_use_negative:
- Input is raw LC-MS data (.d files) that has not yet been converted to mzXML and
  peak-picked with XCMS/CAMERA — the PLS-DA model needs a processed intensity matrix
- The peak table has not undergone batch/run-order effect removal — differential peak
  counts collapse dramatically (56–68 peaks vs 172), so biomarker calling would be
  dominated by technical variation
- The goal is batch effect removal itself rather than downstream biomarker identification
  — use WaveICA 2.0 / QC-RLSC / QC-SVRC first
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: R
  role: Environment in which PLS-DA biomarker identification, FDR correction, and
    the repeated SVM cross-validation are implemented
- name: WaveICA_2.0
  role: Produces the batch-effect-removed peak intensity matrix that serves as input
    to PLS-DA differential peak identification
  repo: https://github.com/dengkuistat/WaveICA_2.0
- name: MetaboAnalystR
  role: Metabolomics data analysis pipeline context in which the WaveICA-based biomarker
    identification workflow operates
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1007/s11306-021-01839-7
    title: 'WaveICA 2.0: a novel batch effect removal method for untargeted metabolomics
      data without using batch information'
  source_content_hash: sha256:0952e7c6f9161cfd9d8b3e98e9061dab51ed7ba07d4057d57761f08ac6446e30
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/dengkuistat/WaveICA_2.0
  tools:
  - MetaboAnalystR
  - R
  - WaveICA_2.0
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# plsda-differential-peak-identification

## Summary

Identifies differential metabolite peaks between two clinical groups (here CRC vs CE patients) from an LC-MS peak intensity matrix using PLS-DA with FDR < 0.05 and VIP > 1 selection, then validates the resulting biomarker panel by repeated five-fold cross-validated SVM AUC.

## When to use

Use when you have a batch-corrected metabolomics peak table (e.g., 6,461 peaks from XCMS/CAMERA) with two-class labels (CRC vs CE) and need to call differential peaks and estimate their predictive AUC.

## When NOT to use

- Input is raw LC-MS data (.d files) that has not yet been converted to mzXML and peak-picked with XCMS/CAMERA — the PLS-DA model needs a processed intensity matrix
- The peak table has not undergone batch/run-order effect removal — differential peak counts collapse dramatically (56–68 peaks vs 172), so biomarker calling would be dominated by technical variation
- The goal is batch effect removal itself rather than downstream biomarker identification — use WaveICA 2.0 / QC-RLSC / QC-SVRC first

## Inputs

- Batch-corrected peak intensity matrix (samples x peaks; e.g., 6,461 peaks) with retention time, m/z, and intensity, produced after WaveICA 2.0 / QC-RLSC / QC-SVRC correction
- Class labels for subject samples (CRC vs CE patient groups)
- Corrected QCS/subject sample annotations used to verify correction quality before biomarker calling

## Outputs

- List of differential peaks passing FDR < 0.05 and VIP > 1 (potential biomarkers) with per-peak statistics
- Count of identified differential peaks per correction method (e.g., 172 for WaveICA 2.0, 56 for QC-RLSC, 68 for QC-SVRC on batch 1)
- SVM five-fold cross-validation AUC (mean ± SD over 100 repeats) quantifying biomarker predictive ability
- Results table compiling PCCs and differential peak counts per method (e.g., batch1_comparison_results.csv)

## How to apply

Take the batch-effect-removed peak intensity matrix (retention time, m/z, intensity per sample) as input — in this study a plasma metabolomics table of 6,461 peaks from 568 patients (497 CRC, 71 CE). Fit a PLS-DA discriminant model between the two disease groups and, for each peak, compute an FDR-adjusted p-value and its variable importance in projection (VIP). Retain peaks with FDR < 0.05 AND VIP > 1 as potential biomarkers; both criteria are required, and the number of retained peaks is sensitive to the quality of the upstream batch correction (172 peaks after WaveICA 2.0 vs 56 for QC-RLSC and 68 for QC-SVRC on batch 1). Finally, evaluate the identified biomarkers' predictive ability using five-fold cross-validation of an SVM with radial basis function, repeated 100 times, and report the AUC (e.g., 0.8831 ± 0.0170 after WaveICA 2.0 correction).

## Related tools

- **R** (Environment in which PLS-DA biomarker identification, FDR correction, and the repeated SVM cross-validation are implemented)
- **WaveICA_2.0** (Produces the batch-effect-removed peak intensity matrix that serves as input to PLS-DA differential peak identification) — https://github.com/dengkuistat/WaveICA_2.0
- **MetaboAnalystR** (Metabolomics data analysis pipeline context in which the WaveICA-based biomarker identification workflow operates)

## Evaluation signals

- Every reported biomarker satisfies both thresholds: FDR < 0.05 and PLS-DA VIP > 1
- Differential peak count on batch 1 after WaveICA 2.0 correction should be 172, versus 56 (QC-RLSC) and 68 (QC-SVRC); only WaveICA 2.0 more than doubles baseline counts
- SVM five-fold cross-validation AUC after WaveICA 2.0 should be ~0.8831 ± 0.0170, above QC-RLSC (0.8585 ± 0.0201) and QC-SVRC (0.8476 ± 0.0203)
- QCS Pearson correlation coefficients should approach 1 (≈0.973 for WaveICA 2.0) before biomarker calling, confirming correction quality
- SVM evaluation must be repeated (100 times) so AUC is reported as mean ± SD rather than a single split

## Limitations

- The FDR < 0.05 / VIP > 1 thresholds are the study-specific defaults; changing them alters biomarker counts directly
- Biomarker yield is contingent on upstream batch correction quality — with weaker correction (QC-RLSC, QC-SVRC) far fewer differential peaks are recovered
- Predictive AUC estimates depend on the repeated (100x) five-fold cross-validation scheme; a single split is not stable

## Evidence

- [methods] FDR/VIP thresholds: [evidence span withheld — anchor status: not_found]
- [methods] PLS-DA + FDR identification step: [evidence span withheld — anchor status: not_found]
- [methods] SVM cross-validation evaluation: [evidence span withheld — anchor status: not_found]
