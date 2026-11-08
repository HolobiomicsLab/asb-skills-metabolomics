---
name: peak-artifact-classification
description: Use when you have raw mzML files plus a detected peak list (mz, rt columns,
  e.g. from XCMS) and need to separate genuine metabolite peaks from artifacts without
  large manually annotated training sets.
when_to_use_negative:
- Input is already a curated, manually annotated feature table where peak labels are
  known — supervised training or direct filtering is more appropriate than active
  learning.
- Data lacks raw mzML scans (only processed peak intensities are available), since
  EIC extraction from raw data is required to build the n x 60 input matrix.
- Only a very small peak list is available (far below the min_peaks=100000 used to
  train the smoother autoencoder), which undermines unsupervised latent-space learning.
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3474
tools:
- name: PeakDetective
  role: Core Python package implementing EIC extraction, the autoencoding smoother,
    the active-learning classifier, and curatePeaks scoring
  repo: https://github.com/pattilab/PeakDetective
- name: XCMS
  role: Upstream R peak detection used to generate the mz/rt peak list that PeakDetective
    then curates (run via detection_helper.PeakList().runXCMS)
- name: NeatMS
  role: Baseline pre-trained peak-curation tool against which PeakDetective's AUROC
    was compared on validation datasets
- name: pyteomics
  role: Dependency used to parse raw mzML scans when building rawData objects
- name: TensorFlow
  role: Backend for training the autoencoding CNN smoother and the active-learning
    classifier
- name: scikit-learn
  role: Used for ROC/AUC evaluation of classifier scores against manual annotations
- name: fastdtw
  role: Dependency supporting retention-time alignment when align=True in makeDataMatrix
provenance:
  source_task_ids:
  - task_008
  - task_002
  source_papers:
  - doi: 10.1021/acs.analchem.3c00764
    title: 'PeakDetective: A Semisupervised Deep Learning-Based Approach for Peak
      Curation in Untargeted Metabolomics'
  source_content_hash: sha256:13d8508f6034b21db484c2fe3329e1d9ef02ade80f184e520e66d43d89bfa209
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/pattilab/PeakDetective
  tools:
  - NeatMS
  - PeakDetective
  - Python
  - TensorFlow
  - XCMS
  - fastdtw
  - matplotlib
  - numpy
  - pyteomics
  - scikit-learn
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 22
---

# peak-artifact-classification

## Summary

Classifies LC/MS peaks from an XCMS-style peak list as artifacts or true peaks using a semisupervised deep-learning pipeline: an autoencoding CNN compresses n x 60 EIC matrices into an n x 5 latent space, and an active-learning classifier scores peaks using fewer than 100 user-labeled peaks.

## When to use

Use when you have raw mzML files plus a detected peak list (mz, rt columns, e.g. from XCMS) and need to separate genuine metabolite peaks from artifacts without large manually annotated training sets.

## When NOT to use

- Input is already a curated, manually annotated feature table where peak labels are known — supervised training or direct filtering is more appropriate than active learning.
- Data lacks raw mzML scans (only processed peak intensities are available), since EIC extraction from raw data is required to build the n x 60 input matrix.
- Only a very small peak list is available (far below the min_peaks=100000 used to train the smoother autoencoder), which undermines unsupervised latent-space learning.

## Inputs

- mzML raw LC/MS data files (one per sample, read into PeakDetective.rawData objects with a ppm tolerance such as 25.5)
- Peak list with mz and rt columns (e.g. CSV exported from XCMS peaks_formatted.csv)
- Optional blank and sample sample-column keys for prior backgroundSubtract filtering before classification

## Outputs

- Per-peak artifact/true-peak scores (peak_scores) from the trained classifier
- Curated peak list (peak_curated) and peak intensities (peak_intensities) after applying the score cutoff via curatePeaks
- Trained smoother and classifier weights saved via integ.save into a PeakDetectiveObject directory for reuse

## How to apply

Load each mzML file into a PeakDetective.rawData object with readRawDataFile using an MS1 ppm tolerance (e.g. 25.5 ppm), and load the peak list as a two-column pandas DataFrame of mz and rt. Initialize a PeakDetective object (e.g. numCores=10, resolution=60) and build the EIC data matrix with makeDataMatrix: each 1-min EIC window (window=1.0) is sampled at 60 data points, optionally with retention-time alignment (align=True) via fastdtw, yielding an n x 60 matrix. Train the smoother autoencoder with trainSmoother (min_peaks=100000, smooth_epochs=10, batchSizeSmoother=64, validationSplitSmoother=0.1) to compress each EIC into a 5-dimensional latent representation. Then train the classifier with trainClassifierActive: label 10 validation peaks, then 10 peaks per active-learning round (classifierBatchSize=4, epochs between 200 and 1000, randomRestarts=1) until the score histogram becomes bimodal and val_mean_absolute_error drops, typically after 5-10 iterations with fewer than 100 total labeled peaks. Inspect the reported FDR/TPR curves on validation peaks to choose a scoring cutoff (e.g. 0.8), and apply it via curatePeaks to obtain curated peaks, scores, and intensities.

## Related tools

- **PeakDetective** (Core Python package implementing EIC extraction, the autoencoding smoother, the active-learning classifier, and curatePeaks scoring) — https://github.com/pattilab/PeakDetective
- **XCMS** (Upstream R peak detection used to generate the mz/rt peak list that PeakDetective then curates (run via detection_helper.PeakList().runXCMS))
- **NeatMS** (Baseline pre-trained peak-curation tool against which PeakDetective's AUROC was compared on validation datasets)
- **pyteomics** (Dependency used to parse raw mzML scans when building rawData objects)
- **TensorFlow** (Backend for training the autoencoding CNN smoother and the active-learning classifier)
- **scikit-learn** (Used for ROC/AUC evaluation of classifier scores against manual annotations)
- **fastdtw** (Dependency supporting retention-time alignment when align=True in makeDataMatrix)

## Examples

```
integ = PeakDetective.PeakDetective(numCores=10, resolution=60); X = integ.makeDataMatrix(raw_data, peaklist['mz'].values, peaklist['rt'].values, align=True); integ.trainSmoother(peaklist, raw_data, min_peaks, smooth_epochs, batchSizeSmoother, validationSplitSmoother); integ.trainClassifierActive(X, [], [], 200, 1000, 4, 1, numVal=10, numManualPerRound=10, inJupyter=True)
```

## Evaluation signals

- After each active-learning round the score histogram for all peaks should become clearly bimodal, separating true peaks from artifacts.
- val_mean_absolute_error on the 10 user-labeled validation peaks should drop dramatically relative to the first rounds once the classifier has converged.
- FDR and TPR estimates reported for each cutoff on validation peaks should show TPR rising while FDR stays low; e.g. a cutoff of 0.8 gave usable separation on the example data.
- On Dataset 1 (negative-mode human plasma HILIC/TOF), active-learning PeakDetective attained a higher AUROC for separating manually annotated true peaks from artifacts than CV-of-QC-peak-area filtering, peak-area-magnitude filtering, pre-trained NeatMS, and supervised training with 250 peaks.
- The trained smoother should reconstruct EICs such that the latent space is only 5 dimensions (n x 5) versus the 60-point input EICs, confirming successful compression.

## Limitations

- The classifier requires interactive user labeling (10 validation peaks then 10 per round), so it is not fully automated and quality depends on user judgment during active-learning rounds.
- Training the smoother autoencoder relies on a large pool of peaks (min_peaks=100000 in the reference workflow); smaller peak lists may not provide enough data.
- The EIC representation is fixed at 60 data points per 1-minute window (resolution=60), which constrains the temporal detail captured per peak.
- RT alignment (align=True) assumes retention-time drift that can be corrected by fastdtw alignment; datasets with severe misalignment may need tuning or alignment disabled.

## Evidence

- [results] Extracted ion chromatograms are concatenated into an n x 60 matrix (60 sampled datapoints per 1-min EIC window) and compressed by an autoencoding convolutional neural network into an n x 5 latent representation encoding each EIC.: [evidence span withheld — anchor status: not_found]
- [supplementary] Active learning training protocol: [evidence span withheld — anchor status: not_found]
- [results] PeakDetective comparison on Dataset 1: [evidence span withheld — anchor status: not_found]
