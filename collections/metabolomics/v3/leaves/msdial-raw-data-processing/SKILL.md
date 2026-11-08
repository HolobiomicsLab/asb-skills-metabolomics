---
name: msdial-raw-data-processing
description: Use when raw MS1 LC-HRMS files from case/control incubation experiments
  (e.g., human liver S9 fractions with vs. without active enzyme) must be converted
  into an aligned feature-intensity matrix and filtered to differentially expressed
  metabolites.
when_to_use_negative:
- Input is already an aligned feature-intensity table (e.g., an existing m/z/RT feature
  matrix) rather than raw MS1 vendor or converted data files
- The goal is identification/annotation of metabolites (CMDN's reaction-based candidature
  space or MS2 similarity network layers) rather than differential-feature screening
- MS2 spectra acquisition or targeted quantification of known standards is needed,
  since this skill covers only MS1-level feature detection, alignment, and differential
  screening
edam_operation: http://edamontology.org/operation_3215
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: MS-Dial
  role: Peak detection, deconvolution, feature extraction, and across-sample alignment
    of raw MS1 LC-HRMS files; version 4.80 is the compatible release cited in the
    CMDN repository
  repo: https://github.com/LeaveMeNotTonight/CMDN
- name: CMDN
  role: Downstream 'top-down' MS data processing framework whose DMS layer consumes
    the dms_feature_table.csv exported by this MS-Dial screening step
  repo: https://github.com/LeaveMeNotTonight/CMDN
- name: LC-HRMS
  role: Instrument platform generating the raw MS1 data from human liver S9 enzymatic
    biotransformation assays that MS-Dial processes
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/acs.est.5c08558
    title: Reaction-Guided Metabolomics Accelerates High-Throughput Annotation of
      Xenobiotic Metabolites for Human Exposome
  source_content_hash: sha256:fde392cdd08ddd2b619bf719ea1931bcce8d2978330cf57125eba27a9561564d
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/LeaveMeNotTonight/CMDN
  tools:
  - CMDN
  - LC-HRMS
  - MS-Dial
  edam_operation: http://edamontology.org/operation_3215
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# msdial-raw-data-processing

## Summary

Process raw MS1 LC-HRMS data in MS-Dial to detect, deconvolve, align metabolic features, and screen differentially expressed metabolites between an incubation group (PG) and a heat-inactivated control group (CG), forming the differential metabolite space (DMS) layer of the CMDN workflow.

## When to use

Use when raw MS1 LC-HRMS files from case/control incubation experiments (e.g., human liver S9 fractions with vs. without active enzyme) must be converted into an aligned feature-intensity matrix and filtered to differentially expressed metabolites.

## When NOT to use

- Input is already an aligned feature-intensity table (e.g., an existing m/z/RT feature matrix) rather than raw MS1 vendor or converted data files
- The goal is identification/annotation of metabolites (CMDN's reaction-based candidature space or MS2 similarity network layers) rather than differential-feature screening
- MS2 spectra acquisition or targeted quantification of known standards is needed, since this skill covers only MS1-level feature detection, alignment, and differential screening

## Inputs

- Raw MS1 LC-HRMS data files from human liver S9-incubation experiments (pesticide incubation group PG and heat-inactivated control group CG)
- MS-Dial ver. 4.80 with default peak-detection/deconvolution parameters

## Outputs

- dms_feature_table.csv containing retained differential features with m/z, retention time, intensities, fold-change (PG/CG), signal-to-noise, and p-value columns

## How to apply

Import raw MS1 LC-HRMS files from the human liver S9-incubation experiments into MS-Dial (ver. 4.80, as bundled with the CMDN R workflow) for peak detection, deconvolution, and feature extraction. Align detected features, defined by accurate m/z and retention time, across the pesticide-treated group (PG) and control group (CG) to build a single aligned feature-intensity matrix. For each feature, compute the PG/CG fold change, signal-to-noise ratio, and unadjusted Student's t-test p-value. Retain only features satisfying all three DMS thresholds — fold-change PG/CG ≥ 2, signal-to-noise ≥ 10, and unadjusted p ≤ 0.05 — then export the retained features (with m/z, RT, intensities, fold change, signal-to-noise, and p-value columns) to dms_feature_table.csv as input for downstream CMDN layers (RCS and SSN).

## Related tools

- **MS-Dial** (Peak detection, deconvolution, feature extraction, and across-sample alignment of raw MS1 LC-HRMS files; version 4.80 is the compatible release cited in the CMDN repository) — https://github.com/LeaveMeNotTonight/CMDN
- **CMDN** (Downstream 'top-down' MS data processing framework whose DMS layer consumes the dms_feature_table.csv exported by this MS-Dial screening step) — https://github.com/LeaveMeNotTonight/CMDN
- **LC-HRMS** (Instrument platform generating the raw MS1 data from human liver S9 enzymatic biotransformation assays that MS-Dial processes)

## Examples

```
Import raw MS1 files into MS-Dial ver. 4.80, align features by m/z/RT, then in R: dms <- read.csv("feature_table.csv"); retained <- subset(dms, fold_change_PG_CG >= 2 & signal_to_noise >= 10 & p_value <= 0.05); write.csv(retained, "dms_feature_table.csv", row.names = FALSE)
```

## Evaluation signals

- Every retained feature in dms_feature_table.csv satisfies all three DMS thresholds simultaneously: fold-change PG/CG ≥ 2, signal-to-noise ≥ 10, and unadjusted p ≤ 0.05
- The exported table carries the m/z and RT fields required by the downstream reaction-based candidature space
- The feature-intensity matrix is aligned across PG and CG samples using accurate m/z and retention time as the feature keys
- Exported columns include m/z, retention time, intensities, fold change, signal-to-noise ratio, and p-value
- No features were dropped that meet the thresholds, and no non-significant features remain (spot-check recomputing t-tests from the intensity matrix)

## Limitations

- The differential screening uses unadjusted Student's t-test p-values (p ≤ 0.05) without multiple-testing correction, so features may pass by chance at omics scale
- The DMS layer is a differential-features screen only; it does not annotate metabolite identities — identity assignment happens in CMDN's reaction-based pseudo-MS1 candidature space (RCS) and MS2 spectrum similarity network (SSN)
- Thresholds (FC ≥ 2, S/N ≥ 10, p ≤ 0.05) and the PG-vs-CG design are tuned for enzymatic biotransformation assays; other experimental designs may need recalibration

## Evidence

- [methods] In the DMS layer, raw MS1 files are imported into MSDial, metabolic features are aligned, and differentially expressed metabolites are screened by Student's t test using the criteria fold-change(PG/CG) ≥2 and signal-to-noise ratio ≥10.: [evidence span withheld — anchor status: not_found]
