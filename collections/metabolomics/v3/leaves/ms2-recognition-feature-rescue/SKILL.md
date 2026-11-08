---
name: ms2-recognition-feature-rescue
description: Use when processing DDA LC-MS mzXML data in which conventional peak picking
  (XCMS.featureTable with snthresh=6, prefilter=c(3,100)) has already run but low-intensity
  or poorly shaped peaks with detectable MS2 spectra were missed.
when_to_use_negative:
- Do not use when processing full-scan or DIA data — the README explicitly warns 'Please
  do not use this function when processing full-scan or DIA data set!' because MR
  recognition depends on DDA MS2 spectra.
- Do not use when the input is already an aligned feature table rather than per-file
  mzXML/raw data with associated MS2 spectra.
- Do not use before any PP feature extraction — MR features are found on top of an
  existing peak-picked feature table.
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: JPA
  role: Provides find.level3features(), the function that recognizes bad-peak-shape
    features via MS2 spectra and adds them as MR features to the feature table
  repo: https://github.com/HuanLab/JPA.git
- name: R
  role: Runtime environment; JPA requires R version 4.0.0 or above
- name: xcms
  role: Embedded XCMS peak-picking functions produce the PP feature table and MSnbase
    object that MR rescue operates on
- name: devtools
  role: Used to install the JPA package from GitHub (install_github('HuanLab/JPA'))
provenance:
  source_task_ids:
  - task_004
  - task_007
  - task_005
  - task_008
  source_papers:
  - doi: 10.3390/metabo12030212
    title: 'JPA: Joint Metabolic Feature Extraction Increases the Depth of Chemical
      Coverage for LC-MS-Based Metabolomics and Exposomics'
  source_content_hash: sha256:ac6c7e3ec3b79318ad2fab889e62b1a2a209e2f9e0ca273f97899b3adbde4581
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/HuanLab/JPA.git
  tools:
  - JPA
  - R
  - devtools
  - xcms
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# ms2-recognition-feature-rescue

## Summary

Rescues metabolic features that XCMS peak picking missed by recognizing features with bad chromatographic peak shapes that nevertheless have tandem MS2 spectra, labeling them as 'MR' level features via JPA's find.level3features().

## When to use

Use when processing DDA LC-MS mzXML data in which conventional peak picking (XCMS.featureTable with snthresh=6, prefilter=c(3,100)) has already run but low-intensity or poorly shaped peaks with detectable MS2 spectra were missed.

## When NOT to use

- Do not use when processing full-scan or DIA data — the README explicitly warns 'Please do not use this function when processing full-scan or DIA data set!' because MR recognition depends on DDA MS2 spectra.
- Do not use when the input is already an aligned feature table rather than per-file mzXML/raw data with associated MS2 spectra.
- Do not use before any PP feature extraction — MR features are found on top of an existing peak-picked feature table.

## Inputs

- MSnbase object or folder of mzXML files from DDA LC-MS runs (e.g., NISTplasmaDDARP(-) mzXML samples)
- Feature table dataframe with 'level' column produced by XCMS.featureTable() or custom.featureTable()

## Outputs

- Augmented feature table dataframe containing additional rows with level = 'MR'
- Per-level (PP vs. MR) feature counts for comparison across parameter settings or cycle times

## How to apply

First extract PP features from the raw mzXML files with XCMS.featureTable() (e.g., ppm=10, peakwidth=c(5,20), snthresh=6, prefilter=c(3,100)), because MR extraction runs only on the resulting MSnbase object. Then call find.level3features(data = MSdata, level3.threshold = 2) to identify features with bad peak shapes whose tandem MS2 spectra confirm them as true signals; the threshold for determining true-positive MR features can be re-estimated with the thresholdEstimate.R script on the HuanLab/JPA GitHub. The function appends these MR features to the feature table with the 'level' column set to 'MR' (they are not duplicated if already present as PP features). For multi-sample analyses, run peak.alignment() (e.g., bw=5, minfrac=0.5, mzwid=0.015) only after both PP and MR features are extracted. The rationale is that conventional peak-detection parameters systematically discard non-Gaussian or low-intensity features that MS2 spectra can still vouch for.

## Related tools

- **JPA** (Provides find.level3features(), the function that recognizes bad-peak-shape features via MS2 spectra and adds them as MR features to the feature table) — https://github.com/HuanLab/JPA.git
- **R** (Runtime environment; JPA requires R version 4.0.0 or above)
- **xcms** (Embedded XCMS peak-picking functions produce the PP feature table and MSnbase object that MR rescue operates on)
- **devtools** (Used to install the JPA package from GitHub (install_github('HuanLab/JPA')))

## Examples

```
featureTable <- find.level3features(data = MSdata, level3.threshold = 2)
```

## Evaluation signals

- Feature table must gain new rows with the 'level' column set to 'MR' after find.level3features(), without duplicating existing PP features.
- In the cycle-time experiment, JPA-MR feature counts should increase monotonically as DDA cycle time goes from 1 s to 3 s while JPA-PP counts decrease (Figure S3 trend).
- Under tolerant thresholds (30 ppm, S/N = 3), JPA-MR counts decrease only slightly and maintain a fidelity rate similar to default settings, whereas JPA-PP fidelity drops ~10% on average (Figure S4 trend).
- Combined PP + MR features should rescue on average ~25% of features missed by conventional peak picking in serial diluted urine samples.
- The chosen level3.threshold (default 2 in the article) should be justified, optionally re-estimated with thresholdEstimate.R from the GitHub.

## Limitations

- Invalid for full-scan or DIA acquisitions — MR recognition requires DDA tandem MS2 spectra.
- The MR rescue threshold (level3.threshold) governs true-positive determination and must be set/estimated carefully; the paper uses level3.threshold = 2.
- MR features rescue only features missed by peak picking due to shape/intensity/parameter issues; they do not replace targeted-list extraction (find.targetfeatures) for compounds absent from the PP/MR output.

## Evidence

- [readme] MR rescue depends on DDA MS2 and is barred for other modes: "After PP features have been extracted, extracting features using MS2 recognition (labeled as "MR") can be performed."
- [readme] Scope restriction to DDA data: "Please do not use this function when processing full-scan or DIA data set!"
- [readme] Threshold estimation guidance: "The threshold used for determining whether the MR features are true positive can be estimated by using the R code "thresholdEstimate.R" provided on the GitHub."
- [methods] Concrete function call and threshold: "featureTable <- find.level3features(data = MSdata, level3.threshold = 2)"
- [abstract] Rationale for rescue: peak picking failure modes: "Conventional feature extraction algorithms fail to recognize features with low signal intensities, poor chromatographic peak shapes, or those that do not fit the parameter settings."
- [abstract] Quantified rescue outcome: "JPA was able to rescue an average of 25% of metabolic features that were missed by the conventional peak picking algorithm due to dilution"
