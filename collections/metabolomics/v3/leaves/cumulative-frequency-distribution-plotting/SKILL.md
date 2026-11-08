---
name: cumulative-frequency-distribution-plotting
description: Use when comparing the precision of corrected vs raw LC-MS/GC-MS metabolomics
  or proteomics feature tables by measuring how many features fall below successive
  RSD (CV%) thresholds in QC samples.
when_to_use_negative:
- The input is already a summary table of RSD bins or percentages — the skill expects
  per-feature QC intensity values so RSDs can be computed from scratch.
- There are too few QC injections (e.g. fewer than a handful) to give a stable CV
  estimate per feature; the article relies on QC repeated every 5–8 subject samples
  across batches.
- The goal is subject-sample (biological) variability, not QC (technical) precision
  — the cumulative RSD plot is specifically computed across aliquoted QC injections.
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: statTarget
  role: R package whose shiftCor function performs the correction and whose QC plotting
    module produces the 'RSD distribution' cumulative plot of QC-feature RSDs
  repo: github.com/13479776/statTarget
- name: shiftCor
  role: statTarget function invoked once per correction method (MLmethod='QCRFSC',
    'QCRLSC', QC-SVR equivalent) with Frule=0.8, imputeM='KNN' to generate corrected
    QC feature tables used as the plot's input curves
  repo: github.com/13479776/statTarget
- name: XCMS
  role: Peak-extraction tool whose transX-generated output can be converted into statTarget
    input format before RSDs are computed
- name: Skyline
  role: Peak-extraction tool for SRM proteomics whose output feeds the same cumulative
    RSD-distribution evaluation in the statTarget workflow
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.aca.2018.08.002
    title: 'statTarget: A streamlined tool for signal drift correction and interpretations
      of quantitative mass spectrometry-based omics data'
  source_content_hash: sha256:a66d276b1f29ffac2fa3a9406dff0b84f6eb354cfbe8a813c1fb4748c5f81eb3
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - QC-RLSC
  - QC-SVR
  - Skyline
  - XCMS
  - shiftCor
  - statTarget
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# cumulative-frequency-distribution-plotting

## Summary

Plot the cumulative frequency distribution of QC-sample feature RSDs (CV%) across binned cutoffs (e.g. CV<5%, 10%, ... 100%) to quantify how a signal-drift correction method, such as QC-RFSC, improves metabolomics/proteomics data precision versus raw data and alternative correction algorithms.

## When to use

Use when comparing the precision of corrected vs raw LC-MS/GC-MS metabolomics or proteomics feature tables by measuring how many features fall below successive RSD (CV%) thresholds in QC samples.

## When NOT to use

- The input is already a summary table of RSD bins or percentages — the skill expects per-feature QC intensity values so RSDs can be computed from scratch.
- There are too few QC injections (e.g. fewer than a handful) to give a stable CV estimate per feature; the article relies on QC repeated every 5–8 subject samples across batches.
- The goal is subject-sample (biological) variability, not QC (technical) precision — the cumulative RSD plot is specifically computed across aliquoted QC injections.

## Inputs

- Feature (peak) intensity matrix with injection order, including aliquoted QC sample rows (e.g. statTarget shiftCor output or raw XCMS/Skyline peak table)
- Uncorrected raw QC peak-intensity table as the baseline condition
- Corrected QC peak tables for each drift-correction method (QC-RFSC, QC-RLSC, QC-SVR)

## Outputs

- Cumulative frequency distribution plot of QC-feature RSDs overlaid across raw and corrected conditions (e.g. rsd_distribution_comparison.png)
- Cumulative-frequency table of feature counts/proportions per RSD bin per condition (optional CSV, e.g. rsd_distribution_comparison.csv)
- statTarget-style 'RSD distribution.pdf' QC figure if generated inside statTarget

## How to apply

Load the per-feature peak intensities of the aliquoted QC injections (the same QC sample list/profile used for drift correction, e.g. statTarget shiftCor output or the raw peak table from XCMS/Skyline extraction). For every feature compute the RSD (CV%) across QC injections, then bin features into cumulative frequency bins — CV<5%, CV<10%, CV<15%, ..., CV<100% — as produced by the statTarget 'RSD distribution' QC plot. Plot the cumulative proportion of features at each bin as a single overlaid chart, one curve per condition: uncorrected raw data, QC-RFSC (MLmethod='QCRFSC'), QC-RLSC (MLmethod='QCRLSC', QCspan=0), and QC-SVR. Rationale: the article evaluates corrections by the shift of the cumulative RSD distribution — e.g. in dataset 1 the proportion of peaks with RSDs <30% moved from 43.9% (raw) to 86.2% (QC-RFSC) versus 65.5% (QC-RLSC) and 61.9% (QC-SVR) — so the plot directly visualizes the precision gain. Save the figure (e.g. rsd_distribution_comparison.png) and optionally the underlying cumulative-frequency table as CSV.

## Related tools

- **statTarget** (R package whose shiftCor function performs the correction and whose QC plotting module produces the 'RSD distribution' cumulative plot of QC-feature RSDs) — github.com/13479776/statTarget
- **shiftCor** (statTarget function invoked once per correction method (MLmethod='QCRFSC', 'QCRLSC', QC-SVR equivalent) with Frule=0.8, imputeM='KNN' to generate corrected QC feature tables used as the plot's input curves) — github.com/13479776/statTarget
- **XCMS** (Peak-extraction tool whose transX-generated output can be converted into statTarget input format before RSDs are computed)
- **Skyline** (Peak-extraction tool for SRM proteomics whose output feeds the same cumulative RSD-distribution evaluation in the statTarget workflow)

## Examples

```
shiftCor(samPeno, samFile, Frule = 0.8, MLmethod = "QCRFSC", QCspan = 0, imputeM = "KNN")  # then plot cumulative RSD distribution of QC features
```

## Evaluation signals

- For raw data in dataset 1 the cumulative proportion of peaks with RSDs <15% should be ≈1.97% and <30% ≈43.9%; after QC-RFSC these should rise to ≈25.1% and ≈86.2% respectively — deviations suggest wrong QC rows or wrong binning.
- In metabolomics dataset 2 the <30% RSD cumulative proportion should be ≈52.1% (raw) rising to ≈90.9% (QC-RFSC), with QC-RLSC ≈80.1% and QC-SVR ≈72.3%.
- The x-axis of the plot must be cumulative RSD cutoffs (5%, 10%, ..., 100%) and the y-axis the cumulative proportion of features; each bin's count must be monotonically non-decreasing as the cutoff increases.
- The raw-data curve should sit at or below every corrected-method curve at each cutoff; if QC-RFSC is below raw at any bin, the correction input (QC sample list / profile) is likely mis-specified.
- The plotted values must match the exported cumulative-frequency table row-by-row (figure and CSV agree).

## Limitations

- The cumulative RSD plot only reflects technical precision in aliquoted QC injections; it does not by itself assess biological signal preservation or classification accuracy.
- Correction quality depends on QC injection frequency (every 5, 6, or 8 subject samples in datasets 1–3) and on the Frule=0.8 feature filter and KNN imputation settings; sparse or irregular QC placement weakens the RSD estimate.
- For proteomics (dataset 3, SRM with 174 transitions across 69 proteins) the same RSD-distribution logic applies, but absolute proportions may differ from metabolomics datasets.
- On macOS, statTarget's GUI-based plotting requires XQuartz instead of X11 support; without it the QC RSD-distribution figure cannot be produced interactively.

## Evidence

- [results] the proportion of peaks with RSDs less than 30 % increased significantly from 43.9% to 86.2 % after QC-RFSC method.: [evidence span withheld — anchor status: not_found]
- [results] The QC-RLSC and QC-SVR algorithms also increased the proportion of peaks with 30% RSDs to 65.5% and 61.9 %, respectively: "The QC-RLSC and QC-SVR algorithms also increased the proportion of peaks with 30% RSDs to 65.5% and 61.9 %, respectively"
- [methods] shiftCor(samPeno,samFile, Frule = 0.8, MLmethod = "QCRFSC", QCspan = 0,imputeM = "KNN"): "shiftCor(samPeno,samFile, Frule = 0.8, MLmethod = "QCRFSC", QCspan = 0,imputeM = "KNN")"
- [methods] The aliquoted QC samples were analyzed after every five aliquoted real samples in the entire batch: "The aliquoted QC samples were analyzed after every five aliquoted real samples in the entire batch"
- [results] we evaluated the cumulative RSD distribution of all features in metabolomics data (dataset 2): "we evaluated the cumulative RSD distribution of all features in metabolomics data (dataset 2)"
