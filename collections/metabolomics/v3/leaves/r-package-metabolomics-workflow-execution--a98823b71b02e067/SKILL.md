---
name: r-package-metabolomics-workflow-execution
description: Use when raw LC-MS mzXML files (e.g., serial-diluted urine MTBLS2631)
  must be processed with JPA in R to count PP vs. MR features and compute a per-sample
  MS2-based feature rescue percentage.
when_to_use_negative:
- Processing full-scan or DIA data — the README warns 'Please do not use this function
  when processing full-scan or DIA data set!' for MS2 recognition
- A single aligned multi-sample feature table is the input — JPA's custom.featureTable
  requires per-file CSV tables and 'Aligned feature table is not acceptable'
- The goal is multi-sample consolidated analysis — then peak.alignment (bw=5, minfrac=0.5,
  mzwid=0.015) is appropriate, not the per-dilution-level single-sample rescue computation
edam_operation: http://edamontology.org/operation_3215
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_3520
tools:
- name: JPA
  role: R package providing XCMS.featureTable for PP peak picking and find.level3features
    for MR feature rescue from raw mzXML data
  repo: https://github.com/HuanLab/JPA
- name: xcms
  role: Embedded peak-picking engine invoked via XCMS.featureTable with ppm=10, peakwidth=c(5,20),
    snthresh=6 parameters
- name: MSnbase
  role: Object class returned by XCMS.featureTable that carries MS2 spectra used by
    find.level3features
- name: devtools
  role: Used to install the JPA package from GitHub
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.3390/metabo12030212
    title: 'JPA: Joint Metabolic Feature Extraction Increases the Depth of Chemical
      Coverage for LC-MS-Based Metabolomics and Exposomics'
  source_content_hash: sha256:82113bd3beb57b9a28d1341a683498c2acde9de07fd5b4afd133b0e8d7c7b5b3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/HuanLab/JPA
  tools:
  - JPA
  - MSnbase
  - R
  - devtools
  - xcms
  edam_operation: http://edamontology.org/operation_3215
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.45454545454545453
n_claims: 11
---

# r-package-metabolomics-workflow-execution

## Summary

Executes the JPA R package workflow (XCMS.featureTable -> find.level3features) on raw mzXML LC-MS data to quantify how many metabolic features missed by conventional peak picking (PP) can be rescued by MS2-recognition (MR), benchmarked against the ~25% rescue reported for the MTBLS2631 serial-diluted urine dataset.

## When to use

Use when raw LC-MS mzXML files (e.g., serial-diluted urine MTBLS2631) must be processed with JPA in R to count PP vs. MR features and compute a per-sample MS2-based feature rescue percentage.

## When NOT to use

- Processing full-scan or DIA data — the README warns 'Please do not use this function when processing full-scan or DIA data set!' for MS2 recognition
- A single aligned multi-sample feature table is the input — JPA's custom.featureTable requires per-file CSV tables and 'Aligned feature table is not acceptable'
- The goal is multi-sample consolidated analysis — then peak.alignment (bw=5, minfrac=0.5, mzwid=0.015) is appropriate, not the per-dilution-level single-sample rescue computation

## Inputs

- Raw LC-MS mzXML file(s) (one folder, no unrelated mzXML files; any vendor format compatible with MS-Convert)
- Serial-diluted urine metabolomics dataset MTBLS2631 (six dilution concentrations)
- R environment >= 4.0.0 with JPA, xcms, and MSnbase installed

## Outputs

- Per-sample dataframe feature table with mz, rt, rtmin, rtmax, maxo, sample, and 'level' columns labeling features as 'PP' or 'MR'
- MSnbase object returned by XCMS.featureTable for downstream JPA functions
- Per-dilution-level rescue percentage (MR features / total features) and comparison against ~25% average, ~5% most-dilute, and ~40% RP(+) original-concentration benchmarks

## How to apply

Install JPA from GitHub (devtools::install_github('HuanLab/JPA'), requires R >= 4.0.0) and place all mzXML files in one folder with no unrelated mzXML files. Build the baseline PP feature list per sample with XCMS.featureTable(ppm=10, peakwidth=c(5,20), mzdiff=0.01, snthresh=6, integrate=1, prefilter=c(3,100), noise=100), which returns a dataframe feature table plus an MSnbase object with a 'level' column labeling each feature 'PP'. Then run find.level3features(data=MSdata, level3.threshold=2) on each sample to append MR features (bad peak-shape features that have MS2 spectra but were missed by peak picking), confirming the true-positive threshold with the thresholdEstimate.R helper from the GitHub repo. Because the dilution series is analyzed per level, do NOT run peak.alignment — alignment applies only to multi-sample analysis; instead count PP-only vs. newly added MR features per dilution level from the 'level' column and compute rescue % = MR features / total features. Validate against reported benchmarks: ~25% average rescue across six dilution levels, ~5% in the most dilute samples, and ~40% feature increase for the original-concentration RP(+) sample.

## Related tools

- **JPA** (R package providing XCMS.featureTable for PP peak picking and find.level3features for MR feature rescue from raw mzXML data) — https://github.com/HuanLab/JPA
- **xcms** (Embedded peak-picking engine invoked via XCMS.featureTable with ppm=10, peakwidth=c(5,20), snthresh=6 parameters)
- **MSnbase** (Object class returned by XCMS.featureTable that carries MS2 spectra used by find.level3features)
- **devtools** (Used to install the JPA package from GitHub)

## Examples

```
featureTable <- XCMS.featureTable(dir=dir, ppm=10, peakwidth=c(5,20), mzdiff=0.01, snthresh=6, integrate=1, prefilter=c(3,100), noise=100); featureTable <- find.level3features(data=MSdata, level3.threshold=2)
```

## Evaluation signals

- Every row of the feature table carries a 'level' value of 'PP' (baseline) or 'MR' (added by find.level3features)
- Computed rescue fractions average ~25% across the six urine dilution levels, matching the abstract benchmark
- Most dilute samples show ~5% rescue and the original-concentration RP(+) sample shows ~40% feature increase; flag any level falling far outside these values
- No peak.alignment step was executed on the dilution series (each dilution level's feature table remains single-sample)
- level3.threshold=2 was validated against thresholdEstimate.R output before counting MR features as true positives

## Limitations

- find.level3features must not be used on full-scan or DIA datasets; it requires MS2 (DDA) spectra
- MR features are features with MS2 spectra but bad peak shapes that conventional peak picking missed, so rescue rates depend on the chosen level3.threshold and the thresholdEstimate.R calibration
- XCMS.featureTable parameter choices (ppm=10, peakwidth=c(5,20), snthresh=6, prefilter=c(3,100), noise=100) directly affect how many features are labeled PP and thus the rescue fraction

## Evidence

- [abstract] JPA rescued an average of 25% of metabolic features missed by conventional peak picking in serial diluted urine samples: "JPA was able to rescue an average of 25% of metabolic features that were missed by the conventional peak picking algorithm due to dilution"
- [readme] XCMS.featureTable call with parameters: "featureTable <- XCMS.featureTable(dir = dir, mz.tol = 10, ppm=10, peakwidth=c(5,20), mzdiff = 0.01,
                             snthresh = 6, integrate = 1, prefilter = c(3,100), noise = 100)"
- [methods] MR feature extraction threshold: "featureTable <- find.level3features(data = MSdata, level3.threshold = 2)"
- [readme] DIA/full-scan restriction: "This step is optional. Please do not use this function when processing full-scan or DIA data set!"
- [readme] Threshold estimation helper: "The threshold used for determining whether the MR features are true positive can be estimated by using the R code "thresholdEstimate.R" provided on the GitHub."
