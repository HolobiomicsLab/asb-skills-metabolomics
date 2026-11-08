---
name: spiked-standard-validation-analysis
description: Use when a dataset contains spiked metabolite standards (e.g., 81 standards
  in a P. pastoris extract) and you need to quantify true-peak recall against XCMS-detected
  features.
when_to_use_negative:
- The dataset has no spiked standards or other ground-truth labels, so recall on known-true
  features cannot be computed
- Peak detection has not yet been performed and no XCMS peak list exists to curate
- No trained PeakDetective model (autoencoder, smoother, classifier) is available
  and active-learning labeling cannot be performed
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: PeakDetective
  role: Semisupervised deep learning classifier that scores and labels each XCMS-detected
    peak as artifact or true peak via integ.curatePeaks at cutoff 0.8
  repo: https://github.com/pattilab/PeakDetective
- name: XCMS
  role: Upstream peak detection producing the peak list (peaks_formatted.csv) from
    the spiked mzML files that is then curated and evaluated
- name: numpy
  role: Numerical computation of the true-peak fractions and their variability (mean
    +/- SD) across runs
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1021/acs.analchem.3c00764
    title: 'PeakDetective: A Semisupervised Deep Learning-Based Approach for Peak
      Curation in Untargeted Metabolomics'
  source_content_hash: sha256:3e44e8fbe610cc50aede834ab8df6f71a470e352e6490af2c526718478268bfe
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/pattilab/PeakDetective
  tools:
  - PeakDetective
  - Python
  - XCMS
  - numpy
  - pyteomics
  edam_operation: http://edamontology.org/operation_2990
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# spiked-standard-validation-analysis

## Summary

Validates PeakDetective peak-classification performance against ground truth defined by metabolite standards spiked into a sample: compute the fraction of all detected peaks versus the fraction of spiked-metabolite features classified as true peaks.

## When to use

Use when a dataset contains spiked metabolite standards (e.g., 81 standards in a P. pastoris extract) and you need to quantify true-peak recall against XCMS-detected features.

## When NOT to use

- The dataset has no spiked standards or other ground-truth labels, so recall on known-true features cannot be computed
- Peak detection has not yet been performed and no XCMS peak list exists to curate
- No trained PeakDetective model (autoencoder, smoother, classifier) is available and active-learning labeling cannot be performed

## Inputs

- Raw LC/MS data in mzML format (P. pastoris extract spiked with 81 metabolite standards)
- XCMS-generated peak list CSV (peaks_formatted.csv) with mz and rt columns
- Trained PeakDetective model weights saved in a PeakDetectiveObject directory
- Classification cutoff value (e.g., 0.8)

## Outputs

- peaks_curated.csv (artifact/true-peak classification per feature)
- peak_scores.csv (classifier score per peak)
- peak_intensities.csv (per-sample peak intensities)
- Fraction of all peaks classified as true peaks (reported: 66.7 +/- 0.34%)
- Fraction of spiked-metabolite features classified as true peaks (reported: 83.9 +/- 0.81%)

## How to apply

Load the saved trained PeakDetective model weights from the PeakDetectiveObject directory into a PeakDetective object and set the classification cutoff to 0.8. Read the raw mzML files with rawData.readRawDataFile using an ms1ppm tolerance of 25.5, then read the XCMS-generated peak list (peaks_formatted.csv) and extract the m/z and retention time columns. Run integ.curatePeaks on the raw data and peak list with threshold=cutoff (0.8) and align=True so that every peak is scored and classified as artifact or true peak, writing peaks_curated.csv, peak_scores.csv, and peak_intensities.csv. Finally, split the curated classifications into two strata — all XCMS-detected peaks and the subset of features matching the 81 spiked metabolite standards — and compute the fraction classified as true peaks in each stratum. Rationale: because spiked features are known true peaks by construction, their recall (83.9 +/- 0.81% reported) should exceed the overall true-peak fraction (66.7 +/- 0.34%), and a significance test (p<0.0001 reported) on the difference validates the classifier's specificity for genuine analyte shapes over artifacts.

## Related tools

- **PeakDetective** (Semisupervised deep learning classifier that scores and labels each XCMS-detected peak as artifact or true peak via integ.curatePeaks at cutoff 0.8) — https://github.com/pattilab/PeakDetective
- **XCMS** (Upstream peak detection producing the peak list (peaks_formatted.csv) from the spiked mzML files that is then curated and evaluated)
- **numpy** (Numerical computation of the true-peak fractions and their variability (mean +/- SD) across runs)

## Examples

```
integ = PeakDetective.PeakDetective(numCores=10, resolution=60); integ.load('example_data_for_colab/xcms_example_data/PeakDetectiveObject/'); peak_curated, peak_scores, peak_intensities = integ.curatePeaks(raw_data, peaklist, threshold=0.8, align=True)
```

## Evaluation signals

- Mean fraction of all XCMS-detected peaks classified as true peaks is ~66.7 +/- 0.34%
- Mean fraction of spiked-metabolite features classified as true peaks is ~83.9 +/- 0.81%
- Spiked-feature recall significantly exceeds the overall true-peak fraction (reported p<0.0001)
- All output files (peaks_curated.csv, peak_scores.csv, peak_intensities.csv) are written and peak counts match the input peak list
- Classification cutoff used is 0.8 and ms1ppm tolerance is 25.5, matching the reported workflow

## Limitations

- Recall estimates depend on the chosen cutoff (0.8 here); inspecting TPR/FDR curves from validation peaks is required to set the threshold for the desired specificity/sensitivity tradeoff
- Performance is tailored to specific LC/MS methods and sample types — a model trained on one dataset type may not transfer to another
- Reported means carry non-zero variance (+/- 0.34% and +/- 0.81%), so single-run fractions may deviate from published values

## Evidence

- [results] spiked-standard validation result: [evidence span withheld — anchor status: not_found]
