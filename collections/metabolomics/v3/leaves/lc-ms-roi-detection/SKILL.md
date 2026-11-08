---
name: lc-ms-roi-detection
description: Use when raw LC/MS mzML files must be mined for peaks without a pre-existing
  peak list — e.g. recovering signals an XCMS run missed, or benchmarking how many
  unique (non-degenerate) peaks a deep-learning detector adds over a conventional
  peak-picker.
when_to_use_negative:
- Input is already a peak/feature list (e.g., an XCMS peaks.csv) and the goal is artifact
  classification of those peaks — use the PeakDetective peak-curation (curatePeaks)
  workflow instead of de novo ROI detection.
- The dataset lacks blank injections, so backgroundSubtract blank subtraction (which
  defines how many recovered peaks are counted as unique) cannot be performed.
- Only degenerate-signal annotation (isotopes/adducts/fragments) is needed for an
  existing peak list — removeRedundancy/mz.unity alone suffices without de novo ROI
  mining.
tools:
- name: PeakDetective
  role: 'Core de novo detector: rawData objects hold mzML scans; ROI extraction, smoother
    training, and classifier scoring of detected peaks as artifacts or true peaks'
  repo: https://github.com/pattilab/PeakDetective
- name: XCMS
  role: Baseline conventional peak-picker run on the same mzML files to define the
    set of peaks PeakDetective may be recovering (missed-by-XCMS intersection)
- name: mz.unity (via detection_helper.PeakList)
  role: Annotates degenerate signals (isotopes, adducts, fragments) so recovered peaks
    are counted as unique signals after removeRedundancy
- name: pyteomics / numpy
  role: Required Python dependencies for reading mass-spectrometric raw data and numeric
    processing in the PeakDetective package
  repo: https://github.com/pattilab/PeakDetective
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1021/acs.analchem.3c00764
    title: 'PeakDetective: A Semisupervised Deep Learning-Based Approach for Peak
      Curation in Untargeted Metabolomics'
  source_content_hash: sha256:62ad9f799ae5da6e4066c3b51be7586fe350131bb029b546d2bdaa45a7d3b96d
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/pattilab/PeakDetective
  tools:
  - PeakDetective
  - Python
  - XCMS
  - mz.unity (via detection_helper.PeakList)
  - numpy
  - pyteomics
  - pyteomics / numpy
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# lc-ms-roi-detection

## Summary

De novo LC/MS peak (ROI) detection using PeakDetective: raw mzML data is mined for regions of interest (3 consecutive scans above the noise threshold), smoothed, and scored by a trained classifier as artifact vs true peak, recovering signals that centroided peak-pickers like XCMS miss.

## When to use

Use when raw LC/MS mzML files must be mined for peaks without a pre-existing peak list — e.g. recovering signals an XCMS run missed, or benchmarking how many unique (non-degenerate) peaks a deep-learning detector adds over a conventional peak-picker.

## When NOT to use

- Input is already a peak/feature list (e.g., an XCMS peaks.csv) and the goal is artifact classification of those peaks — use the PeakDetective peak-curation (curatePeaks) workflow instead of de novo ROI detection.
- The dataset lacks blank injections, so backgroundSubtract blank subtraction (which defines how many recovered peaks are counted as unique) cannot be performed.
- Only degenerate-signal annotation (isotopes/adducts/fragments) is needed for an existing peak list — removeRedundancy/mz.unity alone suffices without de novo ROI mining.

## Inputs

- Raw LC/MS mzML files (e.g., SARS-CoV-2 hamster plasma acquisition in negative mode)
- MS1 tolerance parameter (ms1ppm, e.g., 25.5 ppm)
- Blank/acquisition sample-type keys used for backgroundSubtract (blankKey, sampleKey)
- Trained PeakDetective smoother/classifier weights or enough user-labeled peaks for active-learning training (<100 labels)
- Reference peak list from a baseline detector (e.g., XCMS peaks.csv) for recovery comparison

## Outputs

- peak_scores_pd.csv — per-ROI classifier scores
- peak_intensities_pd.csv — per-peak sample intensity matrix
- De novo peak table (mz, rt, score) for peaks above cutoff
- XCMS-vs-PeakDetective recovery comparison table (e.g., peakdetective_xcms_peak_recovery_comparison.csv) quantifying recovered and unique peaks

## How to apply

Load each mzML file into a PeakDetective.rawData object with the MS1 tolerance matching the instrument (ms1ppm = 25.5 ppm for the SARS-CoV-2 hamster plasma data), then run de novo ROI detection: an ROI is defined by numDataPoints = 3 consecutive scans above the noise intensityCutoff (=1000 for this dataset). Train the smoother and classifier (via active learning, <100 user-labeled peaks) until the score histogram is bimodal, inspect the TPR/FDR curves, and set cutoff = 0.8; run detection with detectFrac = 0.5 (minimum fraction of samples where a metabolite must be detected) and RT alignment enabled. After detection, apply the cleanup filters — imputeRowMin, then backgroundSubtract with blankKey/sampleKey and factor 2 against blanks, then removeRedundancy (corrThresh = 0.9, rtThresh = 0.5 min, polarity = -1, 25.5 ppm) to annotate isotopes/adducts/fragments via mz.unity. Count peaks scored above cutoff that intersect with (i.e., are missed by) the XCMS baseline run at peakWidth = (13.8, 114.6) s, s2n = 13.6, noise = 1, mzDiff = 0.0144, prefilter = 5.

## Related tools

- **PeakDetective** (Core de novo detector: rawData objects hold mzML scans; ROI extraction, smoother training, and classifier scoring of detected peaks as artifacts or true peaks) — https://github.com/pattilab/PeakDetective
- **XCMS** (Baseline conventional peak-picker run on the same mzML files to define the set of peaks PeakDetective may be recovering (missed-by-XCMS intersection))
- **mz.unity (via detection_helper.PeakList)** (Annotates degenerate signals (isotopes, adducts, fragments) so recovered peaks are counted as unique signals after removeRedundancy)
- **pyteomics / numpy** (Required Python dependencies for reading mass-spectrometric raw data and numeric processing in the PeakDetective package) — https://github.com/pattilab/PeakDetective

## Examples

```
temp = PeakDetective.rawData(); temp.readRawDataFile(mzmlFolder + "/" + file, ms1ppm); raw_data.append(temp)
```

## Evaluation signals

- Classifier score distribution across all ROIs is bimodal after 5–10 active-learning iterations, with val_mean_absolute_error dropping dramatically.
- At the chosen cutoff (0.8), the reported TPR and FDR estimates from validation peaks are reproduced from the plotted TPR/FDR curves.
- Intersecting recovered peaks with the XCMS list reproduces the reported 2,414 XCMS-missed peaks recovered on the SARS-CoV-2 hamster dataset.
- After backgroundSubtract (factor 2) and removeRedundancy (corrThresh = 0.9), the count of unique recovered signals matches the reported 623 peaks; >70% of recovered peaks being degenerate is expected.
- Output CSVs (peak_scores_pd.csv, peak_intensities_pd.csv) exist and contain score/intensity columns indexed by detected peaks.

## Limitations

- Most recovered peaks are degenerate: blank subtraction and mz.unity annotation showed >70% of the 2,414 peaks XCMS missed were isotopes/adducts/fragments, so raw recovery counts overstate unique new signals.
- The classifier must be trained per LC/MS method and sample type (via active learning with user labeling) to maximize performance; model parameters like numDataPoints = 3 are 'not recommended to change' without cause.
- mz.unity and XCMS support requires installing R (>v4.0) packages, taking >20 min, with R on the system path.

## Evidence
