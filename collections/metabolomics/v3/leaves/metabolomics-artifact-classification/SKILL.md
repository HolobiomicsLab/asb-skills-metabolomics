---
name: metabolomics-artifact-classification
description: Use when untargeted metabolomics peak lists (e.g., XCMS centWave output
  from mzML files) need curation to remove EIC artifacts, or when quantifying what
  fraction of detected features are artifacts.
when_to_use_negative:
- Input is already a manually curated or validated feature table with trusted annotations
  — classification would add little and risk discarding true peaks
- Data are not LC/MS untargeted metabolomics profiles; PeakDetective was tailored
  to LC/MS methods and must be retrained per sample type
- You only need peak detection without curation — running XCMS alone suffices and
  avoids the active-learning labeling burden
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: PeakDetective
  role: Semisupervised deep learning package that trains an autoencoder smoother and
    active-learning classifier to score and curate peaks as artifacts or true peaks
  repo: https://github.com/pattilab/PeakDetective
- name: XCMS
  role: R-based peak detection producing the centWave peak list that PeakDetective
    curates; run via detection_helper.PeakList().runXCMS with IPO-optimized parameters
- name: pyteomics
  role: Required Python dependency used by PeakDetective for reading raw mass-spectrometry
    data files
- name: numpy
  role: Required Python dependency for array handling in the PeakDetective workflow
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/acs.analchem.3c00764
    title: 'PeakDetective: A Semisupervised Deep Learning-Based Approach for Peak
      Curation in Untargeted Metabolomics'
  source_content_hash: sha256:a5135f2c184e5792da86b116f48a4a9a4c6ece1b843d18cf562917ae758a5a4e
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
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7058823529411765
n_claims: 17
---

# metabolomics-artifact-classification

## Summary

Classify XCMS-detected LC/MS peaks as artifacts or true peaks using PeakDetective's semisupervised autoencoder-plus-classifier workflow with active learning, and quantify artifact fractions in peak lists.

## When to use

Use when untargeted metabolomics peak lists (e.g., XCMS centWave output from mzML files) need curation to remove EIC artifacts, or when quantifying what fraction of detected features are artifacts.

## When NOT to use

- Input is already a manually curated or validated feature table with trusted annotations — classification would add little and risk discarding true peaks
- Data are not LC/MS untargeted metabolomics profiles; PeakDetective was tailored to LC/MS methods and must be retrained per sample type
- You only need peak detection without curation — running XCMS alone suffices and avoids the active-learning labeling burden

## Inputs

- Raw LC/MS mzML files
- XCMS centWave peak list (mz, rt columns as CSV)
- User labels for a small set of validation/training peaks (<100) during active learning
- CentWave parameters: ms1ppm, peakWidth, s2n, noise, mzDiff, prefilter

## Outputs

- Curated peak list with artifact/true-peak classification
- Per-peak classifier scores (peak_scores)
- Peak intensities matrix
- Artifact fraction summary per dataset (e.g., artifact_fractions.csv)

## How to apply

Install PeakDetective via pip and install the R packages needed for XCMS with detection_helper.PeakList().installRPackages(). Run XCMS peak detection on mzML files in the appropriate polarity mode using tuned centWave parameters (e.g., ms1ppm 25.5, peakWidth 13.8-114.6 s, s2n 13.6, noise 1, mzDiff 0.0144, prefilter 5 consecutive scans), then load raw data with PeakDetective.rawData().readRawDataFile at the same 25.5 ppm m/z tolerance. Generate EIC data matrices with makeDataMatrix (with optional RT alignment), train the smoother (trainSmoother, min_peaks 100000, smooth_epochs 10), and train the classifier with active learning (trainClassifierActive), labeling fewer than 100 peaks over 5-10 iterations until a bimodal score distribution appears. Set the score cutoff based on FDR/TPR curves (e.g., cutoff = 0.8), then apply curatePeaks to split the peak list into curated peaks, per-peak scores, and intensities; compute per-dataset artifact percentages by labeling annotated features as artifacts or true peaks.

## Related tools

- **PeakDetective** (Semisupervised deep learning package that trains an autoencoder smoother and active-learning classifier to score and curate peaks as artifacts or true peaks) — https://github.com/pattilab/PeakDetective
- **XCMS** (R-based peak detection producing the centWave peak list that PeakDetective curates; run via detection_helper.PeakList().runXCMS with IPO-optimized parameters)
- **pyteomics** (Required Python dependency used by PeakDetective for reading raw mass-spectrometry data files)
- **numpy** (Required Python dependency for array handling in the PeakDetective workflow)

## Examples

```
integ.trainClassifierActive(X,[],[],minClassifierTrainEpochs,maxClassifierTrainEpochs,classifierBatchSize,randomRestarts,numVal=numValPeaks,numManualPerRound=numPeaksPerRound,inJupyter=True)
```

## Evaluation signals

- Mean artifact percentage across the four Metabolomics Workbench datasets should match the reported 44.3% even with IPO-optimized centWave parameters
- Classifier training shows a bimodal distribution of scores after 5-10 active-learning iterations and val_mean_absolute_error drops dramatically
- FDR and TPR estimates are reported per cutoff on the validation peaks; the chosen cutoff (e.g., 0.8) must be justified by these curves
- Per-feature scores are written to peak_scores.csv and per-dataset artifact fractions to artifact_fractions.csv with internally consistent row counts
- Annotated features match XCMS-detected peaks within the 25.5 ppm m/z tolerance and retention time before labeling

## Limitations

- The classifier must be tailored to each LC/MS method and sample type to maximize performance, so models trained on one dataset may not transfer
- R packages for XCMS/mz.unity take >20 minutes to install and require R (>v4.0) on the path
- Active learning requires manual user labeling of peaks (numValPeaks 10, 10 peaks per round), which adds interactive overhead
- Artifact fraction remains high (~44.3%) even with IPO-optimized centWave parameters, so detection-tuning alone does not solve artifact contamination

## Evidence
