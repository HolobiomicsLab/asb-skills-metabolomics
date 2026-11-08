---
name: metabolomics-signal-drift-correction
description: Use when a metabolomics or proteomics feature intensity table (e.
when_to_use_negative:
- The dataset contains no pooled QC injections (e.g. a QC-free design) — use the statTarget
  Combat workflow via shiftCor_dQC instead of QCRFSC.
- Peak intensities have not been extracted yet (raw mzXML/MS vendor files rather than
  a feature table) — first use XCMS or Skyline peak detection and statTarget transX
  before applying correction.
- The aim is inter-batch combat adjustment only after QC correction has already been
  applied, not feature-level drift modelling against injection order.
edam_operation: http://edamontology.org/operation_3801
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: statTarget
  role: R package implementing the shiftCor QC-RFSC correction and the statAnalysis
    downstream statistics; invoked with MLmethod='QCRFSC', Frule=0.8, ntree=500
  repo: github.com/13479776/statTarget
- name: random forest
  role: Ensemble-learning regression (ntree=500) fitted on QC intensities vs injection
    order to predict the correction factor Fij for each feature
- name: XCMS
  role: Peak abundance extraction producing the metabolomics feature table that statTarget
    transX converts into shiftCor input format
- name: Skyline
  role: Tryptic peptide peak extraction for SRM proteomics (174 transitions, 69 proteins)
    feeding the proteomics correction workflow
provenance:
  source_task_ids:
  - task_001
  - task_002
  - task_003
  source_papers:
  - doi: 10.1016/j.aca.2018.08.002
    title: 'statTarget: A streamlined tool for signal drift correction and interpretations
      of quantitative mass spectrometry-based omics data'
  source_content_hash: sha256:3b001445504019be1a614783b0e9f3c3f43c7306d6a2507303757c333ff3ccf8
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - QC-RLSC
  - QC-SVR
  - Skyline
  - XCMS
  - random forest
  - shiftCor
  - statTarget
  edam_operation: http://edamontology.org/operation_3801
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7
n_claims: 20
---

# metabolomics-signal-drift-correction

## Summary

Applies QC-based random forest signal correction (QC-RFSC) from statTarget to remove inter- and intra-batch signal drift and batch variation at the feature level in LC/GC-MS metabolomics and proteomics intensity tables, by learning a per-feature correction factor from pooled QC injections across the run order.

## When to use

Use when a metabolomics or proteomics feature intensity table (e.g. from XCMS, Skyline, MZmine2 or SIEVE) is ordered by injection sequence and pooled QC samples were injected at fixed intervals every 5-8 real samples, and the goal is to correct feature-level signal drift and batch effects before statistical analysis.

## When NOT to use

- The dataset contains no pooled QC injections (e.g. a QC-free design) — use the statTarget Combat workflow via shiftCor_dQC instead of QCRFSC.
- Peak intensities have not been extracted yet (raw mzXML/MS vendor files rather than a feature table) — first use XCMS or Skyline peak detection and statTarget transX before applying correction.
- The aim is inter-batch combat adjustment only after QC correction has already been applied, not feature-level drift modelling against injection order.

## Inputs

- Sample meta-information table (samPeno) with sample names, batch, injection order and class labels
- Raw feature-by-sample intensity matrix (samFile) exported from XCMS, Skyline, MZmine2 or SIEVE (convertible via statTarget transX)
- Injection-order-ordered QC injections (pooled QC injected after every 5 real samples in dataset 1, every 6 plasma samples in dataset 2, every 8 subject samples in dataset 3)

## Outputs

- shift_all_cor.csv — integrated QC-RFSC-corrected feature table across all batches
- shift_QC_cor.csv — corrected QC-sample intensities only
- shift_sample_cor.csv — corrected biological-sample intensities only
- RSD distribution plot (RSDresult / RSD distribution.pdf) summarising the relative standard deviation of corrected QC peaks

## How to apply

Run the statTarget shiftCor function on a sample meta-information file (batch, injection order, sample class) and the raw peak intensity table, with MLmethod='QCRFSC', Frule=0.8, QCspan=0, ntree=500, and imputeM='KNN'. For each feature the random forest (500 trees) is fitted on QC-sample intensities versus injection order, the fitted regression value at each sample's injection position serves as the correction factor Fij, and each raw intensity xij is divided by Fij (eq 1-2). Features are filtered by the 80 percent rule (non-zero values in at least 80 percent of samples in any one group) and remaining missing values are KNN-imputed. If no QC samples are available, fall back to shiftCor_dQC with MLmethod='Combat' instead. The function emits shift_all_cor.csv (integrated corrected data), shift_QC_cor.csv and shift_sample_cor.csv, plus an RSD distribution QC plot; downstream PCA should be computed via statAnalysis with scaling='Pareto' and glog=TRUE.

## Related tools

- **statTarget** (R package implementing the shiftCor QC-RFSC correction and the statAnalysis downstream statistics; invoked with MLmethod='QCRFSC', Frule=0.8, ntree=500) — github.com/13479776/statTarget
- **random forest** (Ensemble-learning regression (ntree=500) fitted on QC intensities vs injection order to predict the correction factor Fij for each feature)
- **XCMS** (Peak abundance extraction producing the metabolomics feature table that statTarget transX converts into shiftCor input format)
- **Skyline** (Tryptic peptide peak extraction for SRM proteomics (174 transitions, 69 proteins) feeding the proteomics correction workflow)

## Examples

```
shiftCor(samPeno, samFile, Frule = 0.8, MLmethod = "QCRFSC", QCspan = 0, imputeM = "KNN")
```

## Evaluation signals

- Proportion of QC peaks with RSD below 30% rises markedly after correction — reported from 43.9% (raw) to 86.2% for dataset 1 and 52.1% (raw) to 90.9% for metabolomics dataset 2.
- Peaks within 15% RSD increase ~12.7-fold (1.97% to 25.1%) in dataset 1 after QC-RFSC.
- PCA score plots computed on corrected data (statAnalysis, Pareto scaling, glog) show QC samples clustering within a markedly smaller variance region than on raw intensities.
- Output files shift_all_cor.csv, shift_QC_cor.csv and shift_sample_cor.csv exist with corrected intensities for every input feature that passed Frule=0.8.
- R-square/predictive accuracy of the random forest model should be high at median and first-quartile values for dataset 1, as reported for RF, QRF and Loess.

## Limitations

- QC-RFSC requires pooled QC injections interleaved across the batch; datasets lacking periodic QC samples cannot be corrected with this method (use Combat via shiftCor_dQC for QC-free data).
- Features failing the 80% rule (non-zero values in at least 80% of samples in any one group) or with excessive missingness are removed before correction, which shrinks the feature table.
- Performance was characterised on specific LC-MS metabolomics and SRM proteomics datasets; correction quality depends on QC coverage (every 5-8 samples) and may not generalise to sparse-QC designs.
- Requires R >= 3.3.0; on macOS the GUI additionally needs XQuartz and specific RGtk2 versions (RGtk2 2.20.31 with R 3.3.0 was tested).

## Evidence

- [results] Improvement in proportion of peaks below 30% RSD after correction: [evidence span withheld — anchor status: not_found]
- [methods] statTarget citation and purpose in paper: [evidence span withheld — anchor status: not_found]
