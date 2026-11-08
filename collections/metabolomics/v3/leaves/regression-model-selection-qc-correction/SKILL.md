---
name: regression-model-selection-qc-correction
description: Use when designing a QC-based signal correction pipeline and needing
  to choose which regression algorithm should model signal drift from serially injected
  QC samples across candidate models.
when_to_use_negative:
- The data are a single-batch or QC-free dataset — Combat-style correction (statTarget's
  shiftCor_dQC) or batch-free approaches are more appropriate than drift regression
  fitted on QC injections
- The goal is evaluating correction performance (RSD distributions, precision) rather
  than choosing the regression model itself
- Features are unfiltered raw peaks — apply quality filtering first (e.g., 80 percent
  rule / Frule = 0.8) so drift is modeled on reliable features
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_3474
tools:
- name: statTarget
  role: Implements QC-RFSC correction with the selected random forest regression (shiftCor
    function, MLmethod='QCRFSC') and provides statAnalysis downstream statistics
  repo: github.com/13479776/statTarget
- name: random forest
  role: The selected regression basis of QC-RFSC, chosen after benchmarking twelve
    supervised ML algorithms by 10-fold cross validated R-square
- name: QC-RLSC
  role: LOESS-based QC correction baseline compared against the random forest model
- name: QC-SVR
  role: Support vector regression QC correction baseline compared against the random
    forest model
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.aca.2018.08.002
    title: 'statTarget: A streamlined tool for signal drift correction and interpretations
      of quantitative mass spectrometry-based omics data'
  source_content_hash: sha256:d66a110712ad78f9e8263de5fcc1418ce707444d1fd5590e37a7e2c0792f79e9
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - QC-RLSC
  - QC-SVR
  - random forest
  - statTarget
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# regression-model-selection-qc-correction

## Summary

Benchmarks candidate supervised regression algorithms (e.g., twelve models via 10-fold cross validation on QC-injected LC-MS/GC-MS data) by predictive accuracy/R-square to select the regression basis for quality control-based signal drift correction, as done for statTarget's QC-RFSC random forest ensemble.

## When to use

Use when designing a QC-based signal correction pipeline and needing to choose which regression algorithm should model signal drift from serially injected QC samples across candidate models.

## When NOT to use

- The data are a single-batch or QC-free dataset — Combat-style correction (statTarget's shiftCor_dQC) or batch-free approaches are more appropriate than drift regression fitted on QC injections
- The goal is evaluating correction performance (RSD distributions, precision) rather than choosing the regression model itself
- Features are unfiltered raw peaks — apply quality filtering first (e.g., 80 percent rule / Frule = 0.8) so drift is modeled on reliable features

## Inputs

- Feature abundance table from LC-MS or GC-MS metabolomics (e.g., XCMS-extracted peak abundances) with serial QC injections and injection-order metadata
- Sample metadata/phenotype file pairing sample names with class and injection order (statTarget samPeno format)
- Per-model cross-validation accuracy/R-square distributions (e.g., violin-plot data underlying the model comparison for dataset 1)

## Outputs

- Ranked table of candidate regression models with median and first-quartile R-square (CSV)
- Selected regression basis for the QC-based correction algorithm (random forest, a 500-tree ensemble in QC-RFSC)
- Corrected feature table produced once the selected model is wired into the correction routine

## How to apply

Assemble a dataset with serially injected pooled QC samples (e.g., QC injections after every five real samples as in statTarget's dataset 1) whose repetitive measurement makes drift predictable. Evaluate each candidate supervised regression model (twelve models in the statTarget study, including random forest, quantile random forest QRF, Loess, QC-RLSC and QC-SVR baselines) using 10-fold cross validation, recording the predictive accuracy/R-square distribution per model. Rank models by the median and first-quartile R-square rather than best-case performance, since correction must be reliable across the whole distribution. In the statTarget evaluation, RF, QRF and Loess showed high predictive accuracy or R-square at median and first quartile value for dataset 1, and a 500-tree random forest ensemble was selected as the QC-RFSC regression basis. Implement the selected model within the correction function so that each feature's drift is predicted from injection order and corrected with a per-feature correction factor, e.g., via statTarget's shiftCor with MLmethod set to the chosen model.

## Related tools

- **statTarget** (Implements QC-RFSC correction with the selected random forest regression (shiftCor function, MLmethod='QCRFSC') and provides statAnalysis downstream statistics) — github.com/13479776/statTarget
- **random forest** (The selected regression basis of QC-RFSC, chosen after benchmarking twelve supervised ML algorithms by 10-fold cross validated R-square)
- **QC-RLSC** (LOESS-based QC correction baseline compared against the random forest model)
- **QC-SVR** (Support vector regression QC correction baseline compared against the random forest model)

## Examples

```
library(statTarget); shiftCor(samPeno,samFile, Frule = 0.8, MLmethod = "QCRFSC", QCspan = 0, imputeM = "KNN")
```

## Evaluation signals

- The selected model must match the reported result that RF, QRF and Loess showed high predictive accuracy or R-square at median and first quartile value for dataset 1
- Ranking must be based on both median and first-quartile cross-validated R-square, not single-point best accuracy
- When deployed as QC-RFSC, correction on dataset 1 should reproduce the reported quality gain: peaks within 15% RSD rising from 1.97% (raw) to 25.1%, and peaks <30% RSD rising from 43.9% to 86.2%, outperforming QC-RLSC (65.5%) and QC-SVR (61.9%) at 30% RSD
- The random forest ensemble uses 500 trees as in the QC-RFSC implementation
- The comparison covers the same scope: twelve supervised machine learning algorithms evaluated with 10-fold cross validation on dataset 1

## Limitations

- Model ranking is dataset-dependent: the evaluation was performed on specific MS-based metabolomics datasets (dataset 1 urine metabolomics, dataset 2 plasma from Giga Science ID 100108) and SRM proteomics data (dataset 3, 174 transitions for 69 proteins), so the winner may differ for other platforms
- Cross-validated R-square rankings require serial QC injections in the raw data; the article used QC samples after every 5, 6, or 8 real samples in datasets 1-3 respectively
- The article's funding acknowledgment (HKBU IRMC/13-14/03-CHE, NSFC21675176, NSFC91543202) is present but no explicit data/code availability statement for the QC-RFSC algorithm itself was flagged

## Evidence
