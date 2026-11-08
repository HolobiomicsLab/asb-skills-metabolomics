---
name: lc-ms-ms2-feature-recognition
description: Use when processing DDA LC-MS metabolomics mzXML files where conventional
  peak picking (XCMS-style) misses low-abundance or bad peak-shape features, and you
  want to quantify or recover those lost features via their MS2 spectra.
when_to_use_negative:
- Do not use find.level3features when processing full-scan or DIA data sets — MR extraction
  requires DDA data with MS2 spectra.
- Do not use when the goal is multi-sample alignment-driven analysis without per-sample
  rescue quantification — alignment (peak.alignment) applies only after feature extraction
  across samples.
- Do not use when features already exist in the table (PP or MR); duplicates are not
  re-added.
edam_operation: http://edamontology.org/operation_3216
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: JPA
  role: R package providing find.level3features() for MS2-recognition (MR) feature
    extraction and XCMS.featureTable() for baseline PP extraction
  repo: https://github.com/HuanLab/JPA
- name: R
  role: Runtime environment; JPA requires R version 4.0.0 or above
- name: xcms
  role: Embedded peak picking functions used inside XCMS.featureTable() to generate
    baseline PP features
- name: MSnbase
  role: Object class holding raw MS data and MS2 spectra passed into find.level3features()
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.3390/metabo12030212
    title: 'JPA: Joint Metabolic Feature Extraction Increases the Depth of Chemical
      Coverage for LC-MS-Based Metabolomics and Exposomics'
  source_content_hash: sha256:1bbc76c973f4361be340854679c99884c689d5717c1cafe87dff3a44bd955adf
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/HuanLab/JPA
  tools:
  - JPA
  - MSnbase
  - R
  - xcms
  edam_operation: http://edamontology.org/operation_3216
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# lc-ms-ms2-feature-recognition

## Summary

Rescue metabolic features missed by conventional peak picking by detecting features with poor chromatographic peak shapes that nonetheless have MS2 spectra, using JPA's MS2 recognition (MR) strategy on raw LC-MS mzXML data.

## When to use

Use when processing DDA LC-MS metabolomics mzXML files where conventional peak picking (XCMS-style) misses low-abundance or bad peak-shape features, and you want to quantify or recover those lost features via their MS2 spectra.

## When NOT to use

- Do not use find.level3features when processing full-scan or DIA data sets — MR extraction requires DDA data with MS2 spectra.
- Do not use when the goal is multi-sample alignment-driven analysis without per-sample rescue quantification — alignment (peak.alignment) applies only after feature extraction across samples.
- Do not use when features already exist in the table (PP or MR); duplicates are not re-added.

## Inputs

- Raw LC-MS mzXML file(s) (any vendor format compatible with MS-Convert), placed in one folder with no other irrelevant mzXML files
- level3.threshold value confirmed via the thresholdEstimate.R helper script

## Outputs

- A dataframe-format featureTable with 'level' column distinguishing PP (peak-picked) and MR (MS2-recognition) features
- Per-sample rescue percentage (MR features / total features)
- MSnbase object carrying the MS2 spectra

## How to apply

First build the baseline feature table with JPA's XCMS.featureTable() (e.g., ppm=10, peakwidth=c(5,20), mzdiff=0.01, snthresh=6, prefilter=c(3,100), noise=100), which labels conventional peak-picked features as 'PP'. Then, for each sample, run find.level3features(data=MSdata, level3.threshold=2) to append MR features — features with MS2 spectra but bad peak shapes that peak picking missed; the true-positive threshold can be confirmed with the thresholdEstimate.R helper from the GitHub repo. Do not run peak.alignment when handling single-sample feature tables (alignment is only for multi-sample analysis); instead, count PP-only vs newly added MR features per sample and compute the rescue percentage as MR features / total features from the 'level' column. Validate against reported benchmarks: an average of ~25% features rescued across serial diluted urine samples, ~5% rescue in the most dilute samples, and ~40% feature increase for the original-concentration RP(+) sample.

## Related tools

- **JPA** (R package providing find.level3features() for MS2-recognition (MR) feature extraction and XCMS.featureTable() for baseline PP extraction) — https://github.com/HuanLab/JPA
- **R** (Runtime environment; JPA requires R version 4.0.0 or above)
- **xcms** (Embedded peak picking functions used inside XCMS.featureTable() to generate baseline PP features)
- **MSnbase** (Object class holding raw MS data and MS2 spectra passed into find.level3features())

## Examples

```
featureTable <- find.level3features(data = MSdata, level3.threshold = 2)
```

## Evaluation signals

- Average rescue fraction across serial diluted urine dilution levels ≈ 25% (MR features / total features)
- Rescue fraction in the most dilute urine samples ≈ 5%
- Original-concentration RP(+) urine sample shows ~40% feature increase from MR rescue
- featureTable 'level' column contains only 'PP' and 'MR' labels; MR rows only appear after find.level3features() runs
- thresholdEstimate.R-derived threshold (e.g., level3.threshold=2) yields MR features whose peak shapes and MS2 support are judged true-positive

## Limitations

- MR extraction must not be run on full-scan or DIA data sets (README Part 3 warning).
- Alignment must be skipped for single-sample analyses; alignment is only meaningful after extracting level features across multiple samples.
- Rescue performance depends on the MS2-recognition threshold; the README recommends estimating it with thresholdEstimate.R.
- Features recovered by MR rescue reflect poor peak shapes/low intensity — their chromatographic peak shape quality should be inspected before downstream quantification.

## Evidence

- [methods] Find MR features and add them to the original feature table.: "Find MR features and add them to the original feature table."
- [methods] featureTable <- find.level3features(data = MSdata, level3.threshold = 2): "featureTable <- find.level3features(data = MSdata, level3.threshold = 2)"
- [abstract] JPA was able to rescue an average of 25% of metabolic features that were missed by the conventional peak picking algorithm due to dilution: "JPA was able to rescue an average of 25% of metabolic features that were missed by the conventional peak picking algorithm due to dilution"
- [readme] The threshold used for determining whether the MR features are true positive can be estimated by using the R code "thresholdEstimate.R" provided on the GitHub.: "The threshold used for determining whether the MR features are true positive can be estimated by using the R code "thresholdEstimate.R" provided on the GitHub."
- [readme] After PP features have been extracted, extracting features using MS2 recognition (labeled as "MR") can be performed. This step is optional. Please do not use this function when processing full-scan or DIA data set!: "After PP features have been extracted, extracting features using MS2 recognition (labeled as "MR") can be performed. This step is optional. Please do not use this function when processing full-scan"
