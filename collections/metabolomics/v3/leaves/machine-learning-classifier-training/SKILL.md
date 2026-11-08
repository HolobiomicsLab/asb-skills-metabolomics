---
name: machine-learning-classifier-training
description: Use when building or retraining a classifier (e.g., random forest peak-reporting
  model) from tabulated feature/quality scores plus curated 0/1/2 expert labels, with
  F1-validated hyperparameter tuning.
when_to_use_negative:
- Raw mzML files have not yet been preprocessed by process_batch() to produce qslog.tsv
  quality scores — train_model() cannot run on raw chromatograms
- Only a single new LC method with no expert-annotated data is available; the article
  notes automRm retreats to manual peak review for new LC methods until sufficient
  relevant training data exist
edam_operation: http://edamontology.org/operation_3791
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_3172
tools:
- name: caret
  role: R machine-learning framework used for hyperparameter tuning and training of
    the peak-picking and peak-reporting classifiers (RF, NNet, SVM, XGB)
- name: automRm
  role: Provides train_model() and process_batch(); train_model() trains the ML models
    used in process_batch()
  repo: https://gitlab.gwdg.de/joerg.buescher/automrm
- name: openxlsx
  role: Reads the manual_peakcheck.xlsx expert labels and writes the manual_peakcheck_template.xlsx
    training template
- name: R
  role: Runtime environment (developed in R 3.6.0, tested in 4.0.2) in which train_model()
    and caret execute
provenance:
  source_task_ids:
  - task_005
  - task_006
  source_papers:
  - doi: 10.1021/acs.analchem.1c05224
    title: 'automRm: An R Package for Fully Automatic LC-QQQ-MS Data Preprocessing
      Powered by Machine Learning'
  source_content_hash: sha256:39ad33ecc1a89fc5a6ac7ce0c327fde83a46a7795d93cfc913dad681b12f8c1b
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - automRm
  - caret
  - mzR
  - openxlsx
  edam_operation: http://edamontology.org/operation_3791
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# machine-learning-classifier-training

## Summary

Trains supervised ML classifiers for LC-QQQ-MS peak reporting/picking in the automRm R package by combining qslog.tsv quality scores with expert labels from manual_peakcheck.xlsx, using the caret package with hyperparameter tuning and an 80%/20% train/validation split.

## When to use

Use when building or retraining a classifier (e.g., random forest peak-reporting model) from tabulated feature/quality scores plus curated 0/1/2 expert labels, with F1-validated hyperparameter tuning.

## When NOT to use

- Raw mzML files have not yet been preprocessed by process_batch() to produce qslog.tsv quality scores — train_model() cannot run on raw chromatograms
- Only a single new LC method with no expert-annotated data is available; the article notes automRm retreats to manual peak review for new LC methods until sufficient relevant training data exist

## Inputs

- qslog.tsv containing peak-picking classification quality scores for every metabolite in every sample
- manual_peakcheck.xlsx with expert labels (0 = do not report, 1 = exclude from training, 2 = report) per metabolite-sample pair
- Training data sets of raw LC-QQQ-MS MRM data (e.g., HILIC 1a, HILIC 1b, HILIC 2, reversed-phase, or a combined 48-sample set)

## Outputs

- Serialized peak-reporting ML model on disk, later consumed by process_batch()
- Training-summary metrics, including F1 score of the trained model on the 20% validation subset
- Optionally, comparison tables of correct-reporting rates across training-set/algorithm combinations (e.g., correct_peak_reporting_comparison.csv)

## How to apply

Run automRm's process_batch() to generate qslog.tsv (peak-picking classification scores for every metabolite in every sample) and the manual_peakcheck_template.xlsx template; have an expert fill manual_peakcheck.xlsx with scores 0 (do not report), 1 (exclude from training), or 2 (report) per metabolite-sample pair. Invoke train_model() in R, which combines qslog.tsv quality scores with the manual_peakcheck.xlsx labels and splits the data 80% training / 20% validation. Train via caret with hyperparameter tuning — random forest is the article's primary choice, but NNet, SVM, and XGB were benchmarked; RF and XGB nearly perfectly classified the peak-reporting task. Serialize the trained model to disk for later use by process_batch() and record validation metrics such as the F1 score.

## Related tools

- **caret** (R machine-learning framework used for hyperparameter tuning and training of the peak-picking and peak-reporting classifiers (RF, NNet, SVM, XGB))
- **automRm** (Provides train_model() and process_batch(); train_model() trains the ML models used in process_batch()) — https://gitlab.gwdg.de/joerg.buescher/automrm
- **openxlsx** (Reads the manual_peakcheck.xlsx expert labels and writes the manual_peakcheck_template.xlsx training template)
- **R** (Runtime environment (developed in R 3.6.0, tested in 4.0.2) in which train_model() and caret execute)

## Examples

```
library(automRm); train_model()  # after process_batch() produced qslog.tsv and expert labels in manual_peakcheck.xlsx
```

## Evaluation signals

- F1 score of the trained peak-reporting model on the 20% validation subset; the paper reports F1 > 0.98 for combined models trained on combined data sets
- Correct-reporting rate (fraction of reported peaks matching the manually defined correct peak) when process_batch() is run on a held-out batch
- Random forest and XGB variants should nearly perfectly classify the peak-reporting task; large gaps vs. NNet/SVM indicate a training-data or label problem
- Serialized model file is produced and loadable by a subsequent process_batch() run
- The combined 48-sample training set (7817 positive / 32032 negative peak-picking examples; 4987 positive / 4164 negative peak-reporting examples) should yield the most generally applicable model

## Limitations

- Results reach manual-review quality only 'with sufficient training on relevant raw data'; a combined training set across LC methods is more generally applicable than single-method models
- The choice of training data set and ML algorithm demonstrably affects correct peak reporting
- automRm still retreats to manual peak review for excessively dirty samples, very-low-input samples, or new LC methods
- At least one qualifier in addition to the quantifier is required for optimal automRm performance

## Evidence

- [methods] caret for ML: "caret13 for machine learning (ML)"
- [methods] train_model purpose: "train_model() for training the ML models used in process_batch()"
- [discussion] training data sufficiency: "With sufficient training on relevant raw data, results from automRm can reach a similar level of quality as manual peak review."
- [abstract] training set / algorithm sensitivity: "We demonstrate the impact of the choice of training data set, of the applied ML algorithm, and of individual peak characteristics on automRm’s ability to correctly report chromatographic peaks."
