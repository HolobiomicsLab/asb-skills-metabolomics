---
name: differential-metabolite-feature-detection
description: Use when raw MS1 LC-HRMS data from paired treated vs. heat-inactivated
  control incubations (e.g., 1,021 pesticides in human liver S9) must be feature-detected,
  aligned, and differentially screened to feed reaction-guided metabolite annotation.
when_to_use_negative:
- The input is already an aligned feature table or only MS2/PRM data without paired
  PG/CG MS1 acquisitions — proceed directly to the RCS or SSN layers instead
- The goal is targeted quantification of known parent compounds rather than discovery
  of unannotated biotransformation-derived features
- Samples lack a heat-inactivated (or otherwise enzymatically inactivated) control
  group, making the differential PG/CG fold-change screening undefined
edam_operation: http://edamontology.org/operation_3214
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: MS-Dial
  role: Peak detection, deconvolution, feature extraction, and cross-sample alignment
    of MS1 LC-HRMS features (ver. 4.80) prior to differential screening
  repo: https://github.com/LeaveMeNotTonight/CMDN
- name: CMDN
  role: The 'top-down' framework whose DMS layer wraps the differential screening
    and passes retained features to the RCS and SSN layers; R scripts run via tidyverse,
    CluMSID, OrgMassSpecR, pheatmap, reshape2, MSMSsim, msentropy, readxl
  repo: https://github.com/LeaveMeNotTonight/CMDN
- name: LC-HRMS
  role: Liquid chromatography-coupled high-resolution mass spectrometry acquisition
    producing the raw MS1 files that are the DMS input
provenance:
  source_task_ids:
  - task_004
  - task_001
  source_papers:
  - doi: 10.1021/acs.est.5c08558
    title: Reaction-Guided Metabolomics Accelerates High-Throughput Annotation of
      Xenobiotic Metabolites for Human Exposome
  source_content_hash: sha256:00e56911b0342e2af92812f06f2be80d25fc95e3f1cb019a68264a8b73bbe752
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
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# differential-metabolite-feature-detection

## Summary

The differential metabolite space (DMS) layer of the CMDN 'top-down' framework: peak detection, feature alignment, and fold-change/SNR/t-test screening of LC-HRMS MS1 features between pesticide-incubated and heat-inactivated control samples, to isolate biotransformation-derived features for downstream CMDN layers.

## When to use

Use when raw MS1 LC-HRMS data from paired treated vs. heat-inactivated control incubations (e.g., 1,021 pesticides in human liver S9) must be feature-detected, aligned, and differentially screened to feed reaction-guided metabolite annotation.

## When NOT to use

- The input is already an aligned feature table or only MS2/PRM data without paired PG/CG MS1 acquisitions — proceed directly to the RCS or SSN layers instead
- The goal is targeted quantification of known parent compounds rather than discovery of unannotated biotransformation-derived features
- Samples lack a heat-inactivated (or otherwise enzymatically inactivated) control group, making the differential PG/CG fold-change screening undefined

## Inputs

- Raw MS1 LC-HRMS data files (.d/.mzML/vendor formats) from human liver S9 enzymatic incubations of the pesticide-treated group (PG)
- Raw MS1 LC-HRMS data files from the matched heat-inactivated control group (CG)
- Sample metadata mapping MS1 files to PG/CG groups

## Outputs

- Aligned feature-intensity matrix (m/z, retention time, per-sample intensities)
- dms_feature_table.csv — differentially expressed features with m/z, RT, intensities, fold change, signal-to-noise, and p-value columns
- Count of upregulated features (expected 793 of 32,827 total aligned features in the reference run)

## How to apply

Import raw MS1 LC-HRMS files from the pesticide-treated group (PG) and heat-inactivated control group (CG) into MS-Dial (ver. 4.80) for peak detection, deconvolution, and feature extraction; align features across samples by accurate m/z and retention time into a single feature-intensity matrix. For each aligned feature compute the PG/CG fold change, signal-to-noise ratio, and unadjusted p-value via Student's t test between groups. Retain only features meeting all three DMS thresholds — fold-change PG/CG >= 2, S/N >= 10, and unadjusted p <= 0.05 — as the input space for the downstream reaction-based pseudo-MS1 candidature space (RCS). In the reference application this screening reduced 32,827 total aligned features to 793 upregulated features. The retained table must preserve m/z and RT fields because the RCS layer matches theoretical reaction-derived masses against these values.

## Related tools

- **MS-Dial** (Peak detection, deconvolution, feature extraction, and cross-sample alignment of MS1 LC-HRMS features (ver. 4.80) prior to differential screening) — https://github.com/LeaveMeNotTonight/CMDN
- **CMDN** (The 'top-down' framework whose DMS layer wraps the differential screening and passes retained features to the RCS and SSN layers; R scripts run via tidyverse, CluMSID, OrgMassSpecR, pheatmap, reshape2, MSMSsim, msentropy, readxl) — https://github.com/LeaveMeNotTonight/CMDN
- **LC-HRMS** (Liquid chromatography-coupled high-resolution mass spectrometry acquisition producing the raw MS1 files that are the DMS input)

## Examples

```
install.packages(c("tidyverse","CluMSID","CluMSIDdata","OrgMassSpecR","pheatmap","reshape2","MSMSsim","msentropy","readxl"))  # then run MS-Dial ver. 4.80 on PG/CG MS1 files and export dms_feature_table.csv
```

## Evaluation signals

- Every row of dms_feature_table.csv satisfies all three thresholds: fold-change PG/CG >= 2, S/N >= 10, and unadjusted p <= 0.05
- The retained table carries m/z and RT columns required by the downstream reaction-based candidature space (RCS)
- On the reference 1,021-pesticide human liver S9 dataset, the pipeline reports 32,827 total aligned features and 793 upregulated features after DMS screening
- Feature alignment is defined by accurate m/z + retention time, so duplicate features sharing both values are collapsed into single entries
- Downstream RCS reproduction (6,181 MS1 candidate matches, 2,886 nonredundant annotations) depends on this table, so mismatches there indicate DMS-layer errors

## Limitations

- Differential screening uses unadjusted p-values from Student's t test; no multiple-testing correction is reported, so weakly supported features may pass
- The S/N >= 10 and fold-change >= 2 thresholds were tuned for enzymatic vs. heat-inactivated S9 incubations and may not transfer to other matrix designs
- In silico prediction alone (biotransformation/fragmentation tools) 'can provide partial insights, yet often fall short of providing direct evidence under real-world, omics-scale conditions' — the DMS layer supplies the experimental contrast but is still only the first of three CMDN layers

## Evidence

- [other] DMS layer screening criteria: [evidence span withheld — anchor status: not_found]
