---
name: srn-proteomics-signal-drift-correction
description: Use when an SRM or label-free LC-MS proteomics/omics feature table (e.g.
  Skyline transition intensities with pooled QC injections) shows inter-batch signal
  drift that must be corrected before downstream statistical analysis.
when_to_use_negative:
- When the dataset lacks pooled quality-control samples injected at regular intervals
  — QC-RFSC requires QC-anchored regression and statTarget instead offers Combat via
  shiftCor_dQC for QC-free datasets.
- When the input is already a drift-corrected or post-integration matrix; re-running
  correction on corrected data is not covered by the method.
- When the goal is only statistical analysis (grouping, univariate testing) without
  signal correction; statAnalysis handles those steps independently.
edam_operation: http://edamontology.org/operation_3218
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_3520
tools:
- name: statTarget
  role: Provides shiftCor, transX and statAnalysis functions implementing the QC-RFSC
    algorithm and evaluation workflow for signal drift correction
  repo: https://github.com/13479776/statTarget
- name: Skyline
  role: Source of SRM transition peak abundances for proteomics feature extraction
    before correction
- name: ProteoWizard
  role: Converts MS raw data files to mzXML format prior to feature extraction
- name: random forest
  role: Underlying regression algorithm used by QC-RFSC (ntree = 500) to model signal
    drift per feature using QC samples
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.aca.2018.08.002
    title: 'statTarget: A streamlined tool for signal drift correction and interpretations
      of quantitative mass spectrometry-based omics data'
  source_content_hash: sha256:117ab495f643440449c2e6bad0e81fcaef9c4b107fc22d04e8f0a646b2340965
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/13479776/statTarget
  tools:
  - ProteoWizard
  - Skyline
  - random forest
  - statTarget
  edam_operation: http://edamontology.org/operation_3218
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# srn-proteomics-signal-drift-correction

## Summary

Apply quality control-based random forest signal correction (QC-RFSC) to SRM proteomics feature tables to remove inter- and intra-batch signal drift, evaluated via RSD cumulative distributions and pre/post-correction PCA.

## When to use

Use when an SRM or label-free LC-MS proteomics/omics feature table (e.g. Skyline transition intensities with pooled QC injections) shows inter-batch signal drift that must be corrected before downstream statistical analysis.

## When NOT to use

- When the dataset lacks pooled quality-control samples injected at regular intervals — QC-RFSC requires QC-anchored regression and statTarget instead offers Combat via shiftCor_dQC for QC-free datasets.
- When the input is already a drift-corrected or post-integration matrix; re-running correction on corrected data is not covered by the method.
- When the goal is only statistical analysis (grouping, univariate testing) without signal correction; statAnalysis handles those steps independently.

## Inputs

- Skyline- or XCMS-extracted proteomics feature table (transitions x samples) with pooled QC injections interspersed throughout the run
- Sample metadata table mapping sample names to injection order, batch, and phenotype (e.g. PD patient vs control) as required by shiftCor
- statTarget-formatted input files produced by transX from Skyline, MZmine2, SIEVE or XCMS output

## Outputs

- QC-RFSC corrected feature/sample matrix with inter- and intra-batch signal drift removed
- Raw (uncorrected) matrix retained for comparison
- RSD% cumulative frequency distribution plots for raw, QC-RFSC, QC-RLSC and QC-SVR corrected data
- PCA score plots (pre/post-correction) demonstrating batch-effect removal

## How to apply

Load the proteomics feature table (e.g. 174 transitions x 69 proteins from Skyline, with pooled QC injections every eight subject samples) using the statTarget transX function to generate the required input format, then run shiftCor with Frule = 0.8 (80-percent rule) and imputeM = 'KNN' to filter low-quality peaks and impute missing values before correction. Apply QC-RFSC (MLmethod = 'QCRFSC', ntree = 500) which trains a random forest regression per feature using QC samples ordered by injection order, then uses the fitted model to predict correction factors for every sample, removing inter- and intra-batch unwanted variation at the feature level. Retain the raw matrix alongside the corrected one, then quantify improvement by computing the relative standard deviation (RSD%) of each transition in QC samples and plotting cumulative frequency distributions, and by running PCA (statAnalysis with Pareto scaling and glog transform) on raw versus corrected matrices to confirm batch-effect removal. Compare against QC-RLSC and QC-SVR baselines; on the paper's Dataset 3, QC-RFSC raised the fraction of peaks with RSD < 30% to 87.9% versus 68.4% (QC-RLSC) and 47.7% (QC-SVR), so QC-RFSC is the preferred method.

## Related tools

- **statTarget** (Provides shiftCor, transX and statAnalysis functions implementing the QC-RFSC algorithm and evaluation workflow for signal drift correction) — https://github.com/13479776/statTarget
- **Skyline** (Source of SRM transition peak abundances for proteomics feature extraction before correction)
- **ProteoWizard** (Converts MS raw data files to mzXML format prior to feature extraction)
- **random forest** (Underlying regression algorithm used by QC-RFSC (ntree = 500) to model signal drift per feature using QC samples)

## Examples

```
shiftCor(samPeno,samFile, Frule = 0.8, MLmethod = "QCRFSC", QCspan = 0, imputeM = "KNN")
```

## Evaluation signals

- Compute RSD% of every transition in QC samples before and after correction; expect a large increase in the proportion of peaks with RSD < 30% (paper reported 87.9% for Dataset 3 after QC-RFSC).
- Generate cumulative frequency distribution curves of RSD% for raw vs corrected data and confirm the corrected curve lies above the raw one.
- Run PCA on raw and corrected matrices with Pareto scaling and glog transform; confirm corrected PCA score plot shows reduced batch separation and preserved biological grouping (PD vs control).
- Compare QC-RFSC against QC-RLSC and QC-SVR baselines using the same RSD metric to verify superiority (e.g. 87.9% vs 68.4% vs 47.7%).
- Verify that shiftCor with Frule = 0.8 removed features failing the 80-percent rule and that KNN imputation filled remaining missing values before model fitting.

## Limitations

- Requires regular pooled QC injections within the batch; the correction cannot be applied to datasets without QC samples (statTarget provides Combat via shiftCor_dQC only for QC-free data).
- Performance depends on QC sampling frequency and the quality of the injected QC material; datasets with very few QC points or high QC variability may yield unstable random forest predictions.
- Signal correction operates at the feature level and does not correct instrument-wide retention-time drift; complementary peak-alignment steps may still be needed.

## Evidence
