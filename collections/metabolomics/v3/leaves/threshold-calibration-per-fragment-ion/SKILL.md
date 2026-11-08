---
name: threshold-calibration-per-fragment-ion
description: Use when per-m/z-bin binary classifiers output probabilities that must
  be turned into presence/absence peak calls for MS/MS spectra reconstruction, using
  SingleFrag-style validation-calibrated thresholds.
when_to_use_negative:
- You need calibrated peak intensities rather than presence/absence peak calls — SingleFrag
  deliberately predicts peak presence instead of intensity.
- You only need the pretrained ANN prediction path via spectraPrediction.ipynb, where
  thresholds are already shipped in the Figshare modelData folder.
- Your target m/z range falls outside the 1,000 most frequented training-set bins,
  for which no threshold is defined.
edam_topics:
- http://edamontology.org/topic_0219
- http://edamontology.org/topic_3318
tools:
- name: SingleFrag
  role: Provides the per-bin binary classifiers (ANN, GNN, COM) whose validation probabilities
    are thresholded, plus the shipped thresholdsANN/GNN/COM.pickle files
  repo: https://github.com/MaribelPR/SingleFrag
- name: mol2vec
  role: Generates the 300-dimensional SMILES embedding that feeds the ANN/COM classifiers
    whose predictions get thresholded
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1101/2024.11.04.621329
    title: 'SingleFrag: A deep learning tool for MS/MS fragment and spectral prediction
      and metabolite annotation'
  source_content_hash: sha256:053092889d8ca08b68654102ab3925cd02fc206dfd58f9de062ce17b87265efa
schema_version: 0.2.0
metadata:
  license_tier: restricted
  repo_url: https://github.com/MaribelPR/SingleFrag
  tools:
  - SingleFrag
  - mol2vec
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0219
  - http://edamontology.org/topic_3318
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# threshold-calibration-per-fragment-ion

> **License: restricted** — no clear open-source license detected for the underlying tool; verify licensing before commercial use or redistribution. <!-- asb-license-banner -->
## Summary

Calibrates a binary decision threshold for each of the 1,000 most frequented m/z bins of a tandem mass spectrum so that per-bin classifier probabilities from SingleFrag's ANN/GNN/COM models are converted into peak-present/absent calls before spectra are reconstructed.

## When to use

Use when per-m/z-bin binary classifiers output probabilities that must be turned into presence/absence peak calls for MS/MS spectra reconstruction, using SingleFrag-style validation-calibrated thresholds.

## When NOT to use

- You need calibrated peak intensities rather than presence/absence peak calls — SingleFrag deliberately predicts peak presence instead of intensity.
- You only need the pretrained ANN prediction path via spectraPrediction.ipynb, where thresholds are already shipped in the Figshare modelData folder.
- Your target m/z range falls outside the 1,000 most frequented training-set bins, for which no threshold is defined.

## Inputs

- Validation-split spectra with binarized peak presence/absence for the 1,000 most frequented m/z positions
- Per-bin classifier probabilities from ANNmodel.py, GNNmodel.py, or COMmodel.py outputs
- Trained per-bin model files (ModelVal*.pickle) from the SingleFrag Figshare deposit

## Outputs

- thresholdsANN.pickle, thresholdsGNN.pickle, thresholdsCOM.pickle (one calibrated threshold per m/z bin)
- Per-m/z-position prediction files results/dfPredictionPeak_***.pickle
- Reconstructed predicted spectra results/predSpectra.json

## How to apply

SingleFrag trains one binary classifier per m/z bin (ANN, GNN, or combined model) rather than predicting peak intensities, so each bin needs its own decision threshold instead of a global cutoff. Calibrate the thresholds using the validation spectra, storing them per model family as thresholdsANN.pickle, thresholdsGNN.pickle, and thresholdsCOM.pickle under data/modelData/. At prediction time, load the pickle matching the model variant and call a peak present in a bin when the classifier probability is greater than or equal to that bin's threshold. Thresholds cover only the most frequented 1,000 m/z positions in the training set, so restrict peak calls to those bins. After thresholding, write the per-position predictions (results/dfPredictionPeak_***.pickle) and reconstruct the full predicted spectrum (results/predSpectra.json). All pickle files were created with Python 3.8.19, so use a compatible environment to avoid deserialization issues.

## Related tools

- **SingleFrag** (Provides the per-bin binary classifiers (ANN, GNN, COM) whose validation probabilities are thresholded, plus the shipped thresholdsANN/GNN/COM.pickle files) — https://github.com/MaribelPR/SingleFrag
- **mol2vec** (Generates the 300-dimensional SMILES embedding that feeds the ANN/COM classifiers whose predictions get thresholded)

## Examples

```
import pickle; thresholds = pickle.load(open('data/modelData/thresholdsANN.pickle','rb')); peak_called = pred[mz] >= thresholds[mz]
```

## Evaluation signals

- Each threshold pickle contains exactly one threshold per entry, matching the 1,000 most frequented m/z training positions.
- Thresholded calls reproduce the reported mean F1 on the 10% test split: 0.411 (ANN), 0.399 (combined), 0.290 (GNN).
- A peak is called in a bin only when the model prediction is >= the bin's own threshold, as defined for thresholdsANN.pickle.
- Reconstructed spectrum (predSpectra.json) contains peaks only at bins covered by a threshold entry.
- Pickle files load cleanly under Python 3.8.19, matching the version used by the authors.

## Limitations

- Thresholds exist only for the most frequented 1,000 m/z positions in the training set; rarer bins cannot be thresholded or predicted.
- Authors deliberately kept model architectures simple and unoptimized, so per-bin performance (e.g., GNN mean F1 of 0.290) is a lower bound rather than a ceiling.
- The toy input spectrum format contains only integer m/z values, while the models predict fragments with 2-decimal resolution.

## Evidence

- [methods] Workflow card describing threshold calibration step: [evidence span withheld — anchor status: not_found]
