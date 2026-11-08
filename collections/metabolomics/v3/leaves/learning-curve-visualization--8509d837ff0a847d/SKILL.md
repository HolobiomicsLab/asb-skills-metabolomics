---
name: learning-curve-visualization
description: Use when you need to show how model performance (precision, recall) scales
  with training-set size — e.g., to justify a dataset's size or demonstrate diminishing
  returns on a spectral classification task.
when_to_use_negative:
- You want absolute model performance rather than performance-vs-data-size scaling
  — evaluate the single trained BitterMasS model instead
- The spectra are negative ion mode or not EI-MS/ESI-MS/MS positive-mode spectra,
  outside BitterMasS's stated applicability ('the model makes predictions based on
  EI-MS or ESI-MS/MS spectra obtained in positive ion mode')
edam_operation: http://edamontology.org/operation_0564
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_0121
tools:
- name: BitterMasS
  role: Random Forest classifier retrained at each training-subset size to produce
    the precision/recall points of the learning curve
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1021/acs.jafc.3c09767
    title: 'BitterMasS: Predicting Bitterness from Mass Spectra'
  source_content_hash: sha256:8e318bf770a0706da7c3d9829702b870d4240e3a100310dd1fbb8c06695ef1b5
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BitterMasS
  edam_operation: http://edamontology.org/operation_0564
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# learning-curve-visualization

## Summary

Retrain a fixed classifier (here BitterMasS, a Random Forest over 5414 MassBank EI-MS/ESI-MS/MS spectra) on progressively larger training subsets and plot precision/recall versus training-set size to characterize diminishing returns, reproducing Figure S6 of the BitterMasS paper.

## When to use

Use when you need to show how model performance (precision, recall) scales with training-set size — e.g., to justify a dataset's size or demonstrate diminishing returns on a spectral classification task.

## When NOT to use

- You want absolute model performance rather than performance-vs-data-size scaling — evaluate the single trained BitterMasS model instead
- The spectra are negative ion mode or not EI-MS/ESI-MS/MS positive-mode spectra, outside BitterMasS's stated applicability ('the model makes predictions based on EI-MS or ESI-MS/MS spectra obtained in positive ion mode')

## Inputs

- Combined bitter/nonbitter mass spectral training dataset (5414 experimental EI-MS and ESI-MS/MS positive-mode spectra from MassBank)
- Series of progressively larger training-subset sizes with a fixed held-out internal test set
- Per-size precision and recall values from retrained BitterMasS Random Forest models

## Outputs

- learning-curve figure (learning_curve.png) plotting precision and recall against training-set size
- Table of precision/recall values per training-set size

## How to apply

Load the combined bitter/nonbitter training corpus (5414 experimental EI-MS and ESI-MS/MS positive-mode spectra from MassBank) and carve out a fixed internal test set. Define a series of progressively larger training-subset sizes by sampling from the remaining data so the test set is never contaminated. For each subset size, retrain the Random Forest classifier in the BitterMasS configuration and record precision and recall on the held-out internal test set (the full model achieves precision = 0.83 and recall = 0.90). Plot the performance metrics against training-set size and save the figure as learning_curve.png. Validate that the curve exhibits the reported pattern: dramatic improvement with initial increases in training-set size, followed by only moderate gains thereafter.

## Related tools

- **BitterMasS** (Random Forest classifier retrained at each training-subset size to produce the precision/recall points of the learning curve)

## Examples

```
import matplotlib.pyplot as plt; plt.plot(train_sizes, precision, label='precision'); plt.plot(train_sizes, recall, label='recall'); plt.xlabel('training-set size'); plt.savefig('learning_curve.png')
```

## Evaluation signals

- The learning-curve figure is saved as learning_curve.png
- Performance metrics are evaluated on a fixed internal test set that is held out identically across all subset sizes
- The curve reproduces the reported shape of Figure S6: dramatic early gains with initial increases in training-set size, then only moderate gains
- At the full training size, metrics approach the reported internal-test-set values of precision = 0.83 and recall = 0.90
- Each point corresponds to a freshly retrained Random Forest, not a partially trained single model

## Limitations

- Only a small fraction of training compounds had spectra available (about 30% of the initial bitter and nonbitter sets), so the curve is limited by spectral data scarcity
- Nonbitter compound spectral representation in the training set is insufficient, which the authors note could bias performance at larger training sizes
- The authors do not provide code, version, or hyperparameters for BitterMasS in the article, so the curve may not be exactly reproducible

## Evidence

- [results] performance improves dramatically with initial increases in training set size, followed by only moderate gains thereafter: [evidence span withheld — anchor status: not_found]
- [methods] retrain the Random Forest classifier (BitterMasS configuration) and evaluate precision and recall on the held-out internal test set: [evidence span withheld — anchor status: not_found]
- [methods] Plot the prediction performance metrics against training-set size and save the resulting learning-curve figure as learning_curve.png.: [evidence span withheld — anchor status: not_found]
- [abstract] We present BitterMasS, a Random Forest classifier that was trained on 5414 experimental mass spectra: "We present BitterMasS, a Random Forest classifier that was trained on 5414 experimental mass spectra"
