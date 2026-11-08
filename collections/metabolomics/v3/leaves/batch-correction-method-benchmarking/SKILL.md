---
name: batch-correction-method-benchmarking
description: Use when you have LC-MS or GC-MS metabolomics/proteomics peak tables
  with pooled QC injections interspersed among study samples (e.g. every 5, 6 or 8
  samples) and need to decide which batch-correction algorithm to apply by comparing
  post-correction RSD distributions.
when_to_use_negative:
- Your data lacks pooled QC injections (e.g. QC-free datasets) — for those, statTarget
  offers Combat via shiftCor_dQC instead, and this benchmarking scheme is not applicable.
- You are still at raw file processing stage (mzML/mzXML conversion, peak detection
  via XCMS/Skyline) — this skill operates on the extracted peak-intensity feature
  table, not on MS raw data.
- You have only a single correction method to evaluate and no raw-data or alternative-method
  baseline to compare against — the benchmark requires a raw or reference condition.
edam_operation: http://edamontology.org/operation_3800
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: statTarget
  role: Provides the shiftCor function used to run QC-RFSC, QC-RLSC (and Combat for
    QC-free data) and the RSD distribution QC plot used for benchmarking.
  repo: github.com/13479776/statTarget
- name: QC-RFSC
  role: The QC-based random forest signal correction algorithm being benchmarked (with
    ntree=500, imputeM='KNN') against other drift-correction methods.
  repo: github.com/13479776/statTarget
- name: QC-RLSC
  role: QC-based LOESS signal correction run as a comparator method in the benchmark.
  repo: github.com/13479776/statTarget
- name: QC-SVR
  role: QC-based support vector regression correction run as a second comparator method
    in the benchmark.
  repo: github.com/13479776/statTarget
- name: randomForest
  role: Underlying random forest regression engine used by the QC-RFSC algorithm (ntree=500)
    to model signal drift.
  repo: github.com/13479776/statTarget
- name: XCMS
  role: Upstream tool for peak extraction whose output can be transformed (via transX)
    into statTarget-compatible input files for the benchmark.
- name: Skyline
  role: Upstream peak-extraction tool whose output can be transformed (via transX)
    into statTarget-compatible input files for the benchmark.
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.aca.2018.08.002
    title: 'statTarget: A streamlined tool for signal drift correction and interpretations
      of quantitative mass spectrometry-based omics data'
  source_content_hash: sha256:dccd57096d7481cb1d1533aac170b6b5872c6413474b4dffcb91712d4e759f53
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - QC-RFSC
  - QC-RLSC
  - QC-SVR
  - Skyline
  - XCMS
  - randomForest
  - shiftCor
  - statTarget
  edam_operation: http://edamontology.org/operation_3800
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8125
n_claims: 16
---

# batch-correction-method-benchmarking

## Summary

Benchmark QC-based signal drift correction methods (QC-RFSC, QC-RLSC, QC-SVR) by comparing the cumulative RSD% distribution of features in QC injections against raw, uncorrected data, to establish which correction improves data precision most.

## When to use

Use when you have LC-MS or GC-MS metabolomics/proteomics peak tables with pooled QC injections interspersed among study samples (e.g. every 5, 6 or 8 samples) and need to decide which batch-correction algorithm to apply by comparing post-correction RSD distributions.

## When NOT to use

- Your data lacks pooled QC injections (e.g. QC-free datasets) — for those, statTarget offers Combat via shiftCor_dQC instead, and this benchmarking scheme is not applicable.
- You are still at raw file processing stage (mzML/mzXML conversion, peak detection via XCMS/Skyline) — this skill operates on the extracted peak-intensity feature table, not on MS raw data.
- You have only a single correction method to evaluate and no raw-data or alternative-method baseline to compare against — the benchmark requires a raw or reference condition.

## Inputs

- Feature peak-intensity table (samples × features) from metabolomics or proteomics LC-MS/GC-MS runs (e.g. statTarget-compatible profile CSV derived from XCMS, MZmine2, SIEVE or Skyline)
- Sample metadata / annotation file identifying which injections are pooled QC samples and their batch/run order (statTarget sampleList / samPeno format)
- Pooled QC injections interspersed at a fixed frequency within each batch (e.g. every 5, 6 or 8 subject samples)

## Outputs

- Per-feature RSD (CV%) computed across QC injections for each correction condition (raw, QC-RFSC, QC-RLSC, QC-SVR)
- Cumulative frequency distribution (e.g. % of features with RSD < 15%, < 30%) per condition, plotted on one chart (analogous to statTarget's 'RSD distribution.pdf')
- A ranked comparison of correction methods by the proportion of QC features achieving acceptable precision

## How to apply

Run the same peak-intensity feature table through each candidate correction method in statTarget's shiftCor: QC-RFSC (MLmethod='QCRFSC', ntree=500, imputeM='KNN', Frule=0.8), QC-RLSC (MLmethod='QCRLSC', QCspan=0), and QC-SVR, keeping the raw uncorrected peak table as baseline. For each corrected (and raw) feature table, compute the relative standard deviation (RSD%, CV) of every feature across the pooled QC injections. Build a cumulative frequency distribution of QC-feature RSDs (e.g. proportion of peaks with RSD < 15%, < 30%, etc.) for each condition and compare them on a single chart, as in the package's 'RSD distribution.pdf' plot. The method that most increases the proportion of QC peaks below a 30% RSD threshold — in this paper QC-RFSC lifted that proportion from 43.9% (raw) to 86.2%, versus 65.5% for QC-RLSC and 61.9% for QC-SVR in dataset 1 — is judged the better correction.

## Related tools

- **statTarget** (Provides the shiftCor function used to run QC-RFSC, QC-RLSC (and Combat for QC-free data) and the RSD distribution QC plot used for benchmarking.) — github.com/13479776/statTarget
- **QC-RFSC** (The QC-based random forest signal correction algorithm being benchmarked (with ntree=500, imputeM='KNN') against other drift-correction methods.) — github.com/13479776/statTarget
- **QC-RLSC** (QC-based LOESS signal correction run as a comparator method in the benchmark.) — github.com/13479776/statTarget
- **QC-SVR** (QC-based support vector regression correction run as a second comparator method in the benchmark.) — github.com/13479776/statTarget
- **randomForest** (Underlying random forest regression engine used by the QC-RFSC algorithm (ntree=500) to model signal drift.) — github.com/13479776/statTarget
- **XCMS** (Upstream tool for peak extraction whose output can be transformed (via transX) into statTarget-compatible input files for the benchmark.)
- **Skyline** (Upstream peak-extraction tool whose output can be transformed (via transX) into statTarget-compatible input files for the benchmark.)

## Examples

```
shiftCor(samPeno,samFile, Frule = 0.8, MLmethod = "QCRFSC", QCspan = 0,imputeM = "KNN")
```

## Evaluation signals

- The proportion of QC features with RSD < 30% should increase relative to the raw (uncorrected) data for every tested correction method; in this paper's dataset 1 the expected pattern was raw 43.9% → QC-RFSC 86.2% → QC-RLSC 65.5% → QC-SVR 61.9%.
- For raw data in dataset 1 the proportion of peaks with RSD < 15% was only 1.97% of total peaks, rising to 25.1% (a 12.7-fold increase) after QC-RFSC — this magnitude of improvement is a sanity anchor.
- The cumulative RSD distribution must be computed on the same QC injection set for every condition, so curves are directly comparable on one chart.
- The Frule=0.8 filter (features kept if non-zero in ≥80% of samples in any one group) should be applied consistently before correction in every condition being compared.
- Plotting each condition's cumulative distribution as a single figure (mirroring statTarget's 'RSD distribution.pdf') confirms the benchmark was assembled as a head-to-head comparison, not as isolated per-method metrics.

## Limitations

- The comparison depends on pooled QC injections being present at a regular interval within each batch (every 5, 6, or 8 subject samples in the paper's datasets 1–3); datasets without regular QC injections cannot use QC-based correction methods or this benchmark.
- Loess-based correction can overfit when few QC samples are available; the paper notes that RF, QRF and Loess all showed high predictive accuracy (R² at median and first quartile) for dataset 1, but small-QC scenarios are a known fragility of QC-RLSC (hence QCspan=0 default in statTarget).
- The RSD-based benchmark assesses precision only — it does not by itself evaluate biological signal preservation; separate statistical accuracy checks (e.g. downstream statAnalysis with glog transformation and Pareto scaling) are needed to confirm the correction improves statistical accuracy as well.

## Evidence

- [results] The QC-RLSC and QC-SVR algorithms also increased the proportion of peaks with 30% RSDs to 65.5% and 61.9 %, respectively: "The QC-RLSC and QC-SVR algorithms also increased the proportion of peaks with 30% RSDs to 65.5% and 61.9 %"
- [results] The proportion of peaks with RSDs less than 30 % increased significantly from 43.9% to 86.2 % after QC-RFSC method.: [evidence span withheld — anchor status: not_found]
- [results] In the raw data, the proportion of peaks within 15 % RSDs was only 1.97 % of the total peaks.: "In the raw data, the proportion of peaks within 15 % RSDs was only 1.97 % of the total peaks."
- [methods] shiftCor(samPeno,samFile, Frule = 0.8, MLmethod = "QCRFSC", QCspan = 0,imputeM = "KNN"): "shiftCor(samPeno,samFile, Frule = 0.8, MLmethod = "QCRFSC", QCspan = 0,imputeM = "KNN")"
- [methods] The aliquoted QC samples were analyzed after every five aliquoted real samples in the entire batch: "The aliquoted QC samples were analyzed after every five aliquoted real samples in the entire batch"
- [readme] Dependent on R (>= 3.3.0): [evidence span withheld — anchor status: external_doc]
- [methods] A variable will be kept if it has a non-zero value for at least n precent of samples in any one group. (Default: 0.8): "A variable will be kept if it has a non-zero value for at least n precent of samples in any one group. (Default: 0.8)"
- [methods] In addition, Combat is also provided for QC free datasets: "In addition, Combat is also provided for QC free datasets"
