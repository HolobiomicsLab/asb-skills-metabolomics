---
name: calibration-curve-limit-of-detection-estimation
description: Use when serial-diluted standard mixture data (e.g., 134-metabolite standard,
  HILIC(−)/RP(+)) must be quantified as per-metabolite LODs to compare two feature
  tables such as full JPA vs PP-only extraction.
when_to_use_negative:
- Data are not from a serial dilution series with known concentration levels — calibration
  curves cannot be fit without graded inputs
- You only need feature detection/annotation, not quantitative LOD comparison — use
  plain JPA extraction functions instead
- Processing full-scan or DIA data where MS2 recognition (MR) features are invalid
  — the README explicitly warns not to use find.level3features in those cases
edam_operation: http://edamontology.org/operation_3443
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: JPA
  role: R package performing PP/MR/targeted feature extraction and peak.alignment
    on the dilution series, supplying the intensity matrices from which calibration
    curves and LODs are computed
  repo: https://github.com/HuanLab/JPA.git
- name: R
  role: Runtime environment (>= 4.0.0) in which JPA is installed via devtools and
    calibration/LOD calculations are scripted
- name: xcms
  role: Embedded peak-picking functions inside JPA that generate the PP-only feature
    table used as the conventional-processing baseline for LOD comparison
- name: devtools
  role: Used to install JPA from GitHub (install_github("HuanLab/JPA"))
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.3390/metabo12030212
    title: 'JPA: Joint Metabolic Feature Extraction Increases the Depth of Chemical
      Coverage for LC-MS-Based Metabolomics and Exposomics'
  source_content_hash: sha256:d9dc9c43970ad7a80ec74d9bb5dd033ac68adea990a5b994b4fbf1123b22d06b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/HuanLab/JPA.git
  tools:
  - JPA
  - R
  - devtools
  - xcms
  edam_operation: http://edamontology.org/operation_3443
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# Calibration curve limit-of-detection estimation

## Summary

Builds per-metabolite calibration curves from aligned intensities of serial-diluted standard mixture LC-MS data to compute limits of detection (LODs), and compares LODs across feature extraction strategies (e.g., full JPA vs conventional peak picking alone) as LOD ratios.

## When to use

Use when serial-diluted standard mixture data (e.g., 134-metabolite standard, HILIC(−)/RP(+)) must be quantified as per-metabolite LODs to compare two feature tables such as full JPA vs PP-only extraction.

## When NOT to use

- Data are not from a serial dilution series with known concentration levels — calibration curves cannot be fit without graded inputs
- You only need feature detection/annotation, not quantitative LOD comparison — use plain JPA extraction functions instead
- Processing full-scan or DIA data where MS2 recognition (MR) features are invalid — the README explicitly warns not to use find.level3features in those cases

## Inputs

- Serial-diluted metabolite standard mixture mzXML files in HILIC(−) and RP(+) modes
- In-house standard metabolite library in CSV format (e.g., LibraryCSVHILIC-.csv) with targeted m/z and retention times
- Aligned JPA feature tables (PP-only and full-JPA runs) with maxo intensities per sample

## Outputs

- Per-metabolite LOD values for the PP-only and full-JPA feature tables
- Per-metabolite LOD ratios (PP-only LOD / full-JPA LOD)
- lod_comparison.csv summarizing LOD comparisons

## How to apply

Process the serial-dilution mzXML files twice: once with conventional peak picking only (XCMS.featureTable with mz.tol=10, ppm=10, peakwidth=c(5,20), mzdiff=0.01, snthresh=6, integrate=1, prefilter=c(3,100), noise=100) and once with the full JPA pipeline adding MR features via find.level3features(level3.threshold=2) and targeted features via find.targetfeatures(target.threshold=2) against the in-house standard library CSV. Align both feature tables across the dilution series with peak.alignment(bw=5, minfrac=0.5, mzwid=0.015, minsamp=1, max=100, quantitative.method="maxo"). For each standard metabolite, match features on m/z and retention time from the targeted list, fit a calibration curve from aligned intensities (maxo) across dilution levels, and derive the per-metabolite LOD for each table. Finally compute the per-metabolite LOD ratio (PP-only LOD / full-JPA LOD) and export as lod_comparison.csv; correct application reproduces ratios in the thousands-of-fold range (up to 3425× in HILIC(−) and 15,074× in RP(+)).

## Related tools

- **JPA** (R package performing PP/MR/targeted feature extraction and peak.alignment on the dilution series, supplying the intensity matrices from which calibration curves and LODs are computed) — https://github.com/HuanLab/JPA.git
- **R** (Runtime environment (>= 4.0.0) in which JPA is installed via devtools and calibration/LOD calculations are scripted)
- **xcms** (Embedded peak-picking functions inside JPA that generate the PP-only feature table used as the conventional-processing baseline for LOD comparison)
- **devtools** (Used to install JPA from GitHub (install_github("HuanLab/JPA")))

## Examples

```
library(JPA); ft_PP <- XCMS.featureTable(dir, mz.tol=10, ppm=10, peakwidth=c(5,20), mzdiff=0.01, snthresh=6, integrate=1, prefilter=c(3,100), noise=100); ft_JPA <- find.targetfeatures(data=MSdata, tarFTdir=tarFTdir, tarFTname="LibraryCSVHILIC-.csv", target.threshold=2); ft_JPA <- peak.alignment(data=MSdata, bw=5, minfrac=0.5, mzwid=0.015, minsamp=1, max=100, quantitative.method="maxo")
```

## Evaluation signals

- lod_comparison.csv contains one row per standard metabolite with both PP-only and full-JPA LODs and a finite positive ratio
- Computed LOD ratios reproduce the reported order of magnitude: up to 3425-fold lower LODs in HILIC(−) and up to 15,074-fold lower in RP(+) for JPA vs PP-only
- Aligned intensities use maxo as the quantitative method (quantitative.method="maxo") and features are matched to standards by both m/z and retention time from the targeted list
- Both runs share identical peak-picking and alignment parameters so LOD differences are attributable to extraction strategy, not parameterization

## Limitations

- Conventional feature extraction fails to recognize features with low signal intensities, poor chromatographic peak shapes, or those not fitting parameter settings, which inflates PP-only LODs relative to JPA
- The MR feature step must not be used when processing full-scan or DIA datasets
- The MR threshold should be estimated beforehand with the thresholdEstimate.R script provided on the GitHub repository

## Evidence

- [abstract] JPA achieved LODs up to thousands of folds lower when processing serial diluted metabolite standard mixture data in HILIC(-) and RP(+) modes: "JPA was able to achieve a limit of detection (LOD) that was up to thousands of folds lower when automatically processing metabolomics data of a serial diluted metabolite standard mixture analyzed in"
- [abstract] Conventional extraction fails on low-signal features: "Conventional feature extraction algorithms fail to recognize features with low signal intensities, poor chromatographic peak shapes, or those that do not fit the parameter settings."
- [methods] LOD computed from aligned maxo intensities after peak.alignment: "featureTable <- peak.alignment(data = MSdata, bw = 5, minfrac = 0.5, mzwid = 0.015,"
- [methods] In-house standard library CSV supplies targeted m/z and retention times: [evidence span withheld — anchor status: not_found]
- [readme] PP features extracted with embedded XCMS functions and stated parameters: "featureTable <- XCMS.featureTable(dir = dir, mz.tol = 10, ppm=10, peakwidth=c(5,20), mzdiff = 0.01, snthresh = 6, integrate = 1, prefilter = c(3,100), noise = 100)"
- [readme] Sample alignment performed after all level features extracted: "Note: for multi-sample analysis, sample alignment is performed after feature extraction."
