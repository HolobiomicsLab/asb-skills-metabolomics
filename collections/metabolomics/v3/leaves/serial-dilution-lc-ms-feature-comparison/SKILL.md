---
name: serial-dilution-lc-ms-feature-comparison
description: Use when you have serial diluted urine mzXML files and need to quantify
  the fraction of features missed by conventional peak picking (JPA-PP) but rescued
  by MS2 recognition (JPA-MR), per dilution level.
when_to_use_negative:
- Processing full-scan or DIA datasets — the README warns 'Please do not use this
  function when processing full-scan or DIA data set!' for MS2 recognition
- Multi-sample studies where feature tables must be aligned before comparison — here
  each dilution level's table must be treated separately
- Only quantifying absolute limits of detection for pure metabolite standards rather
  than rescued-feature proportions in complex urine samples
edam_operation: http://edamontology.org/operation_3216
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: JPA
  role: R package providing XCMS.featureTable for PP feature extraction and find.level3features
    for MR feature rescue
  repo: https://github.com/HuanLab/JPA
- name: xcms
  role: Embedded peak picking engine whose parameters (ppm=10, peakwidth=c(5,20),
    snthresh=6) define the conventional JPA-PP baseline
- name: MSnbase
  role: Data container class produced by JPA's feature table functions; holds the
    mzXML data and MS2 spectra for MR recognition
- name: R
  role: Runtime environment (version >= 4.0.0 required) for installing and executing
    JPA
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.3390/metabo12030212
    title: 'JPA: Joint Metabolic Feature Extraction Increases the Depth of Chemical
      Coverage for LC-MS-Based Metabolomics and Exposomics'
  source_content_hash: sha256:823bef49fafac1f025201e24935b13ff939ebb4154ead9bc6b31b2d4c56b4080
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
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# serial-dilution-lc-ms-feature-comparison

## Summary

Quantifies how many metabolic features a combined MS1-peak-picking + MS2-recognition workflow (JPA) rescues relative to conventional peak picking alone, by processing each serial dilution level of a urine LC-MS dataset (MTBLS2631) as a single-sample feature table and computing the rescued fraction.

## When to use

Use when you have serial diluted urine mzXML files and need to quantify the fraction of features missed by conventional peak picking (JPA-PP) but rescued by MS2 recognition (JPA-MR), per dilution level.

## When NOT to use

- Processing full-scan or DIA datasets — the README warns 'Please do not use this function when processing full-scan or DIA data set!' for MS2 recognition
- Multi-sample studies where feature tables must be aligned before comparison — here each dilution level's table must be treated separately
- Only quantifying absolute limits of detection for pure metabolite standards rather than rescued-feature proportions in complex urine samples

## Inputs

- Serial diluted urine mzXML files (MTBLS2631), one per dilution concentration, DDA-compatible format
- thresholdEstimate.R helper script from github.com/HuanLab/JPA

## Outputs

- Per-dilution-level feature table with 'level' column distinguishing PP and MR features
- Rescue percentage per dilution level (MR features / total features) and cross-dilution average

## How to apply

Load the serial diluted urine mzXML files into an MSnbase object and build the per-sample baseline feature table with JPA's XCMS.featureTable using embedded XCMS peak picking (ppm=10, peakwidth=c(5,20), snthresh=6, mzdiff=0.01, prefilter=c(3,100), noise=100), labeling every feature 'PP'. Then run find.level3features(data=MSdata, level3.threshold=2) on each sample to append MR features — features with MS2 spectra but bad peak shapes that peak picking missed — validating the true-positive threshold with the thresholdEstimate.R helper from the HuanLab/JPA GitHub. Do NOT run sample alignment, since these are single-sample per-dilution feature tables and alignment is only for multi-sample analysis. For each dilution level, count PP-only and newly added MR features and compute the rescue percentage (MR features / total features) from the 'level' column of the feature table.

## Related tools

- **JPA** (R package providing XCMS.featureTable for PP feature extraction and find.level3features for MR feature rescue) — https://github.com/HuanLab/JPA
- **xcms** (Embedded peak picking engine whose parameters (ppm=10, peakwidth=c(5,20), snthresh=6) define the conventional JPA-PP baseline)
- **MSnbase** (Data container class produced by JPA's feature table functions; holds the mzXML data and MS2 spectra for MR recognition)
- **R** (Runtime environment (version >= 4.0.0 required) for installing and executing JPA)

## Examples

```
featureTable <- XCMS.featureTable(dir=dir, ppm=10, peakwidth=c(5,20), mzdiff=0.01, snthresh=6, integrate=1, prefilter=c(3,100), noise=100); featureTable <- find.level3features(data=MSdata, level3.threshold=2)
```

## Evaluation signals

- Average rescue fraction across all six urine dilution concentrations should be ~25% of total features
- Most dilute samples should show a rescue fraction near ~5%
- Original-concentration RP(+) urine sample should show roughly a ~40% feature increase from MR rescue
- All baseline features in each output table carry level='PP' before find.level3features runs; newly appended rows carry the MR label
- Dilution levels whose rescue fraction falls far outside the ~25% average / ~5% dilute benchmarks should be flagged as validation failures

## Limitations

- MS2-recognition features cannot be validated on full-scan or DIA data; the rescue workflow requires DDA-style MS2 spectra
- Dilution drives feature loss by conventional peak picking, so rescue proportions are concentration-dependent and vary by mode (RP(+) vs HILIC(-))
- The MR true-positive threshold (level3.threshold=2) is method-specific and should be confirmed with thresholdEstimate.R per dataset

## Evidence

- [abstract] JPA rescued an average of 25% of metabolic features missed by conventional peak picking in serial diluted urine samples: "JPA was able to rescue an average of 25% of metabolic features that were missed by the conventional peak picking algorithm due to dilution"
- [readme] MR feature extraction function and threshold: "Find MR features and add them to the"
- [methods] MR threshold estimation helper: "The threshold used for determining whether the MR features are true positive can be estimated by using the R code "thresholdEstimate.R" provided on the GitHub."
- [readme] XCMS.featureTable call with baseline parameters: "featureTable <- XCMS.featureTable(dir = dir, mz.tol = 10, ppm=10, peakwidth=c(5,20), mzdiff = 0.01, snthresh = 6, integrate = 1, prefilter = c(3,100), noise = 100)"
- [methods] PP feature labeling convention: "extract metabolic features (labeled as "PP") using peak picking algorithms"
