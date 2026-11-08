---
name: peak-detection-sensitivity-benchmarking
description: Use when you need to quantify how correctly reported chromatographic
  peaks depend on training data set choice (e.g., HILIC vs reversed-phase vs combined)
  and ML algorithm (NNet, RF, SVM, XGB) for automated LC-MRM-MS preprocessing.
when_to_use_negative:
- The analysis goal is deploying a single production model on a batch, not comparing
  model variants — use process_batch() with one trained model instead of the benchmarking
  loop.
- The input data lacks expert ground-truth labels for the training data sets, so correct-reporting
  rates cannot be verified against manually defined correct peaks.
- The raw data is not LC-MS data recorded in MRM mode (automRm is built for MRM-mode
  preprocessing on QQQ instruments).
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3474
tools:
- name: automRm
  role: Provides train_model() to train the peak picking and peak reporting ML models
    and process_batch() to preprocess the held-out batch for correct-reporting evaluation
  repo: https://gitlab.gwdg.de/joerg.buescher/automrm
- name: caret
  role: Machine learning backend for training NNet, RF, SVM, and XGB model variants
    with hyperparameter tuning and an 80%/20% training/validation split
- name: mzR
  role: Parses the mzML raw files of the held-out batch fed into process_batch()
- name: openxlsx
  role: Reading and writing of xlsx files for tabulating benchmark results
- name: R
  role: Implementation environment for the automRm benchmarking workflow (tested in
    R version 3.6.0 and 4.0.2)
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1021/acs.analchem.1c05224
    title: 'automRm: An R Package for Fully Automatic LC-QQQ-MS Data Preprocessing
      Powered by Machine Learning'
  source_content_hash: sha256:c4b1240be81a29d65a134b23aa92fa799463ad861862b35ef3d5966952ae3703
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - automRm
  - caret
  - mzR
  - openxlsx
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 14
---

# peak-detection-sensitivity-benchmarking

## Summary

Benchmark how sensitive automated LC-QQQ-MS peak reporting is to the choice of training data set and machine learning algorithm by training peak picking and peak reporting models across data-set × algorithm combinations and measuring correct-reporting rates on a held-out batch.

## When to use

Use when you need to quantify how correctly reported chromatographic peaks depend on training data set choice (e.g., HILIC vs reversed-phase vs combined) and ML algorithm (NNet, RF, SVM, XGB) for automated LC-MRM-MS preprocessing.

## When NOT to use

- The analysis goal is deploying a single production model on a batch, not comparing model variants — use process_batch() with one trained model instead of the benchmarking loop.
- The input data lacks expert ground-truth labels for the training data sets, so correct-reporting rates cannot be verified against manually defined correct peaks.
- The raw data is not LC-MS data recorded in MRM mode (automRm is built for MRM-mode preprocessing on QQQ instruments).

## Inputs

- Training data sets of peak-picking and peak-reporting examples with expert ground-truth labels (e.g., 7817 positive / 32032 negative peak-picking examples and 4987 positive / 4164 negative peak-reporting examples from four LC-QQQ-MS methods)
- Held-out batch of LC-QQQ-MS raw data in mzML format (e.g., recorded on an Agilent 1290 Infinity II UHPLC in line with an Agilent 6495 QQQ-MS)
- List of caret algorithms to compare (NNet, RF, SVM, XGB)

## Outputs

- Correct-reporting rate (fraction of correctly reported peaks) per training-data-set/algorithm combination
- Comparison table of correct-reporting rates (saved as correct_peak_reporting_comparison.csv)
- Trained peak picking and peak reporting model variants for each combination

## How to apply

Load a combined training data set (48 samples from four LC-QQQ-MS methods, with 7817 positive / 32032 negative peak-picking examples and 4987 positive / 4164 negative peak-reporting examples) plus the individual data sets (HILIC 1a, HILIC 1b, HILIC 2, reversed-phase) and their expert ground-truth labels. For each training data set, train peak picking and peak reporting models with train_model() in automRm using four caret algorithms — artificial neural network (NNet), random forest (RF), support vector machine (SVM), and extreme gradient boosting (XGB) — with hyperparameter tuning and an 80%/20% training/validation split. For each trained model variant, run process_batch() on a held-out batch of raw data (mzML files parsed via mzR) and record for every metabolite-sample pair whether the reported peak matches the manually defined correct peak. Compute the correct-reporting rate (fraction of correctly reported peaks) for each training-data-set/algorithm combination and tabulate the rates. Validate the outcome against the expected behavior: random forest and extreme gradient boosting nearly perfectly classify the peak-reporting task, and the combined training data set is the most generally applicable variant.

## Related tools

- **automRm** (Provides train_model() to train the peak picking and peak reporting ML models and process_batch() to preprocess the held-out batch for correct-reporting evaluation) — https://gitlab.gwdg.de/joerg.buescher/automrm
- **caret** (Machine learning backend for training NNet, RF, SVM, and XGB model variants with hyperparameter tuning and an 80%/20% training/validation split)
- **mzR** (Parses the mzML raw files of the held-out batch fed into process_batch())
- **openxlsx** (Reading and writing of xlsx files for tabulating benchmark results)
- **R** (Implementation environment for the automRm benchmarking workflow (tested in R version 3.6.0 and 4.0.2))

## Examples

```
devtools::install_gitlab('joerg.buescher/automRm@master', host = 'https://gitlab.gwdg.de')
```

## Evaluation signals

- RF and XGB variants should nearly perfectly classify the peak-reporting task on the held-out batch, matching the paper's reported behavior.
- The combined training set variant should yield the highest and most generally applicable correct-reporting rate compared with the individual HILIC 1a, HILIC 1b, HILIC 2, and reversed-phase training sets.
- Every metabolite-sample pair in the held-out batch must have a recorded match/mismatch against the manually defined correct peak before rates are computed.
- The comparison table must contain one correct-reporting rate per training-data-set × algorithm combination (5 data sets × 4 algorithms).
- Top-performing variants' rates should be cross-checked against the claim that automRm results can reach quality similar to manual peak review given sufficient relevant training data.

## Limitations

- The benchmark depends on expert ground-truth labels; automRm users otherwise retreat to manual peak review for selected metabolites in special cases such as excessively dirty samples, very-low-input samples, or new LC methods.
- Model applicability is method-dependent: the combined training set is the most generally applicable, but sufficient training on relevant raw data is required to reach quality similar to manual peak review.
- Adding qualifiers to the acquisition method reduces the number of analytes that can be detected simultaneously in MRM mode, which constrains the feature set available for peak reporting.

## Evidence

- [abstract] We demonstrate the impact of the choice of training data set, of the applied ML algorithm, and of individual peak characteristics on automRm's ability to correctly report chromatographic peaks.: "We demonstrate the impact of the choice of training data set, of the applied ML algorithm, and of individual peak characteristics on automRm's ability to correctly report chromatographic peaks."
- [methods] train_model() for training the ML models used in process_batch(): "train_model() for training the ML models used in process_batch()"
- [discussion] With sufficient training on relevant raw data, results from automRm can reach a similar level of quality as manual peak review.: "With sufficient training on relevant raw data, results from automRm can reach a similar level of quality as manual peak review."
- [abstract] automRm can replicate results obtained by manual peak review on published data.: "automRm can replicate results obtained by manual peak review on published data."
