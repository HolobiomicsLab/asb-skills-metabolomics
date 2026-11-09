---
name: serial-dilution-feature-count-analysis
description: Use when processing a serial dilution series of LC-MS samples (raw mzXML,
  one chromatographic mode per folder) to measure how total and conventional peak-picking
  feature counts drop with dilution, e.g. dilution-response or LOD benchmarking.
when_to_use_negative:
- The goal is to recover dilution-lost features rather than quantify their loss —
  use JPA's MS2 recognition (MR) step instead of counting PP-only features
- Input is an aligned or CSV feature table from other software rather than raw mzXML
  files — the XCMS.featureTable(dir) route requires raw LC-MS data; use custom.featureTable
  instead
- Processing full-scan or DIA data sets, for which the README explicitly warns against
  using the MS2-recognition function
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_0128
tools:
- name: JPA
  role: R package whose XCMS.featureTable function performs the MS1 PP peak-picking
    feature extraction on each dilution sample
  repo: https://github.com/HuanLab/JPA.git
- name: xcms
  role: Embedded peak-picking algorithm invoked by JPA's XCMS.featureTable with the
    paper's parameter set
- name: devtools
  role: Used to install the JPA package from GitHub
- name: BiocManager
  role: Dependency installer required before installing devtools/JPA
- name: R
  role: Runtime environment (version 4.0.0 or above) for the JPA workflow and count
    computation
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.3390/metabo12030212
    title: 'JPA: Joint Metabolic Feature Extraction Increases the Depth of Chemical
      Coverage for LC-MS-Based Metabolomics and Exposomics'
  source_content_hash: sha256:95fb79c502190ce6a34cb9b758e1eb3ecfeb1fc6abe94856ba53c4a1913e92f2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/HuanLab/JPA.git
  tools:
  - BiocManager
  - JPA
  - MSnbase
  - R
  - devtools
  - xcms
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_0128
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# serial-dilution-feature-count-analysis

## Summary

Quantifies how JPA's conventional peak-picking (PP) feature counts degrade across the serial diluted urine LC-MS series (dilution factors 1-10, RP(+) and HILIC(-) modes), demonstrating the dilution-driven feature loss that JPA's MR strategy rescues (~25% of features missed by conventional peak picking).

## When to use

Use when processing a serial dilution series of LC-MS samples (raw mzXML, one chromatographic mode per folder) to measure how total and conventional peak-picking feature counts drop with dilution, e.g. dilution-response or LOD benchmarking.

## When NOT to use

- The goal is to recover dilution-lost features rather than quantify their loss — use JPA's MS2 recognition (MR) step instead of counting PP-only features
- Input is an aligned or CSV feature table from other software rather than raw mzXML files — the XCMS.featureTable(dir) route requires raw LC-MS data; use custom.featureTable instead
- Processing full-scan or DIA data sets, for which the README explicitly warns against using the MS2-recognition function

## Inputs

- Raw mzXML files of serial diluted urine samples (dilution factors 1-10) per chromatographic mode (RP(+) and HILIC(-)), MetaboLights accession MTBLS2631
- JPA R package environment (R >= 4.0.0, RStudio, xcms/MSnbase dependencies)

## Outputs

- Dataframe-formatted feature table with mz, rt, rtmin, rtmax, maxo, sample, and 'level' columns (level = 'PP')
- Per-dilution PP feature counts per chromatographic mode, written to dilution_feature_counts.csv
- Percent decrease in total features and JPA-PP-only features from original to 10-fold dilution

## How to apply

Obtain the raw mzXML files of the serial diluted urine samples (MetaboLights MTBLS2631, RP(+) and HILIC(-)) and place all files of one mode in a single folder with no irrelevant mzXML files. Install JPA from GitHub (HuanLab/JPA) under R 4.0.0+ via BiocManager/devtools. For each dilution sample, run the MS1 peak-picking step with the paper's XCMS parameters (mz.tol = 10, ppm = 10, peakwidth = c(5,20), mzdiff = 0.01, snthresh = 6, integrate = 1, prefilter = c(3,100), noise = 100) to obtain features labeled 'PP' in the 'level' column of the returned dataframe feature table. Count PP features per dilution level and per chromatographic mode, then compute the percent decrease from the original (undiluted) sample to the 10-fold dilution. The expected result reproduces the reported dilution-response behavior: total features decreased by 52% and JPA-PP-only features decreased by ~59% (82% to 34%) from original to 10-fold dilution, consistent with the paper's finding that dilution causes conventional peak picking to miss an average of 25% of metabolic features.

## Related tools

- **JPA** (R package whose XCMS.featureTable function performs the MS1 PP peak-picking feature extraction on each dilution sample) — https://github.com/HuanLab/JPA.git
- **xcms** (Embedded peak-picking algorithm invoked by JPA's XCMS.featureTable with the paper's parameter set)
- **devtools** (Used to install the JPA package from GitHub)
- **BiocManager** (Dependency installer required before installing devtools/JPA)
- **R** (Runtime environment (version 4.0.0 or above) for the JPA workflow and count computation)

## Examples

```
featureTable <- XCMS.featureTable(dir = dir, mz.tol = 10, ppm = 10, peakwidth = c(5,20), mzdiff = 0.01, snthresh = 6, integrate = 1, prefilter = c(3,100), noise = 100); nrow(subset(featureTable, level == 'PP'))
```

## Evaluation signals

- Total features detected by JPA (PP plus MR) decrease by ~52% from original concentration to 10-fold dilution
- JPA-PP-only features decrease by ~59% (from 82% to 34%) over the same dilution range
- Every feature row in the dataframe carries level = 'PP', confirming the conventional peak-picking stage ran correctly
- Counts are computed per dilution level (1-10) and per chromatographic mode (RP(+) vs HILIC(-)), matching the expected monotonic dilution-response behavior
- Per-dilution counts are exported to dilution_feature_counts.csv with one row per dilution level and mode

## Limitations

- PP-only counts quantify the loss, not the recovery — rescued features require the optional MR (MS2 recognition) step, which should not be used on full-scan or DIA data sets
- All mzXML files of one mode must be placed in a folder containing no other irrelevant mzXML files, or extraction will include unrelated samples
- Results depend on the specific XCMS parameters (snthresh = 6, prefilter = c(3,100), noise = 100); changing them changes the feature counts and breaks comparability with the reported 52%/59% decreases

## Evidence

- [abstract] Dilution-response expected magnitude: "JPA was able to rescue an average of 25% of metabolic features that were missed by the conventional peak picking algorithm due to dilution"
- [readme] XCMS.featureTable call with parameters: "featureTable <- XCMS.featureTable(dir = dir, mz.tol = 10, ppm=10, peakwidth=c(5,20), mzdiff = 0.01, snthresh = 6, integrate = 1, prefilter = c(3,100), noise = 100)"
- [readme] level column semantics: "The XCMS.featureTable() function outputs a dataframe formatted featuretable as well as an MSnbase object. The "level" column shows the level of each feature."
- [methods] Installation requirements: "To install `JPA` package R version 4.0.0 or above is required"
- [readme] mzXML folder requirement: "All mzXML file(s) need to be placed in the same folder containing no other irrelevant mzXML files."
