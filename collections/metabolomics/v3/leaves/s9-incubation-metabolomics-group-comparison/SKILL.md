---
name: s9-incubation-metabolomics-group-comparison
description: Use when raw MS1 LC-HRMS data from enzymatic (S9) incubation experiments
  with matched heat-inactivated controls need differential feature screening to isolate
  biotransformation-derived metabolites.
when_to_use_negative:
- Input is already a curated/annotated metabolite list or feature table rather than
  raw MS1 incubation data — use downstream CMDN layers (RCS/SSN) instead.
- The comparison lacks matched heat-inactivated controls, so differential PG/CG statistics
  (fold change, SNR, t-test) cannot be computed meaningfully.
- Goal is targeted quantification of known compounds rather than untargeted discovery
  of biotransformation-derived features.
edam_operation: http://edamontology.org/operation_3214
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: MS-Dial
  role: Imports raw MS1 files and performs peak detection, deconvolution, feature
    extraction, and cross-sample alignment of features by accurate m/z and retention
    time
  repo: https://github.com/LeaveMeNotTonight/CMDN
- name: LC-HRMS
  role: Instrument platform generating the raw MS1 incubation data subjected to differential
    screening
- name: CMDN
  role: Top-down MS data processing framework whose DMS layer this skill implements;
    its output table feeds the RCS and SSN layers
  repo: https://github.com/LeaveMeNotTonight/CMDN
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/acs.est.5c08558
    title: Reaction-Guided Metabolomics Accelerates High-Throughput Annotation of
      Xenobiotic Metabolites for Human Exposome
  source_content_hash: sha256:b2e3fda2d8db2ed354bddac1cefb94644ef97817999bedfeaa0311e6f0f3a49d
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/LeaveMeNotTonight/CMDN
  tools:
  - CMDN
  - LC-HRMS
  - MS-Dial
  edam_operation: http://edamontology.org/operation_3214
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 18
---

# s9-incubation-metabolomics-group-comparison

## Summary

Differential metabolic feature screening between pesticide-incubation (PG) and heat-inactivated control (CG) human liver S9 LC-HRMS samples, forming the differential metabolite space (DMS) layer of the CMDN workflow. Features are detected and aligned in MS-Dial, then filtered by fold-change, signal-to-noise, and t-test thresholds.

## When to use

Use when raw MS1 LC-HRMS data from enzymatic (S9) incubation experiments with matched heat-inactivated controls need differential feature screening to isolate biotransformation-derived metabolites.

## When NOT to use

- Input is already a curated/annotated metabolite list or feature table rather than raw MS1 incubation data — use downstream CMDN layers (RCS/SSN) instead.
- The comparison lacks matched heat-inactivated controls, so differential PG/CG statistics (fold change, SNR, t-test) cannot be computed meaningfully.
- Goal is targeted quantification of known compounds rather than untargeted discovery of biotransformation-derived features.

## Inputs

- Raw MS1 LC-HRMS data files from human liver S9 incubation experiments (PG samples and heat-inactivated CG controls)
- Sample group metadata distinguishing pesticide-treated (PG) vs heat-inactivated control (CG) injections

## Outputs

- dms_feature_table.csv containing differentially expressed features with m/z, retention time, intensities, fold change (PG/CG), signal-to-noise ratio, and p-value columns
- Aligned feature-intensity matrix from MS-Dial covering all PG and CG samples

## How to apply

Import raw MS1 files from the human liver S9 incubation experiments into MS-Dial (ver. 4.80) for peak detection, deconvolution, and feature extraction. Align features across PG and CG samples by accurate m/z and retention time to build a single feature-intensity matrix. For each aligned feature compute the PG/CG fold change, the signal-to-noise ratio, and an unadjusted Student's t-test p-value. Retain only features meeting all three DMS thresholds: fold-change PG/CG >= 2, signal-to-noise >= 10, and unadjusted p <= 0.05. Export the surviving features (m/z, RT, intensities, fold change, SNR, p-value) as dms_feature_table.csv, which serves as the input space for the downstream CMDN reaction-based candidature space (RCS) and MS2 similarity network (SSN) layers.

## Related tools

- **MS-Dial** (Imports raw MS1 files and performs peak detection, deconvolution, feature extraction, and cross-sample alignment of features by accurate m/z and retention time) — https://github.com/LeaveMeNotTonight/CMDN
- **LC-HRMS** (Instrument platform generating the raw MS1 incubation data subjected to differential screening)
- **CMDN** (Top-down MS data processing framework whose DMS layer this skill implements; its output table feeds the RCS and SSN layers) — https://github.com/LeaveMeNotTonight/CMDN

## Examples

```
Rscript dms_layer.R --msdial-project PG_CG_alignment.msdial --group-PG PG/ --group-CG CG/ --fc 2 --snr 10 --p 0.05 --out dms_feature_table.csv
```

## Evaluation signals

- Every retained feature in dms_feature_table.csv satisfies all three thresholds: fold-change PG/CG >= 2, signal-to-noise >= 10, and unadjusted p <= 0.05.
- The exported table carries m/z and RT fields required by the downstream reaction-based candidature space (RCS).
- No features excluded by the thresholds appear in the output table; re-running the filters on the full aligned matrix reproduces the exported feature set exactly.
- Alignment yields a single feature-intensity matrix keyed by accurate m/z and retention time covering both PG and CG groups.

## Limitations

- Unadjusted Student's t-test p-values are used (no multiple-testing correction described), so false positives may pass the p <= 0.05 filter at omics scale.
- Threshold-based screening (fold-change >= 2, SNR >= 10) is a coarse differential criterion; ambiguous or borderline biotransformation features may be lost before downstream annotation layers.
- Feature alignment quality depends on MS-Dial peak detection/deconvolution parameters; misalignment by m/z or RT can corrupt fold-change and SNR estimates.

## Evidence

- [other] DMS screening thresholds and pipeline: [evidence span withheld — anchor status: not_found]
