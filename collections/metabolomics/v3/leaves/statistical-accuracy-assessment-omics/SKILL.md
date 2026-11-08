---
name: statistical-accuracy-assessment-omics
description: Use when evaluating whether signal drift correction (e.g. QC-RFSC) actually
  improved differential feature calling or precision in LC-MS metabolomics/proteomics
  data.
when_to_use_negative:
- When data are already clean / no signal drift correction was applied — the comparison
  of raw vs. corrected significant feature counts is meaningless
- When the analysis goal is differential feature calling itself rather than judging
  the correction method's statistical benefit
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0154
- http://edamontology.org/topic_2269
tools:
- name: statTarget
  role: Provides shiftCor (QC-RFSC correction) and statAnalysis (Welch's T-test +
    FDR) used to generate raw vs. corrected feature matrices and differential feature
    counts
  repo: https://github.com/13479776/statTarget
- name: XCMS
  role: Produces the metabolite feature (peak) table that serves as input for statTarget
    correction and subsequent statistical comparison
- name: Skyline
  role: Produces the tryptic peptide feature table for proteomics data that undergoes
    the same correction and comparison
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1016/j.aca.2018.08.002
    title: 'statTarget: A streamlined tool for signal drift correction and interpretations
      of quantitative mass spectrometry-based omics data'
  source_content_hash: sha256:e5df2d3e457bbba1c9da73350977679f26d4bd7035b10151f1511c2ba2d72a2f
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/13479776/statTarget
  tools:
  - Skyline
  - XCMS
  - statTarget
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0154
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# statistical-accuracy-assessment-omics

## Summary

Quantifies the statistical benefit of QC-RFSC signal drift correction in statTarget by comparing the number of significant features (Welch's T-test with Benjamini-Hochberg FDR) in raw versus corrected metabolomics/proteomics feature matrices, and by tracking how many peaks move under precision RSD thresholds (15% / 30%).

## When to use

Use when evaluating whether signal drift correction (e.g. QC-RFSC) actually improved differential feature calling or precision in LC-MS metabolomics/proteomics data.

## When NOT to use

- When data are already clean / no signal drift correction was applied — the comparison of raw vs. corrected significant feature counts is meaningless
- When the analysis goal is differential feature calling itself rather than judging the correction method's statistical benefit

## Inputs

- Raw feature matrix from XCMS or Skyline peak extraction (metabolomics or label-free LC-MS proteomics)
- Sample-to-QC annotation file (samPeno) mapping injection order and batch
- statTarget shiftCor and statAnalysis output matrices (Before_shiftCor / After_shiftCor)
- RSD distribution table per feature in raw and corrected data

## Outputs

- Count of significant/differential features in raw vs. QC-RFSC corrected data per dataset (metabolomics and proteomics), with fold enlargement factor
- Proportion of peaks meeting RSD precision thresholds (15% and 30%) in raw and corrected data
- Comparative precision metrics against QC-RLSC and QC-SVR (e.g. 65.5% vs 61.9% for 30% RSD in Dataset 1; 80.1% vs 72.3% in Dataset 2)

## How to apply

Run statTarget's shiftCor (MLmethod = 'QCRFSC', Frule = 0.8, imputeM = 'KNN') on the feature matrix derived from XCMS or Skyline peak tables to obtain a corrected output, then run statAnalysis (Frule = 0.8, normM = 'NONE', imputeM = 'KNN', glog = TRUE, scaling = 'Pareto') on both raw and corrected matrices with Welch's T-test and Benjamini-Hochberg FDR. Count significant features in each; report fold enlargement of the differential feature count after correction. Complement this with the cumulative RSD distribution: track the proportion of peaks with RSDs < 15% and < 30% in raw data versus after QC-RFSC, and compare against competing algorithms such as QC-RLSC and QC-SVR.

## Related tools

- **statTarget** (Provides shiftCor (QC-RFSC correction) and statAnalysis (Welch's T-test + FDR) used to generate raw vs. corrected feature matrices and differential feature counts) — https://github.com/13479776/statTarget
- **XCMS** (Produces the metabolite feature (peak) table that serves as input for statTarget correction and subsequent statistical comparison)
- **Skyline** (Produces the tryptic peptide feature table for proteomics data that undergoes the same correction and comparison)

## Examples

```
shiftCor(samPeno, samFile, Frule = 0.8, MLmethod = "QCRFSC", QCspan = 0, imputeM = "KNN"); statAnalysis(file, Frule = 0.8, normM = "NONE", imputeM = "KNN", glog = TRUE, scaling = "Pareto")
```

## Evaluation signals

- Corrected data should show a substantially larger number of significant features than raw data, consistent with the reported 12.7-fold increase in peaks with 15% RSDs for Dataset 1
- Proportion of peaks with RSDs < 30% should rise markedly after QC-RFSC (e.g. 43.9% → 86.2% in Dataset 1; 52.1% → 90.9% in Dataset 2), and exceed what QC-RLSC or QC-SVR achieve
- statAnalysis parameters must be identical (Frule = 0.8, imputeM = 'KNN', glog = TRUE, scaling = 'Pareto') across raw and corrected runs for a fair comparison
- Enlargement of the differential feature count should be reproducible across datasets (metabolomics Datasets 1 & 2 and proteomics Dataset 3)

## Limitations

- statAnalysis defaults (normM = 'NONE', glog = TRUE, scaling = 'Pareto') must match across raw/corrected runs; otherwise the statistical comparison is confounded
- Precision gains (RSD improvement) depend on QC injection frequency in the batch (e.g. every 5 real samples in Dataset 1, every 6 in Dataset 2, every 8 in Dataset 3)
- The comparison of correction methods (QC-RFSC vs QC-RLSC vs QC-SVR) is specific to these datasets; performance is not guaranteed for other omics or acquisition schemes

## Evidence
