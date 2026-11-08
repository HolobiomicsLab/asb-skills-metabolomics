---
name: hyperparameter-tuning-for-peak-models
description: Use when training or selecting peak picking / peak reporting models for
  automRm and you need to benchmark hyperparameter tuning, training-data-set choice
  (individual vs combined LC-QQQ-MS method sets), and caret algorithm choice.
when_to_use_negative:
- The acquisition method lacks at least one qualifier in addition to the quantifier
  — automRm's optimal performance requires qualifiers, so tuning gains will not translate
  into correct reporting.
- Raw data come from excessively dirty samples, very-low-input samples, or entirely
  new LC methods with no relevant training data — the article advises retreating to
  manual peak review instead.
- Input is already preprocessed or labeled — model training with train_model() requires
  raw MRM-mode chromatogram data with ground-truth peak annotations.
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_3474
tools:
- name: automRm
  role: R package whose train_model() trains the peak picking and peak reporting models
    and whose process_batch() evaluates each variant on held-out batches
  repo: https://gitlab.gwdg.de/joerg.buescher/automrm
- name: caret
  role: Supplies the machine learning algorithms (NNet, RF, SVM, XGB), hyperparameter
    tuning, and train/validation splitting
- name: mzR
  role: Parses mzML raw files into chromatogram data for model training and batch
    preprocessing
- name: openxlsx
  role: Writes the comparison results tables (xlsx) produced by the benchmark
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1021/acs.analchem.1c05224
    title: 'automRm: An R Package for Fully Automatic LC-QQQ-MS Data Preprocessing
      Powered by Machine Learning'
  source_content_hash: sha256:0ffd9f6b8765f2372b1ff1f6dfffeca166294455a87ae471e28bf75c2adeb8bf
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
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 20
---

# hyperparameter-tuning-for-peak-models

## Summary

Trains and tunes automRm peak picking and peak reporting ML models across multiple caret algorithms (NNet, RF, SVM, XGB) and training data sets, then compares correct-reporting rates on held-out LC-QQQ-MS batches to select the best model variant.

## When to use

Use when training or selecting peak picking / peak reporting models for automRm and you need to benchmark hyperparameter tuning, training-data-set choice (individual vs combined LC-QQQ-MS method sets), and caret algorithm choice.

## When NOT to use

- The acquisition method lacks at least one qualifier in addition to the quantifier — automRm's optimal performance requires qualifiers, so tuning gains will not translate into correct reporting.
- Raw data come from excessively dirty samples, very-low-input samples, or entirely new LC methods with no relevant training data — the article advises retreating to manual peak review instead.
- Input is already preprocessed or labeled — model training with train_model() requires raw MRM-mode chromatogram data with ground-truth peak annotations.

## Inputs

- LC-QQQ-MS raw data batches in mzML format (parsed via mzR)
- Training data sets with expert ground-truth peak labels (e.g. HILIC 1a, HILIC 1b, HILIC 2, reversed-phase, and combined 48-sample sets)
- Peak-picking and peak-reporting quality scores from automRm

## Outputs

- Trained peak picking and peak reporting ML models per training-data-set/algorithm combination
- Correct-reporting rate comparison table (e.g. correct_peak_reporting_comparison.csv)
- Per metabolite-sample pair peak-reporting assignments on the held-out batch

## How to apply

Load the individual training data sets (HILIC 1a, HILIC 1b, HILIC 2, reversed-phase) plus the combined 48-sample set (7817 positive / 32032 negative peak-picking examples; 4987 positive / 4164 negative peak-reporting examples) with expert ground-truth labels. Train peak picking and peak reporting models via automRm's train_model() using four caret algorithms — artificial neural network (NNet), random forest (RF), support vector machine (SVM), and extreme gradient boosting (XGB) — with hyperparameter tuning and an 80%/20% training/validation split. Run process_batch() with each trained variant on a held-out mzML batch, and for every metabolite-sample pair record whether the reported peak matches the manually defined correct peak. Compute the correct-reporting rate (fraction of correctly reported peaks) per training-data-set/algorithm combination and tabulate results. The combined training set is expected to be the most generally applicable variant.

## Related tools

- **automRm** (R package whose train_model() trains the peak picking and peak reporting models and whose process_batch() evaluates each variant on held-out batches) — https://gitlab.gwdg.de/joerg.buescher/automrm
- **caret** (Supplies the machine learning algorithms (NNet, RF, SVM, XGB), hyperparameter tuning, and train/validation splitting)
- **mzR** (Parses mzML raw files into chromatogram data for model training and batch preprocessing)
- **openxlsx** (Writes the comparison results tables (xlsx) produced by the benchmark)

## Examples

```
devtools::install_gitlab('joerg.buescher/automRm@master', host = 'https://gitlab.gwdg.de')
```

## Evaluation signals

- Random forest and extreme gradient boosting should nearly perfectly classify the peak-reporting task; NNet and SVM correct-reporting rates should be markedly lower.
- The combined training set should yield correct-reporting rates at least as high as (and generalize better than) any single-method training set when evaluated on the held-out batch.
- Every metabolite-sample pair on the held-out batch must have a recorded correct/incorrect flag against the manually defined correct peak before rates are computed.
- Hyperparameter tuning and an 80%/20% train/validation split must be documented for every trained variant.
- Resulting comparison table rows should cover all training-data-set/algorithm combinations (5 data sets x 4 caret algorithms).

## Limitations

- Model quality approaches manual peak review only 'with sufficient training on relevant raw data' — models tuned on unrelated LC-QQQ-MS methods may not generalize.
- automRm requires at least one qualifier transition besides the quantifier; without qualifiers, tuned models cannot reach optimal reporting performance.
- Even after tuning, results 'can reach a similar level of quality as manual peak review' — manual review may still be needed for special cases such as excessively dirty or very-low-input samples.

## Evidence

- [methods] caret for ML algorithms and tuning: "caret13 for machine learning (ML)"
- [methods] train_model trains models used by process_batch: "train_model() for training the ML models used in process_batch()"
- [abstract] Impact of training data set and algorithm demonstrated: "We demonstrate the impact of the choice of training data set, of the applied ML algorithm, and of individual peak characteristics"
- [methods] Peak reporting uses peak-picking quality scores: "peak reporting ML model uses the same quality scores that were previously used by the peak picking ML model"
- [discussion] Sufficient training needed for manual-review quality: "With sufficient training on relevant raw data, results from automRm can reach a similar level of quality as manual peak review."
