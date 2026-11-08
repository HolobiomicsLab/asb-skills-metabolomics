---
name: active-learning-classifier-training
description: Use when curating LC/MS peak lists (mz/rt pairs from XCMS) whose peaks
  must be classified as artifacts or true peaks, and only a small manual labeling
  budget (<100 peaks) is available; especially when supervised baselines trained on
  fixed annotation sets underperform.
when_to_use_negative:
- Input is already a curated feature table with no raw mzML files — EIC extraction
  and smoother training cannot be performed
- A large, high-quality manually annotated peak set exists and a fully supervised
  classifier is preferred (the paper shows supervised training on 250 peaks as a distinct
  baseline)
- The scoring histogram never becomes bimodal after many active-learning rounds, indicating
  the peak set or labels are too noisy to train on reliably
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3474
tools:
- name: PeakDetective
  role: Python package providing makeDataMatrix, trainSmoother, trainClassifierActive,
    and save/load for the active-learning classifier workflow
  repo: https://github.com/pattilab/PeakDetective
- name: TensorFlow
  role: Backend for the autoencoding convolutional network and classifier training
- name: scikit-learn
  role: Used to compute ROC curves and AUC when evaluating the actively trained classifier
    against baselines
- name: XCMS
  role: Generates the input peak list (mz, rt columns) that the classifier is trained
    to curate
provenance:
  source_task_ids:
  - task_002
  - task_008
  - task_004
  source_papers:
  - doi: 10.1021/acs.analchem.3c00764
    title: 'PeakDetective: A Semisupervised Deep Learning-Based Approach for Peak
      Curation in Untargeted Metabolomics'
  source_content_hash: sha256:daa51a4949b25181f405e678bfeb2aa01f7c1cbd63cc76ec18013c862c4f9141
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
  - matplotlib
  - numpy
  - pyteomics
  - scikit-learn
  edam_operation: http://edamontology.org/operation_3927
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5625
n_claims: 16
---

# active-learning-classifier-training

## Summary

Train PeakDetective's artifact-vs-true-peak classifier through iterative manual labeling of small batches of peaks (active learning), so that a discriminative model is learned with fewer than 100 user-labeled peaks in minutes rather than a large supervised annotation set.

## When to use

Use when curating LC/MS peak lists (mz/rt pairs from XCMS) whose peaks must be classified as artifacts or true peaks, and only a small manual labeling budget (<100 peaks) is available; especially when supervised baselines trained on fixed annotation sets underperform.

## When NOT to use

- Input is already a curated feature table with no raw mzML files — EIC extraction and smoother training cannot be performed
- A large, high-quality manually annotated peak set exists and a fully supervised classifier is preferred (the paper shows supervised training on 250 peaks as a distinct baseline)
- The scoring histogram never becomes bimodal after many active-learning rounds, indicating the peak set or labels are too noisy to train on reliably

## Inputs

- mzML raw LC/MS files loaded via rawData.readRawDataFile with ms1ppm tolerance (e.g. 25.5 ppm)
- XCMS-generated peak list CSV with mz and rt columns (e.g. peaks_formatted.csv)
- EIC data matrix from integ.makeDataMatrix (n x 60 sampled datapoints per peak)
- Trained smoother autoencoder weights from integ.trainSmoother

## Outputs

- Trained PeakDetective classifier weights (saved via integ.save into the PeakDetectiveObject directory)
- Per-peak artifact/true-peak scores with bimodal score histogram
- FDR and TPR estimates on validation peaks for each candidate cutoff
- Chosen classification cutoff (e.g. 0.8)

## How to apply

First build the EIC data matrix with PeakDetective.makeDataMatrix (resolution=60 points per 1-min EIC window, align=True for RT alignment) and train the smoother autoencoder (trainSmoother with min_peaks=100000, smooth_epochs=10, batchSizeSmoother=64, validationSplitSmoother=0.1). Then call integ.trainClassifierActive: first label numValPeaks=10 validation peaks, then label 10 peaks per active-learning round (minClassifierTrainEpochs=200, maxClassifierTrainEpochs=1000, classifierBatchSize=4, randomRestarts=1), inspecting the score histogram each round until it becomes bimodal (typically after 5-10 iterations, keeping total labeled peaks under 100) and val_mean_absolute_error drops. After training, FDR and TPR estimates on the held-out validation peaks are reported for each cutoff; inspect the plotted TPR/FDR curves to choose a scoring cutoff (e.g. 0.8). The rationale is that the user labels only the most informative peaks each round, matching the semisupervised autoencoder latent representation (n x 5 from n x 60 EICs) to a classifier trained with minimal supervision.

## Related tools

- **PeakDetective** (Python package providing makeDataMatrix, trainSmoother, trainClassifierActive, and save/load for the active-learning classifier workflow) — https://github.com/pattilab/PeakDetective
- **TensorFlow** (Backend for the autoencoding convolutional network and classifier training)
- **scikit-learn** (Used to compute ROC curves and AUC when evaluating the actively trained classifier against baselines)
- **XCMS** (Generates the input peak list (mz, rt columns) that the classifier is trained to curate)

## Examples

```
integ = PeakDetective.PeakDetective(numCores=10, resolution=60); X = integ.makeDataMatrix(raw_data, peaklist['mz'].values, peaklist['rt'].values, align=True); integ.trainClassifierActive(X, [], [], 200, 1000, 4, 1, numVal=10, numManualPerRound=10, inJupyter=True)
```

## Evaluation signals

- Score histogram of all peaks becomes clearly bimodal after 5-10 active-learning iterations
- val_mean_absolute_error drops dramatically between rounds
- Total manually labeled peaks stays under 100 (10 validation peaks + 10 per round)
- FDR and TPR estimates on validation peaks are reported and consistent with the chosen cutoff (e.g. 0.8)
- On Dataset 1, the active-learning classifier's ROC/AUROC exceeds the supervised model trained on 250 peaks and the CV-filter, peak-area, and NeatMS baselines

## Limitations

- Requires interactive manual labeling each round (inJupyter=True prompts); label quality depends on user judgment
- Model parameters such as resolution=60 and min_peaks=100000 are marked 'not recommended to change' in the README
- Cutoff selection (e.g. 0.8) is made by visual inspection of TPR/FDR curves, introducing user-dependent specificity/sensitivity trade-offs

## Evidence

- [abstract] Active learning trains classifier with <100 labeled peaks: "Through active learning, the classifier is trained with less than 100 user-labeled peaks in a matter of minutes"
- [abstract] Autoencoder latent representation of each peak: "First, an unsupervised autoencoder is used to extract a low-dimensional, latent representation of each peak"
- [readme] README describes training loop and bimodal histogram criterion: "After 5-10 iterations, a bimodal distribution of scores will be produced and the val_mean_absolute_error will drop dramatically."
- [readme] README: FDR/TPR per cutoff guides threshold choice: "the FDR and TPR estimates based on the validation data will be reported for each cutoff"
- [readme] README classifier training invocation: "integ.trainClassifierActive(X,[],[],minClassifierTrainEpochs,maxClassifierTrainEpochs,classifierBatchSize,randomRestarts,numVal = numValPeaks,numManualPerRound=numPeaksPerRound,inJupyter=True)"
