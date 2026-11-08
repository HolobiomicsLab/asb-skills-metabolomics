---
name: random-forest-learning-curve-analysis
description: Use when you must show how a spectral classifier's precision and recall
  change as the training set is grown from small to full size, e.g. to justify a data-collection
  effort or reproduce a published learning curve.
when_to_use_negative:
- The input is a chemical structure rather than a spectrum — BitterPredict (structure-based
  classifier) is the appropriate tool, not spectrum-based retraining.
- 'Only individual EI-MS or individual ESI-MS/MS sets are available: the article reports
  that prediction on the individual sets showed poor results compared with the combined
  set.'
- You need a final production predictor rather than an assessment of training-data
  sufficiency — use the fully trained BitterMasS model directly.
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3473
tools:
- name: BitterMasS
  role: Random Forest classifier retrained from scratch on each progressively larger
    spectral training subset, then evaluated for precision and recall on a fixed internal
    test set
  repo: https://github.com/Niv-Lab/BitterPredict1
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1021/acs.jafc.3c09767
    title: 'BitterMasS: Predicting Bitterness from Mass Spectra'
  source_content_hash: sha256:813de4b5d4e28caefc4b969066925741458ea2844506c6625c2130a6e67768bc
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Niv-Lab/BitterPredict1
  tools:
  - BitterMasS
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3473
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# random-forest-learning-curve-analysis

## Summary

Retrains a Random Forest classifier (BitterMasS configuration) on progressively larger subsets of a combined spectral training corpus and plots precision/recall against training-set size to characterize the data-efficiency of the model.

## When to use

Use when you must show how a spectral classifier's precision and recall change as the training set is grown from small to full size, e.g. to justify a data-collection effort or reproduce a published learning curve.

## When NOT to use

- The input is a chemical structure rather than a spectrum — BitterPredict (structure-based classifier) is the appropriate tool, not spectrum-based retraining.
- Only individual EI-MS or individual ESI-MS/MS sets are available: the article reports that prediction on the individual sets showed poor results compared with the combined set.
- You need a final production predictor rather than an assessment of training-data sufficiency — use the fully trained BitterMasS model directly.

## Inputs

- Combined EI-MS and ESI-MS/MS positive-mode spectral training dataset with bitter/nonbitter labels (5414 MassBank spectra for BitterMasS)
- Series of progressively larger training-subset sizes
- Fixed internal test set held out from all subset sizes

## Outputs

- Precision and recall values for each training-set size on the held-out internal test set
- Learning-curve figure (learning_curve.png) plotting performance metrics against training-set size

## How to apply

Load the combined bitter/nonbitter spectral training set — for BitterMasS this is 5414 experimental EI-MS and ESI-MS/MS positive-ion-mode spectra extracted from MassBank. Split off a fixed internal test set that is held out identically for every training size. Define a series of progressively larger training subsets (e.g. increasing fractions of the combined set) and, for each size, retrain the Random Forest classifier from scratch rather than partially fitting. Evaluate precision and recall on the held-out internal test set for every subset size and plot the metrics versus training-set size, saving the figure (e.g. learning_curve.png). Validate against the published pattern: performance improves dramatically with initial increases in training-set size and then only moderately (BitterMasS reached precision = 0.83 and recall = 0.90 at full size, consistent with Figure S6).

## Related tools

- **BitterMasS** (Random Forest classifier retrained from scratch on each progressively larger spectral training subset, then evaluated for precision and recall on a fixed internal test set) — https://github.com/Niv-Lab/BitterPredict1

## Examples

```
from sklearn.ensemble import RandomForestClassifier; rf = RandomForestClassifier().fit(X_sub, y_sub); scores.append((len(y_sub), precision_score(y_test, rf.predict(X_test)), recall_score(y_test, rf.predict(X_test))))
```

## Evaluation signals

- The reproduced curve shows dramatic improvement with initial increases in training-set size followed by only moderate gains, matching Figure S6.
- At the full training size, precision and recall on the internal test set are close to the reported precision = 0.83 and recall = 0.90.
- The same internal test set is used unchanged across every subset size, so metrics are comparable along the curve.
- The curve is generated from the combined EI-MS + ESI-MS/MS positive-mode dataset, not from individual spectral types.
- The learning-curve figure is produced and saved (learning_curve.png) with performance on the y-axis and training-set size on the x-axis.

## Limitations

- The article provides no model retraining details, hyperparameters, or code availability for BitterMasS, so exact reproduction requires assumptions about Random Forest configuration.
- Only EI-MS and ESI-MS/MS spectra obtained in positive ion mode are covered by the current implementation.
- Spectral data existed for only about 30% of the initial bitter and nonbitter training compounds, and nonbitter spectral representation is insufficient — the curve may reflect label-imbalance effects at small subset sizes.

## Evidence

- [abstract] BitterMasS Random Forest on 5414 MassBank spectra: "We present BitterMasS, a Random Forest classifier that was trained on 5414 experimental mass spectra"
- [abstract] Internal test set performance benchmark: "achieving precision = 0.83 and recall = 0.90 for an internal test set"
- [discussion] Spectral training data source: "Experimental EI-MS and ESI-MS/MS positive ion mode mass spectra for these compounds were extracted from the MassBank database"
- [discussion] Combined set outperforms individual spectral types: "Prediction on the individual EI-MS and ESI-MS/MS sets showed poor results compared with the combined set."
