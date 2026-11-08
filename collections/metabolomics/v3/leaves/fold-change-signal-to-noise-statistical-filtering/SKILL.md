---
name: fold-change-signal-to-noise-statistical-filtering
description: Use when you have raw MS1 LC-HRMS feature-intensity data from a treated-vs-inactivated-control
  incubation design (e.g., pesticide vs heat-inactivated S9) and need to isolate biotransformation-derived
  differential features before reaction-based annotation.
when_to_use_negative:
- Data has no treated-vs-inactivated-control contrast (e.g., single-group profiling
  or human cohort screening), since fold-change(PG/CG) and SNR cannot be computed
  as defined
- The goal is MS2-level annotation or spectrum networking — those belong to the CMDN
  SSN layer, not DMS filtering
- You are running a traditional 'bottom-up' workflow relying on reference spectral
  libraries rather than top-down differential screening of unknown biotransformed
  metabolites
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: MS-Dial
  role: Imports raw MS1 files and performs peak detection, deconvolution, feature
    extraction, and cross-sample alignment by m/z and RT (ver. 4.80 used)
- name: CMDN
  role: Top-down untargeted MS data processing framework whose first (DMS) layer implements
    this fold-change/SNR/t-test screening before the RCS and SSN layers
  repo: https://github.com/LeaveMeNotTonight/CMDN
- name: R (tidyverse)
  role: Scripting environment for computing fold change, SNR, and Student's t test
    and exporting the DMS feature table (README lists tidyverse among required packages)
  repo: https://github.com/LeaveMeNotTonight/CMDN
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/acs.est.5c08558
    title: Reaction-Guided Metabolomics Accelerates High-Throughput Annotation of
      Xenobiotic Metabolites for Human Exposome
  source_content_hash: sha256:2e1ac9a9274a0f59ef81052f86300a75ac07046dc3c60bbef951a306b52b40ed
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/LeaveMeNotTonight/CMDN
  tools:
  - CMDN
  - MS-Dial
  - R (tidyverse)
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# fold-change-signal-to-noise-statistical-filtering

## Summary

The differential metabolite space (DMS) screening step of the CMDN framework: MS1 LC-HRMS features from human liver S9 incubations are aligned in MS-Dial and differentially expressed metabolites between the pesticide group (PG) and heat-inactivated control group (CG) are retained via unadjusted Student's t test with fold-change(PG/CG) ≥ 2, signal-to-noise ≥ 10, and p ≤ 0.05.

## When to use

Use when you have raw MS1 LC-HRMS feature-intensity data from a treated-vs-inactivated-control incubation design (e.g., pesticide vs heat-inactivated S9) and need to isolate biotransformation-derived differential features before reaction-based annotation.

## When NOT to use

- Data has no treated-vs-inactivated-control contrast (e.g., single-group profiling or human cohort screening), since fold-change(PG/CG) and SNR cannot be computed as defined
- The goal is MS2-level annotation or spectrum networking — those belong to the CMDN SSN layer, not DMS filtering
- You are running a traditional 'bottom-up' workflow relying on reference spectral libraries rather than top-down differential screening of unknown biotransformed metabolites

## Inputs

- Raw MS1 LC-HRMS data files from human liver S9-incubation experiments (pesticide-treated PG and heat-inactivated CG)
- Aligned MS-Dial feature-intensity matrix keyed by accurate m/z and retention time

## Outputs

- dms_feature_table.csv with columns: m/z, retention time, PG/CG intensities, fold change, signal-to-noise ratio, unadjusted t-test p-value
- Set of differentially expressed features forming the input space for the downstream RCS and SSN CMDN layers

## How to apply

Import raw MS1 LC-HRMS files from the human liver S9 incubation experiments into MS-Dial (ver. 4.80) for peak detection, deconvolution, and feature extraction, then align features by accurate m/z and retention time into a single feature-intensity matrix spanning PG and CG. For each aligned feature, compute the fold change of PG vs CG intensities, the signal-to-noise ratio, and the unadjusted Student's t test p-value for the between-group comparison. Retain only features passing all three DMS criteria: fold-change(PG/CG) ≥ 2, signal-to-noise ≥ 10, and unadjusted p ≤ 0.05; this 'top-down' differential filter removes endogenous/untreated signals so downstream CMDN layers (reaction-based pseudo-MS1 candidature space and MS2 spectrum similarity network) only evaluate candidate biotransformed derivatives. Export retained features with m/z, RT, intensities, fold change, signal-to-noise, and p-value columns to dms_feature_table.csv.

## Related tools

- **MS-Dial** (Imports raw MS1 files and performs peak detection, deconvolution, feature extraction, and cross-sample alignment by m/z and RT (ver. 4.80 used))
- **CMDN** (Top-down untargeted MS data processing framework whose first (DMS) layer implements this fold-change/SNR/t-test screening before the RCS and SSN layers) — https://github.com/LeaveMeNotTonight/CMDN
- **R (tidyverse)** (Scripting environment for computing fold change, SNR, and Student's t test and exporting the DMS feature table (README lists tidyverse among required packages)) — https://github.com/LeaveMeNotTonight/CMDN

## Examples

```
library(tidyverse); dms <- read.csv("dms_feature_table.csv"); keep <- filter(dms, fold_change_PG_CG >= 2, SNR >= 10, p_value <= 0.05)
```

## Evaluation signals

- Every retained feature in dms_feature_table.csv satisfies all three DMS thresholds: fold-change PG/CG ≥ 2, signal-to-noise ≥ 10, and unadjusted p ≤ 0.05
- dms_feature_table.csv carries the m/z and RT fields required by the downstream reaction-based candidature space (RCS)
- p-values are unadjusted Student's t test results from the PG-vs-CG between-group comparison, not multiple-testing-corrected values
- The aligned feature matrix covers both PG and CG samples with features defined by consistent accurate m/z and retention time before filtering
- Dropped features are those failing any single criterion (e.g., high fold change but SNR < 10 must be excluded)

## Limitations

- p-values are unadjusted (no multiple-testing correction applied), so the filter trades statistical stringency for discovery breadth in a 'top-down' framework
- Thresholds (fold-change ≥ 2, SNR ≥ 10, p ≤ 0.05) were calibrated for the human liver S9 pesticide-incubation PG/CG design and may need re-derivation for other matrices or designs
- Requires MS1-level aligned intensities with computable signal-to-noise; features lacking sufficient replicate data in PG or CG cannot be screened

## Evidence

- [other] differentially expressed metabolites are screened by Student's t test using the criteria fold-change(PG/CG) ≥2 and signal-to-noise ratio ≥10: [evidence span withheld — anchor status: not_found]
