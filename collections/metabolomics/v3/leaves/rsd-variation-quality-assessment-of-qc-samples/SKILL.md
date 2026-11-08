---
name: rsd-variation-quality-assessment-of-qc-samples
description: Use when pooled QC samples were injected periodically (e.g. every 5–8
  samples) during an LC-MS metabolomics or SRM proteomics run and you need to quantify
  precision gains of a drift-correction method via QC RSD distributions.
when_to_use_negative:
- Data contain no repeated pooled-QC injections (QC-free design); use shiftCor_dQC
  with Combat instead of QC-RFSC and QC-RSD assessment.
- Feature intensities have already been filtered/normalized downstream (e.g. via statAnalysis
  Pareto scaling and glog) — RSDs computed there no longer reflect raw instrument
  precision.
- Assessing biological group separation rather than measurement precision; RSD of
  QC samples is not the appropriate metric.
edam_operation: http://edamontology.org/operation_3218
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_3520
tools:
- name: statTarget
  role: Runs shiftCor/statAnalysis to produce the raw and corrected matrices and exports
    the RSD/CV distributions and loplot QC images used for this assessment
  repo: github.com/13479776/statTarget
- name: shiftCor
  role: Applies QC-RFSC drift correction (Frule = 0.8, imputeM = 'KNN', ntree = 500)
    whose effect is then quantified by QC-RSD distributions
- name: statAnalysis
  role: Filters low-quality peaks (Frule = 0.8) and provides the QC quality-control
    statistics and plots supporting the RSD evaluation
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.aca.2018.08.002
    title: 'statTarget: A streamlined tool for signal drift correction and interpretations
      of quantitative mass spectrometry-based omics data'
  source_content_hash: sha256:a71596e979899fce7fd6ae7bede4c077c1628a6e57a1217735e2a9fcc85bef7b
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - shiftCor
  - statAnalysis
  - statTarget
  - transX
  edam_operation: http://edamontology.org/operation_3218
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# RSD variation quality assessment of QC samples

## Summary

Evaluates the relative standard deviation (RSD/CV) distribution of pooled-QC peak intensities before and after signal drift correction, using the shift in the fraction of QC features below a 30 % (or 15 %) RSD cutoff to judge whether statTarget's QC-RFSC correction improved data precision.

## When to use

Use when pooled QC samples were injected periodically (e.g. every 5–8 samples) during an LC-MS metabolomics or SRM proteomics run and you need to quantify precision gains of a drift-correction method via QC RSD distributions.

## When NOT to use

- Data contain no repeated pooled-QC injections (QC-free design); use shiftCor_dQC with Combat instead of QC-RFSC and QC-RSD assessment.
- Feature intensities have already been filtered/normalized downstream (e.g. via statAnalysis Pareto scaling and glog) — RSDs computed there no longer reflect raw instrument precision.
- Assessing biological group separation rather than measurement precision; RSD of QC samples is not the appropriate metric.

## Inputs

- Raw peak-intensity profile matrix from XCMS, MZmine2, SIEVE, or Skyline (converted with statTarget transX if needed)
- Corrected (QC-RFSC or QC-RLSC/QC-SVR/Combat) peak matrix from statTarget shiftCor
- Meta file assigning pooled QC sample names (tagged 'QC') to batch and injection order

## Outputs

- Cumulative RSD/CV distribution of raw versus corrected QC peaks, per batch and overall
- Proportion of QC peaks within 15 % and 30 % RSD cutoffs before and after correction
- Per-feature loplot quality-control images (statTarget plot = TRUE)

## How to apply

Compute the per-feature RSD (CV) of pooled-QC sample intensities across the run, both on the raw profile matrix and on the shiftCor-corrected matrix, then build cumulative RSD distribution plots for QCs per batch and overall. Judge correction success by the rise in the proportion of QC peaks below fixed RSD cutoffs — in the article's Dataset 1 the fraction of peaks with RSD < 30 % increased from 43.9 % (raw) to 86.2 % after QC-RFSC, and peaks within 15 % RSDs rose from 1.97 % to 25.1 % (a 12.7-fold increase) — while comparing against alternative correctors such as QC-RLSC (65.5 %) and QC-SVR (61.9 %). statTarget's statAnalysis/shiftCor output supports this by exporting RSD/CV distributions of raw versus corrected peaks and per-feature loplot QC images (set plot = TRUE), so the corrected matrix is accepted when the QC RSD distribution markedly improves rather than by inspecting individual features.

## Related tools

- **statTarget** (Runs shiftCor/statAnalysis to produce the raw and corrected matrices and exports the RSD/CV distributions and loplot QC images used for this assessment) — github.com/13479776/statTarget
- **shiftCor** (Applies QC-RFSC drift correction (Frule = 0.8, imputeM = 'KNN', ntree = 500) whose effect is then quantified by QC-RSD distributions)
- **statAnalysis** (Filters low-quality peaks (Frule = 0.8) and provides the QC quality-control statistics and plots supporting the RSD evaluation)

## Examples

```
R> library(statTarget); shiftCor(samPeno, samFile, Frule = 0.8, MLmethod = "QCRFSC", QCspan = 0, imputeM = "KNN")
```

## Evaluation signals

- Cumulative RSD distribution of corrected QC peaks should shift leftward relative to raw data; a failure mode is little or no increase in the proportion of peaks with RSD < 30 %.
- Report the fraction of QC peaks within the 15 % RSD cutoff before and after correction; article-level sanity benchmarks: 1.97 % → 25.1 % (Dataset 1) and ~52.1 % → 90.9 % (Dataset 2).
- Compare against QC-RLSC and QC-SVR baselines; QC-RFSC is considered superior when its corrected distribution exceeds theirs (65.5 % and 61.9 % within 30 % RSDs in Dataset 1).
- Verify the 80 % rule (Frule = 0.8) and KNN imputation were applied before correction; low-quality features passing neither filter should be excluded from the RSD summary.
- Check that per-batch QC RSD plots and overall distributions were generated (statTarget plot = TRUE), not just a single batch summary.

## Limitations

- Assessment depends on QC injection density and batch design (article designs used QCs every 5, 6, or 8 subject samples); sparse or non-randomized QC injections weaken the RSD estimate.
- The 15 % RSD fraction in raw data can be very small (1.97 % in Dataset 1), so per-cutoff proportions are sensitive to the chosen cutoff and to the Frule = 0.8 peak filter.
- QC-RFSC depends on the random forest ensemble (ntree = 500) and injection-order factor assignment (Fij), so RSD improvement may vary with QC count and running-order randomization.
- The article notes funding sources (HKBU IRMC/13-14/03-CHE, NSFC21675176, NSFC91543202) but gives no statement on data/code availability for the QC-RFSC algorithm itself, limiting reproducibility of the exact RSD numbers.
- Comparison with Combat (shiftCor_dQC) is not apples-to-apples since Combat is for QC-free datasets; do not use QC-RSD assessment on its output without pooled QCs.

## Evidence

- [results] QC-RFSC improves the 30 % RSD proportion on Dataset 1: [evidence span withheld — anchor status: not_found]
