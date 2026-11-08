---
name: active-learning-peak-annotation
description: Use when an XCMS-generated peak list from mzML files must be curated
  into artifacts vs true peaks and the user can label only ~70–100 peaks interactively;
  active learning with a small labeling budget is required.
when_to_use_negative:
- No human is available to interactively label peaks each round — active learning
  requires user labels per iteration; a fully automated pass cannot run it
- A large trusted set of manually labeled peaks already exists and direct supervised
  training is preferable
- The peak list is already curated/filtered (e.g. post backgroundSubtract or removeRedundancy
  with accepted labels) and only thresholding is needed
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3173
- http://edamontology.org/topic_3474
tools:
- name: PeakDetective
  role: Provides trainClassifierActive for active-learning classifier training, TPR/FDR
    estimation, and curatePeaks at the chosen cutoff
  repo: https://github.com/pattilab/PeakDetective
- name: XCMS
  role: Generates the initial peak list (ms1ppm = 25.5, peakWidth = (13.8, 114.6)
    s, s2n = 13.6, prefilter = 5) that the classifier curates
- name: TensorFlow
  role: Backend for the classifier and smoother neural networks trained during active
    learning
- name: NeatMS
  role: Transfer-learning baseline against which the active-learning classifier's
    accuracy/recall/precision is benchmarked
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1021/acs.analchem.3c00764
    title: 'PeakDetective: A Semisupervised Deep Learning-Based Approach for Peak
      Curation in Untargeted Metabolomics'
  source_content_hash: sha256:3815c00ea0d1425602ce2ae3f1a2fdc2c4a644c780dd7378526d34ca734085c8
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
  - numpy
  - pandas
  - scikit-learn
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3173
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.45
n_claims: 20
---

# active-learning-peak-annotation

## Summary

Train a PeakDetective classifier to discriminate artifact peaks from true LC/MS peaks by interactively labeling fewer than 100 peaks via active learning, then set a score cutoff from TPR/FDR curves. This minimizes manual curation burden in untargeted metabolomics peak curation.

## When to use

Use when an XCMS-generated peak list from mzML files must be curated into artifacts vs true peaks and the user can label only ~70–100 peaks interactively; active learning with a small labeling budget is required.

## When NOT to use

- No human is available to interactively label peaks each round — active learning requires user labels per iteration; a fully automated pass cannot run it
- A large trusted set of manually labeled peaks already exists and direct supervised training is preferable
- The peak list is already curated/filtered (e.g. post backgroundSubtract or removeRedundancy with accepted labels) and only thresholding is needed

## Inputs

- List of PeakDetective.rawData objects parsed from mzML files with ms1ppm = 25.5
- XCMS peak list (mz, rt columns) as a pandas DataFrame
- EIC data matrix X from integ.makeDataMatrix (resolution = 60, window = 1.0 min, align = True)
- Trained PeakDetective smoother
- Interactive user labels for 10 validation peaks plus ~10 peaks per active-learning round

## Outputs

- Trained PeakDetective classifier with artifact/true-peak scores for every peak
- TPR (recall) and FDR (1-precision) curves per cutoff from the 10 validation peaks
- Curated peak list, peak scores, and peak intensities from curatePeaks at the chosen cutoff (e.g. 0.8)
- Saved model weights via integ.save() for auditability

## How to apply

After building the EIC data matrix with makeDataMatrix (ms1ppm = 25.5, resolution = 60 points/min, window = 1.0 min, align = True) and training the smoother (smooth_epochs = 10, batchSizeSmoother = 64, validationSplitSmoother = 0.1), call trainClassifierActive with numVal = 10 validation peaks, 10 peaks labeled per round, min 200 / max 1000 classifier epochs, and classifierBatchSize = 4. Label the validation peaks first, then label the peaks prompted each active-learning round; after 5–10 rounds a bimodal score distribution appears and val_mean_absolute_error drops sharply. Use the plotted TPR/FDR curves on the validation peaks to choose a cutoff (e.g. 0.8) reflecting the desired precision/recall trade-off, then apply curatePeaks with that threshold. This active-learning route needs far fewer labels (70–100) than fully supervised retraining alternatives such as NeatMS transfer learning, which needed ~400 manually labeled peaks to approach comparable accuracy on the same dataset.

## Related tools

- **PeakDetective** (Provides trainClassifierActive for active-learning classifier training, TPR/FDR estimation, and curatePeaks at the chosen cutoff) — https://github.com/pattilab/PeakDetective
- **XCMS** (Generates the initial peak list (ms1ppm = 25.5, peakWidth = (13.8, 114.6) s, s2n = 13.6, prefilter = 5) that the classifier curates)
- **TensorFlow** (Backend for the classifier and smoother neural networks trained during active learning)
- **NeatMS** (Transfer-learning baseline against which the active-learning classifier's accuracy/recall/precision is benchmarked)

## Examples

```
integ.trainClassifierActive(X,[],[],200,1000,4,1,numVal=10,numManualPerRound=10,inJupyter=True)
```

## Evaluation signals

- After 5–10 active-learning rounds, the score histogram is bimodal and val_mean_absolute_error drops dramatically
- With ~70 actively selected labeled peaks, accuracy/recall/precision on the 10 validation peaks exceeds NeatMS retrained on 70, ~300, or even ~400 manually labeled peaks (reported 63.9% at 70 and 69.3% at ~400 labels for NeatMS)
- The chosen cutoff (e.g. 0.8) sits on the plotted TPR/FDR curves at the user's desired precision (1-FDR) point
- Training and validation accuracies are reported each round and are consistent across rounds
- Trained weights load successfully via integ.load() after integ.save(), allowing the benchmark to be audited

## Limitations

- Classification quality depends on the smoothness/quality of the ~70–100 user labels; only 10 peaks serve as validation data, so TPR/FDR estimates carry sampling noise
- min_peaks = 100000 is recommended for training the autoencoder; datasets with fewer peaks may not train the smoother well
- The cutoff is chosen by visual inspection of TPR/FDR curves, so it is subjective and tuned per LC/MS method and sample type
- The active-learning benchmarking versus NeatMS was performed on specific validation datasets (e.g. Dataset 1); performance may vary on other datasets

## Evidence

- [abstract] Through active learning, the classifier is trained with less than 100 user-labeled peaks in a matter of minutes: "Through active learning, the classifier is trained with less than 100 user-labeled peaks in a matter of minutes"
- [abstract] Active learning discriminator workflow: "Second, a classifier is trained with active learning to discriminate between artifacts and true peaks"
- [readme] Bimodal score distribution convergence signal: "After 5-10 iterations, a bimodal distribution of scores will be produced and the val_mean_absolute_error will drop dramatically."
- [readme] Cutoff selection from TPR/FDR curves: "the FDR and TPR estimates based on the validation data will be reported for each cutoff. Inspection on this plot is used to set the threshold"
- [results] Accuracy advantage over retraining approaches: [evidence span withheld — anchor status: not_found]
