---
name: lc-hrms-feature-detection-and-alignment
description: Use when raw MS1 LC-HRMS files from a case/control or treated/control
  design must be converted into an aligned m/z–RT feature-intensity table before differential
  screening, as in the CMDN DMS layer.
when_to_use_negative:
- Input is already an aligned feature table (e.g., an MS-Dial export) — skip directly
  to differential screening (t-test + fold-change/S-N thresholds)
- Goal is targeted confirmation of known metabolites with authentic/stable-isotopic-labeled
  standards rather than untargeted DMS feature discovery
- The study design lacks a matched control group (e.g., no heat-inactivated CG), making
  fold-change(PG/CG) and S/N screening undefined
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0218
tools:
- name: MS-Dial
  role: Peak detection, deconvolution, feature extraction, and cross-sample alignment
    of MS1 LC-HRMS features (ver. 4.80 used in CMDN)
  repo: github.com__LeaveMeNotTonight__CMDN
- name: CMDN
  role: Top-down untargeted MS data processing framework whose DMS layer consumes
    the aligned MS1 feature matrix; implemented as R scripts
  repo: github.com__LeaveMeNotTonight__CMDN
- name: LC-HRMS
  role: Data acquisition platform providing the raw MS1 files and accurate m/z and
    retention time used to define features
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/acs.est.5c08558
    title: Reaction-Guided Metabolomics Accelerates High-Throughput Annotation of
      Xenobiotic Metabolites for Human Exposome
  source_content_hash: sha256:cecfe280a6687113e846c45fe9d31ddc6bbbf78816c8d7a21d46637e25038cc6
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - CMDN
  - LC-HRMS
  - MS-Dial
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0218
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# LC-HRMS feature detection and alignment

## Summary

Detects, deconvolves, and aligns untargeted LC-HRMS MS1 features across sample groups (here, pesticide-incubation PG vs heat-inactivated control CG using human liver S9 assays) to build a single feature-intensity matrix that feeds the differential metabolite space (DMS) layer of the CMDN framework.

## When to use

Use when raw MS1 LC-HRMS files from a case/control or treated/control design must be converted into an aligned m/z–RT feature-intensity table before differential screening, as in the CMDN DMS layer.

## When NOT to use

- Input is already an aligned feature table (e.g., an MS-Dial export) — skip directly to differential screening (t-test + fold-change/S-N thresholds)
- Goal is targeted confirmation of known metabolites with authentic/stable-isotopic-labeled standards rather than untargeted DMS feature discovery
- The study design lacks a matched control group (e.g., no heat-inactivated CG), making fold-change(PG/CG) and S/N screening undefined

## Inputs

- Raw MS1 LC-HRMS data files from human liver S9-incubation experiments (PG and CG samples)
- Sample-group metadata labeling pesticide-treated (PG) vs heat-inactivated control (CG) runs

## Outputs

- Aligned feature-intensity matrix keyed by accurate m/z and retention time
- dms_feature_table.csv of retained differentially expressed features with m/z, retention time, intensities, fold-change(PG/CG), signal-to-noise, and p-value columns

## How to apply

Import the raw MS1 LC-HRMS files (from the human liver S9-incubation experiments) into MS-Dial (ver. 4.80) for peak detection, deconvolution, and feature extraction across samples. Align the detected features — defined by accurate m/z and retention time — across the pesticide-treated group (PG) and heat-inactivated control group (CG) to build a single aligned feature-intensity matrix, as implemented in the CMDN R workflow (tidyverse, CluMSID, OrgMassSpecR, etc.). For each aligned feature, compute the PG/CG fold change, signal-to-noise ratio, and unadjusted Student's t-test p-value. Retain only features passing all three DMS criteria: fold-change(PG/CG) ≥ 2, signal-to-noise ≥ 10, and unadjusted p ≤ 0.05, since these define the differential metabolite space (DMS) that feeds the reaction-based pseudo-MS1 candidature space (RCS) and MS2 spectrum similarity network (SSN). Export the retained features to dms_feature_table.csv with m/z, RT, intensities, fold change, signal-to-noise, and p-value columns; these m/z and RT fields are required by the downstream reaction-based candidature step.

## Related tools

- **MS-Dial** (Peak detection, deconvolution, feature extraction, and cross-sample alignment of MS1 LC-HRMS features (ver. 4.80 used in CMDN)) — github.com__LeaveMeNotTonight__CMDN
- **CMDN** (Top-down untargeted MS data processing framework whose DMS layer consumes the aligned MS1 feature matrix; implemented as R scripts) — github.com__LeaveMeNotTonight__CMDN
- **LC-HRMS** (Data acquisition platform providing the raw MS1 files and accurate m/z and retention time used to define features)

## Evaluation signals

- Every retained feature in dms_feature_table.csv satisfies all three DMS thresholds: fold-change(PG/CG) ≥ 2, signal-to-noise ≥ 10, and unadjusted p ≤ 0.05
- The table carries m/z and retention-time fields required by the downstream reaction-based candidature space (RCS)
- The aligned matrix is a single table covering both PG and CG samples, keyed by accurate m/z and RT rather than per-file peak lists
- The R environment contains the required packages (tidyverse, CluMSID, CluMSIDdata, OrgMassSpecR, pheatmap, reshape2, MSMSsim, msentropy, readxl) and MS-Dial ver. 4.80 was used for feature processing

## Limitations

- Thresholds (fold-change ≥ 2, S/N ≥ 10, p ≤ 0.05) are unadjusted (no multiple-testing correction) and were calibrated for pesticide-biotransformation assays, not general metabolomics
- The screening depends on the availability of a proper heat-inactivated control (CG) to estimate signal-to-noise; results differ if the control design changes
- Bottom-up style library dependence is avoided, but the top-down approach was motivated by the scarcity of authentic standards for xenobiotic metabolites — standard-free annotation still requires downstream validation (RT prediction, murine studies, time-course modeling)

## Evidence

- [intro] In the DMS layer, raw MS1 files are imported into MSDial, metabolic features are aligned, and differentially expressed metabolites are screened by Student's t test using the criteria fold-change(PG/CG) ≥2 and signal-to-noise ratio ≥10.: [evidence span withheld — anchor status: not_found]
- [readme] MSDial (ver. 4.80) and Biotransformer 3.0 are listed among the compatible software; before proceeding, ensure tidyverse, CluMSID, OrgMassSpecR, and related R packages are installed.: [evidence span withheld — anchor status: not_found]
