---
name: metabolomics-qc-workflow-design
description: Use when you must evaluate or correct time-related signal drift in untargeted
  LC–MS batches that include pooled/intrastudy QC samples acquired in order (e.g.,
  mzML runs from MetaboLights MTBLS1919).
when_to_use_negative:
- Input is already a curated feature table without raw mzML data or injection-order
  metadata, so drift cannot be evaluated on QC injections in acquisition order
- The batch contains no pooled/intrastudy QC samples — LOESS batch correction and
  drift evaluation require features present in pooled QC injections
- The goal is metabolite/feature annotation — the paper explicitly notes TidyMS lacks
  feature annotation capability
edam_operation: http://edamontology.org/operation_3435
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: TidyMS
  role: 'Core Python package: reads mzML raw data, runs CWT peak picking, feature
    extraction, signal drift evaluation, and LOESS batch correction'
  repo: https://github.com/griquelme/tidyms
- name: pyopenms
  role: Backend mzML reader used by TidyMS to load raw LC–MS data
- name: scikit-learn
  role: PCA model construction on pooled/intrastudy QC samples to detect time-related
    systematic variance and outlier injections
- name: statsmodels
  role: LOESS smoother implementation used for modeling QC feature-intensity drift
    over injection time and batch correction
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.3390/metabo10100416
    title: A Python-Based Pipeline for Preprocessing LC–MS Data for Untargeted Metabolomics
      Workflows
  source_content_hash: sha256:e9a413cbfceae6220ff8947993683eb6a1ff528df0a0cf2074b47b5124505397
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/griquelme/tidyms
  tools:
  - TidyMS
  - pyopenms
  - scikit-learn
  - statsmodels
  edam_operation: http://edamontology.org/operation_3435
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 6
---

# metabolomics-qc-workflow-design

## Summary

Designs and executes the LC–MS quality-control workflow demonstrated in the TidyMS 2020 paper: system suitability check, system conditioning, signal drift evaluation on pooled/intrastudy QC injections, and QC-driven data curation with LOESS batch correction.

## When to use

Use when you must evaluate or correct time-related signal drift in untargeted LC–MS batches that include pooled/intrastudy QC samples acquired in order (e.g., mzML runs from MetaboLights MTBLS1919).

## When NOT to use

- Input is already a curated feature table without raw mzML data or injection-order metadata, so drift cannot be evaluated on QC injections in acquisition order
- The batch contains no pooled/intrastudy QC samples — LOESS batch correction and drift evaluation require features present in pooled QC injections
- The goal is metabolite/feature annotation — the paper explicitly notes TidyMS lacks feature annotation capability

## Inputs

- mzML raw LC–MS data files (e.g., NIST SRM 1950 / candidate RM 8231 plasma runs from MetaboLights dataset MTBLS1919)
- Sample metadata table labeling pooled/intrastudy QC injections, study samples, and blanks with acquisition order
- Solvent, zero-volume injection, and process blank injections for blank-based filtering (peak area kept only if >10-fold the maximum blank peak area)

## Outputs

- PCA score plots of pooled/intrastudy QC samples revealing drift along PC1
- Drift-evaluation artifact: plot/table of QC signal versus injection order with the fitted LOESS drift trend (e.g., signal_drift_evaluation.png)
- Batch-corrected feature intensity matrix after LOESS drift removal
- Curated feature table after the Rt filter (<90 s removal), pooled-QC presence filter, blank filter, RSD<20% QC filter, 100% prevalence (area threshold 5) filter, and D-ratio filter

## How to apply

Load the mzML raw data with TidyMS (pyopenms-backed reader) together with the sample metadata table (e.g., MetaboLights MTBLS1919 for NIST SRM 1950 / candidate RM 8231 plasma), and extract features with the CWT-based peak-picking algorithm. Assemble the pooled/intrastudy QC injections — equal-aliquot pools of every plasma sample — in acquisition order and track each feature's intensity across the batch. Build PCA models (scikit-learn) on the QC samples to reveal time-related systematic variance (drift along PC1) and flag outlier injections. Apply a LOESS smoother (statsmodels) to the QC-sample feature intensities over injection time to model drift and perform batch correction; only features present in the pooled/intrastudy QC samples are eligible for LOESS correction. During curation, discard features with retention time below 90 s (system dead time ~0.8 min) and eliminate features with RSD larger than 20% across QC injections. The article reports ~2 s per sample for feature detection, so runtime should scale roughly linearly with the number of raw files.

## Related tools

- **TidyMS** (Core Python package: reads mzML raw data, runs CWT peak picking, feature extraction, signal drift evaluation, and LOESS batch correction) — https://github.com/griquelme/tidyms
- **pyopenms** (Backend mzML reader used by TidyMS to load raw LC–MS data)
- **scikit-learn** (PCA model construction on pooled/intrastudy QC samples to detect time-related systematic variance and outlier injections)
- **statsmodels** (LOESS smoother implementation used for modeling QC feature-intensity drift over injection time and batch correction)

## Evaluation signals

- PCA score plots of pooled/intrastudy QC samples show a time-related trend along PC1 before correction, and this trend is absent after LOESS batch correction
- Drift artifact is produced: a plot/table of QC signal versus injection order with the fitted LOESS trend (signal_drift_evaluation.png)
- Runtime scales as reported: ~2 s per sample for feature detection (Application 1 analyzed 31 raw files in 66 s)
- Only features present in pooled/intrastudy QC samples are retained for LOESS batch correction; features with retention time <90 s are discarded
- Curated table contains no features with RSD >20% across pooled/intrastudy QC injections

## Limitations

- TidyMS does not provide feature annotation capability
- No software version numbers are stated for TidyMS, Python, or scikit-learn, so exact reproducibility of versions must be pinned by the practitioner
- Drift evaluation and LOESS correction depend on pooled/intrastudy QC injections being present and ordered in the metadata; batches lacking ordered QC samples cannot be assessed

## Evidence

- [abstract] abstract: "The capabilities of the package are shown with pipelines for an LC–MS system suitability check, system conditioning, signal drift evaluation, and data curation."
- [methods] methods: "The mass spectrometry data have been deposited to the Metabo Lights public repository [38] with the data set identifier MTBLS1919."
- [results] results: "The running time needed to analyze the 31 raw data files used to generate the results shown in Application 1 was 66 s."
