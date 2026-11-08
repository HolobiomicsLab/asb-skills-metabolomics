---
name: feature-fidelity-rate-comparison
description: Use when you need to quantify how relaxing peak-picking thresholds (mass
  tolerance, S/N) changes JPA-PP vs JPA-MR feature counts and the proportion of features
  with acceptable chromatographic peak shapes.
when_to_use_negative:
- When processing full-scan or DIA data sets — find.level3features (MS2 recognition
  / MR) must not be used in those cases per the JPA manual
- When the input is already an aligned feature table rather than raw mzXML files —
  JPA requires per-file CSV tables for custom input and raw mzXML for XCMS.featureTable()
- When the goal is absolute feature quantification rather than parameter-sensitivity
  comparison of fidelity rates
edam_operation: http://edamontology.org/operation_3219
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: JPA
  role: R package providing XCMS.featureTable(), find.level3features(), and plot.features()
    used to extract PP/MR features and evaluate peak shapes under both parameter settings
  repo: https://github.com/HuanLab/JPA.git
- name: R
  role: Runtime environment for the JPA package (R >= 4.0.0; RStudio recommended)
- name: xcms
  role: Embedded XCMS peak-picking functions invoked by XCMS.featureTable() to extract
    level-1 (PP) features with configurable ppm and snthresh
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.3390/metabo12030212
    title: 'JPA: Joint Metabolic Feature Extraction Increases the Depth of Chemical
      Coverage for LC-MS-Based Metabolomics and Exposomics'
  source_content_hash: sha256:1e41da2151c92ff5cd50af98a666096e41d950903140066328518816378f0952
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/HuanLab/JPA.git
  tools:
  - JPA
  - R
  - xcms
  edam_operation: http://edamontology.org/operation_3219
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# feature-fidelity-rate-comparison

## Summary

Reprocesses the same LC-MS mzXML data through the JPA pipeline twice — once with default parameters (ppm=10, snthresh=6) and once with tolerant thresholds (ppm=30, S/N=3) — to quantify how peak-picking (PP) versus MS2-recognition (MR) feature counts and chromatographic peak-shape fidelity rates shift between parameter settings.

## When to use

Use when you need to quantify how relaxing peak-picking thresholds (mass tolerance, S/N) changes JPA-PP vs JPA-MR feature counts and the proportion of features with acceptable chromatographic peak shapes.

## When NOT to use

- When processing full-scan or DIA data sets — find.level3features (MS2 recognition / MR) must not be used in those cases per the JPA manual
- When the input is already an aligned feature table rather than raw mzXML files — JPA requires per-file CSV tables for custom input and raw mzXML for XCMS.featureTable()
- When the goal is absolute feature quantification rather than parameter-sensitivity comparison of fidelity rates

## Inputs

- Raw LC-MS mzXML files (e.g., NISTplasmaDDARP(-) mzXML samples) placed in a folder containing no other irrelevant mzXML files
- JPA R package (R >= 4.0.0) installed via devtools from GitHub

## Outputs

- Feature tables with a 'level' column labeling each feature 'PP' or 'MR' for both default and tolerant parameter runs
- Per-level feature counts for each parameter setting
- Fidelity rates (proportion of features with acceptable chromatographic peak shapes) per level and setting
- fidelity_comparison_table.csv with side-by-side default vs tolerant counts and fidelity rates

## How to apply

Install JPA from github.com/HuanLab/JPA (R >= 4.0.0) via devtools, then run XCMS.featureTable() on the NISTplasmaDDARP(-) mzXML files with default JPA parameters (ppm=10, peakwidth=c(5,20), snthresh=6, mzdiff=0.01, prefilter=c(3,100), noise=100) to build the default JPA-PP feature table, and run find.level3features(level3.threshold=2) to add JPA-MR features. Reprocess the identical raw data with tolerant thresholds (mass tolerance 30 ppm, S/N 3), repeating both the PP and MR steps. Count features per level ('PP' and 'MR') in each setting and compute the fidelity rate — the proportion of features with acceptable chromatographic peak shapes — for each level and parameter setting, following the Figure S4 evaluation protocol. Tabulate counts and fidelity rates side-by-side (default vs tolerant) and export as fidelity_comparison_table.csv; the expected signature is a JPA-PP fidelity drop of ~10% on average under tolerant thresholds while JPA-MR fidelity stays similar, meaning many tolerant-threshold PP features still require JPA rescue.

## Related tools

- **JPA** (R package providing XCMS.featureTable(), find.level3features(), and plot.features() used to extract PP/MR features and evaluate peak shapes under both parameter settings) — https://github.com/HuanLab/JPA.git
- **R** (Runtime environment for the JPA package (R >= 4.0.0; RStudio recommended))
- **xcms** (Embedded XCMS peak-picking functions invoked by XCMS.featureTable() to extract level-1 (PP) features with configurable ppm and snthresh)

## Examples

```
featureTable <- XCMS.featureTable(dir = dir, ppm=30, peakwidth=c(5,20), snthresh=3, integrate=1, prefilter=c(3,100), noise=100); featureTable <- find.level3features(data = MSdata, level3.threshold = 2)
```

## Evaluation signals

- Feature tables contain a 'level' column with values 'PP' and 'MR' for both runs
- Under tolerant thresholds, JPA-PP feature counts increase while JPA-MR counts decrease slightly
- JPA-PP fidelity rate drops by ~10% on average under tolerant thresholds, matching the reported Figure S4 behavior
- JPA-MR fidelity rate remains similar between default and tolerant settings
- fidelity_comparison_table.csv contains side-by-side counts and fidelity rates for default (ppm=10, snthresh=6) and tolerant (ppm=30, S/N=3) runs

## Limitations

- find.level3features (MR extraction) must not be used with full-scan or DIA data sets
- Fidelity rates depend on visual/shape-based peak evaluation (plot.features) following the Figure S4 protocol; acceptable-peak-shape judgment is qualitative
- Only features with acceptable chromatographic peak shapes count toward fidelity; low-intensity or non-Gaussian features rescued by JPA require the MR step to be recovered
- mzXML files must be MS-Convert-compatible and isolated in a folder with no other irrelevant mzXML files

## Evidence

- [readme] default XCMS.featureTable parameters: "featureTable <- XCMS.featureTable(dir = dir, mz.tol = 10, ppm=10, peakwidth=c(5,20), mzdiff = 0.01, snthresh = 6, integrate = 1, prefilter = c(3,100), noise = 100)"
- [readme] MR threshold usage: "The threshold used for determining whether the MR features are true positive can be estimated by using the R code "thresholdEstimate.R" provided on the GitHub."
- [readme] MR restriction to DDA data: "This step is optional. Please do not use this function when processing full-scan or DIA data set!"
- [readme] feature-table structure: "The XCMS.featureTable() function outputs a dataframe formatted featuretable as well as an MSnbase object. The "level" column shows the level of each feature."
- [results] tolerant-threshold comparison finding: [evidence span withheld — anchor status: not_found]
- [methods] JPA installation requirement: "To install `JPA` package R version 4.0.0 or above is required"
