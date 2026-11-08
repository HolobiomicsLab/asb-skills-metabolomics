---
name: dda-metabolomics-dataset-reprocessing
description: Use when you need to assess the sensitivity of a JPA feature-extraction
  workflow to peak-picking thresholds, e.g., comparing default (ppm=10, snthresh=6)
  versus tolerant (ppm=30, S/N=3) settings on the same raw mzXML files.
when_to_use_negative:
- The dataset is full-scan or DIA data — the README explicitly warns 'Please do not
  use this function when processing full-scan or DIA data set!' for MR feature extraction
- The input is already a processed feature table rather than raw mzXML files (use
  custom.featureTable instead of reprocessing raw data)
- A single-pass feature extraction is sufficient and no parameter-sensitivity comparison
  is required
edam_operation: http://edamontology.org/operation_3215
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: JPA
  role: R package providing XCMS.featureTable(), find.level3features(), and plot.features()
    used to extract and evaluate PP/MR features under both parameter settings
  repo: https://github.com/HuanLab/JPA.git
- name: R
  role: Runtime environment (>= 4.0.0 required) in which the JPA package is installed
    and executed
- name: xcms
  role: Embedded XCMS peak-picking functions invoked by XCMS.featureTable() for MS1
    PP feature extraction
- name: devtools
  role: Used to install the JPA package from GitHub
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.3390/metabo12030212
    title: 'JPA: Joint Metabolic Feature Extraction Increases the Depth of Chemical
      Coverage for LC-MS-Based Metabolomics and Exposomics'
  source_content_hash: sha256:f8824cfb50a812b2c38f4fff315d2acb6a993dc6fbaf28d6a95d1e844bc93a3f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/HuanLab/JPA.git
  tools:
  - JPA
  - R
  - devtools
  - xcms
  edam_operation: http://edamontology.org/operation_3215
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# dda-metabolomics-dataset-reprocessing

## Summary

Reprocess the same LC-MS/MS (DDA) mzXML dataset twice through the JPA pipeline — once with default JPA parameters (ppm=10, snthresh=6) and once with tolerant thresholds (ppm=30, S/N=3) — to quantify how peak-picking (JPA-PP) and MS2-recognition (JPA-MR) feature counts and fidelity rates shift with parameter tolerance.

## When to use

Use when you need to assess the sensitivity of a JPA feature-extraction workflow to peak-picking thresholds, e.g., comparing default (ppm=10, snthresh=6) versus tolerant (ppm=30, S/N=3) settings on the same raw mzXML files.

## When NOT to use

- The dataset is full-scan or DIA data — the README explicitly warns 'Please do not use this function when processing full-scan or DIA data set!' for MR feature extraction
- The input is already a processed feature table rather than raw mzXML files (use custom.featureTable instead of reprocessing raw data)
- A single-pass feature extraction is sufficient and no parameter-sensitivity comparison is required

## Inputs

- Raw LC-MS/MS mzXML files (e.g., NISTplasmaDDARP(-) mzXML samples) in a single dedicated folder
- Two parameter sets: default JPA (ppm=10, peakwidth=c(5,20), snthresh=6) and tolerant (mass tolerance=30 ppm, S/N=3)

## Outputs

- Feature tables with 'level' column (PP and MR features) for each parameter setting
- fidelity_comparison_table.csv with side-by-side feature counts and fidelity rates for default vs tolerant runs

## How to apply

Install JPA (R >= 4.0.0) from github.com/HuanLab/JPA via devtools and load it. Run XCMS.featureTable() on the NISTplasmaDDARP(-) mzXML files with default JPA parameters (ppm=10, peakwidth=c(5,20), snthresh=6) to build the JPA-PP feature table, then run find.level3features(level3.threshold=2) to add JPA-MR features. Reprocess the identical raw data with tolerant thresholds (mass tolerance 30 ppm, S/N 3), repeating both the PP and MR steps. Count features per level ('PP' and 'MR') in each setting and compute the fidelity rate (proportion of features with acceptable chromatographic peak shapes) per level per setting, following the Figure S4 evaluation protocol. Tabulate default-versus-tolerant counts and fidelity rates side-by-side and export to fidelity_comparison_table.csv; the expected behavior is a ~10% average drop in JPA-PP fidelity under tolerant thresholds while JPA-MR fidelity stays similar, meaning many features still require JPA rescue.

## Related tools

- **JPA** (R package providing XCMS.featureTable(), find.level3features(), and plot.features() used to extract and evaluate PP/MR features under both parameter settings) — https://github.com/HuanLab/JPA.git
- **R** (Runtime environment (>= 4.0.0 required) in which the JPA package is installed and executed)
- **xcms** (Embedded XCMS peak-picking functions invoked by XCMS.featureTable() for MS1 PP feature extraction)
- **devtools** (Used to install the JPA package from GitHub)

## Examples

```
featureTable <- XCMS.featureTable(dir = dir, ppm=10, peakwidth=c(5,20), snthresh = 6, integrate = 1, prefilter = c(3,100), noise = 100)
```

## Evaluation signals

- Feature counts per level (PP and MR) can be tallied from the 'level' column of each feature table for both parameter settings
- JPA-PP fidelity rate drops by ~10% on average under tolerant thresholds while JPA-MR fidelity rate stays similar, matching Figure S4 evaluation behavior
- JPA-PP extracts more features under tolerant thresholds while JPA-MR features decrease slightly
- A large number of features still require JPA rescue even under tolerant parameters
- fidelity_comparison_table.csv contains side-by-side default vs tolerant counts and fidelity rates per level

## Limitations

- find.level3features (MR extraction) must not be used on full-scan or DIA datasets lacking MS2
- Conventional feature extraction algorithms fail to recognize features with low signal intensities, poor chromatographic peak shapes, or those that do not fit parameter settings
- All mzXML files must be placed in the same folder containing no other irrelevant mzXML files; JPA only accepts raw data compatible with MS-Convert

## Evidence

- [methods] default JPA parameters: "The XCMS.featureTable() function outputs a dataframe formatted featuretable"
- [methods] tolerant thresholds MR extraction: "featureTable <- find.level3features(data = MSdata, level3.threshold = 2)"
- [abstract] peak-shape fidelity evaluation: "More importantly, the chromatographic peak shapes, analytical accuracy, and precision of the rescued metabolic features were all evaluated."
- [readme] feature extraction levels: "The "level" column shows the level of each feature."
- [readme] default XCMS.featureTable parameters: "featureTable <- XCMS.featureTable(dir = dir, mz.tol = 10, ppm=10, peakwidth=c(5,20), mzdiff = 0.01,
                             snthresh = 6, integrate = 1, prefilter = c(3,100), noise = 100)"
- [readme] MR restriction to DDA data: "Please do not use this function when processing full-scan or DIA data set!"
