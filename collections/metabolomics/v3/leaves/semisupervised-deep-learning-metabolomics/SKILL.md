---
name: semisupervised-deep-learning-metabolomics
description: Use when untargeted metabolomics peak lists (e.g., XCMS output from mzML
  raw files) contain artifacts that degrade downstream statistics, and manual curation
  of every peak is infeasible.
when_to_use_negative:
- The input is already a curated feature table with no raw mzML data available to
  extract EICs
- A fully labeled training set is available and a supervised classifier is preferred
  over active learning
- Only simple threshold filtering (CV of QC peak areas or peak-area magnitude) is
  needed, though benchmarks show these baselines underperform active-learning PeakDetective
  in AUROC
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3474
tools:
- name: PeakDetective
  role: 'Core semisupervised deep learning package: builds EIC data matrices, trains
    the smoother and active-learning classifier, and curates peaks'
  repo: https://github.com/pattilab/PeakDetective
- name: XCMS
  role: Generates the initial peak list from raw mzML data (centWave with s2n=13.6,
    prefilter=5, ms1ppm=25.5) prior to curation
- name: NeatMS
  role: Pre-trained deep learning baseline comparator for peak curation quality assessment
- name: TensorFlow
  role: Backend for training the autoencoder smoother and classifier neural networks
- name: scikit-learn
  role: Computes ROC curves and AUC values against manual artifact/true-peak annotations
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1021/acs.analchem.3c00764
    title: 'PeakDetective: A Semisupervised Deep Learning-Based Approach for Peak
      Curation in Untargeted Metabolomics'
  source_content_hash: sha256:1d2a517b990be4dd50114230851166441370167d80de69001aafc9c2657d51ae
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/pattilab/PeakDetective
  tools:
  - NeatMS
  - PeakDetective
  - TensorFlow
  - XCMS
  - matplotlib
  - numpy
  - scikit-learn
  edam_operation: http://edamontology.org/operation_3799
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5333333333333333
n_claims: 15
---

# semisupervised-deep-learning-metabolomics

## Summary

Apply PeakDetective, a semisupervised deep learning pipeline, to classify detected LC/MS peaks as artifacts or true peaks in untargeted metabolomics, using an unsupervised autoencoder smoother plus an active-learning classifier trained with fewer than 100 user-labeled peaks.

## When to use

Use when untargeted metabolomics peak lists (e.g., XCMS output from mzML raw files) contain artifacts that degrade downstream statistics, and manual curation of every peak is infeasible.

## When NOT to use

- The input is already a curated feature table with no raw mzML data available to extract EICs
- A fully labeled training set is available and a supervised classifier is preferred over active learning
- Only simple threshold filtering (CV of QC peak areas or peak-area magnitude) is needed, though benchmarks show these baselines underperform active-learning PeakDetective in AUROC

## Inputs

- mzML raw LC/MS data files
- XCMS-generated peak list (CSV with mz and rt columns)
- ppm tolerance (ms1ppm, e.g., 25.5) and RT alignment flag
- User labels for a small number of peaks (10 validation peaks, 10 per active learning round, <100 total)

## Outputs

- Curated peak list with artifact/true-peak classification
- Peak scores (continuous artifact-vs-true scores with a user-set cutoff, e.g., 0.8)
- Peak intensities matrix
- Histogram of score distributions and FDR/TPR estimates per cutoff on validation peaks
- Saved PeakDetective model object (integ.save to PeakDetectiveObject/)

## How to apply

Load the mzML raw files and the XCMS peak list (ms1ppm = 25.5 ppm tolerance) and build the extracted-ion-chromatogram (EIC) data matrix with PeakDetective.makeDataMatrix with RT alignment (align=True). Train the autoencoder smoother with trainSmoother using min_peaks = 100000, smooth_epochs = 10, batch size 64, and validation split 0.1. Then train the classifier with trainClassifierActive using active learning rounds of 10 manually labeled peaks per iteration (classifier batch size 4, 200–1000 epochs) until the score distribution becomes bimodal and val_mean_absolute_error drops, keeping total labeled peaks under 100. Inspect the reported FDR and TPR curves on the validation peaks to set a score cutoff (e.g., cutoff = 0.8), then call curatePeaks to score peaks as true vs artifact. The rationale is that the autoencoder provides an unsupervised latent representation of each peak shape, which the classifier learns to discriminate with minimal user labeling.

## Related tools

- **PeakDetective** (Core semisupervised deep learning package: builds EIC data matrices, trains the smoother and active-learning classifier, and curates peaks) — https://github.com/pattilab/PeakDetective
- **XCMS** (Generates the initial peak list from raw mzML data (centWave with s2n=13.6, prefilter=5, ms1ppm=25.5) prior to curation)
- **NeatMS** (Pre-trained deep learning baseline comparator for peak curation quality assessment)
- **TensorFlow** (Backend for training the autoencoder smoother and classifier neural networks)
- **scikit-learn** (Computes ROC curves and AUC values against manual artifact/true-peak annotations)

## Examples

```
integ.trainClassifierActive(X,[],[],minClassifierTrainEpochs,maxClassifierTrainEpochs,classifierBatchSize,randomRestarts,numVal=numValPeaks,numManualPerRound=numPeaksPerRound,inJupyter=True)
```

## Evaluation signals

- After 5-10 active learning iterations, the score histogram becomes bimodal and val_mean_absolute_error drops dramatically
- The active-learning PeakDetective ROC curve (AUROC against manual annotations) lies above supervised-with-250-peaks, CV-of-QC-peak-area, peak-area-magnitude, and pre-trained NeatMS baselines
- Total user-labeled peaks remain under 100 while training completes in minutes
- FDR and TPR estimates based on validation peaks are reported for each cutoff and match the user's specificity/sensitivity preference
- The trained model is saved and reloadable via integ.load from the PeakDetectiveObject/ directory

## Limitations

- Requires manual labeling interaction: the user must label 10 validation peaks plus 10 peaks per active-learning round until bimodality is achieved
- Classifier training is stochastic; randomRestarts parameter controls restarts and results may vary
- Running XCMS or mz.unity requires R (>v4.0) installed and R package installation takes >20 min
- Model parameters (resolution=60, window=1.0 min) are documented as 'not recommended to change'

## Evidence
