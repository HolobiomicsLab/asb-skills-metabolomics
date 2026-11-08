---
name: training-subset-sampling
description: Use when quantifying how a classifier's precision/recall scales with
  training-set size — e.g. testing whether adding more MassBank spectra to the bitter/nonbitter
  training set yields diminishing returns or still pays off.
when_to_use_negative:
- The goal is predicting bitterness for a new spectrum rather than studying how training-data
  volume affects model performance
- The analysis question is hyperparameter tuning or model selection rather than training-set-size
  scaling
- Only a single fixed training corpus is available with no ability to draw progressively
  larger subsets while holding out a test set
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_0091
tools:
- name: BitterMasS
  role: Random Forest classifier whose configuration is retrained on each training
    subset and evaluated for precision/recall on the fixed internal test set
  repo: https://github.com/Niv-Lab/BitterPredict1
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1021/acs.jafc.3c09767
    title: 'BitterMasS: Predicting Bitterness from Mass Spectra'
  source_content_hash: sha256:6a35eb5eb545077b48c1519ba149b9998db134f0bdc73831c95876c9b8d41de3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Niv-Lab/BitterPredict1
  tools:
  - BitterMasS
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# training-subset-sampling

## Summary

Generates a learning curve for the BitterMasS Random Forest bitterness classifier by repeatedly retraining it on progressively larger subsets of the combined EI-MS/ESI-MS/MS spectral training set, to characterize how prediction performance scales with data volume.

## When to use

Use when quantifying how a classifier's precision/recall scales with training-set size — e.g. testing whether adding more MassBank spectra to the bitter/nonbitter training set yields diminishing returns or still pays off.

## When NOT to use

- The goal is predicting bitterness for a new spectrum rather than studying how training-data volume affects model performance
- The analysis question is hyperparameter tuning or model selection rather than training-set-size scaling
- Only a single fixed training corpus is available with no ability to draw progressively larger subsets while holding out a test set

## Inputs

- Combined bitter/nonbitter mass spectral training dataset (5414 experimental EI-MS and ESI-MS/MS positive-mode spectra from MassBank)
- Bitter/nonbitter labels for each spectrum
- List of progressively larger training-subset sizes to sample
- Fixed internal test set held out from all training subsets

## Outputs

- Per-subset-size precision and recall values on the held-out internal test set
- Learning-curve figure of performance vs. training-set size (learning_curve.png)
- Qualitative characterization of the curve shape (steep initial rise vs. plateau)

## How to apply

Start from the combined training dataset of 5414 experimental EI-MS and ESI-MS/MS positive-mode spectra (bitter and nonbitter) extracted from MassBank, and carve out a fixed internal test set that is held constant across all runs. Define a series of progressively larger training-subset sizes by sampling from the remaining spectra, keeping the bitter/nonbitter label balance as consistent as possible across sizes. For each subset size, retrain the Random Forest classifier in the BitterMasS configuration and evaluate precision and recall on the same held-out internal test set — the fixed test set is what makes performance comparable across sizes. Plot the performance metrics against training-set size and save the figure (learning_curve.png). Validate by confirming the reproduced curve shows the reported pattern: dramatic improvement with initial increases in training-set size followed by only moderate gains, consistent with Figure S6.

## Related tools

- **BitterMasS** (Random Forest classifier whose configuration is retrained on each training subset and evaluated for precision/recall on the fixed internal test set) — https://github.com/Niv-Lab/BitterPredict1

## Examples

```
from sklearn.ensemble import RandomForestClassifier; rf = RandomForestClassifier().fit(X_subset, y_subset); p, r = evaluate(rf, X_test, y_test)
```

## Evaluation signals

- Performance metrics (precision, recall) improve monotonically or near-monotonically as the training-subset size increases
- The reproduced curve shows the reported shape: dramatic improvement at initial size increases, then only moderate gains — matching Figure S6
- At the largest training size (full combined set), internal-test-set performance approaches the reported precision = 0.83 and recall = 0.90
- The same fixed internal test set is used for every subset size, so metric differences across sizes are attributable to training data volume
- The largest-size model trained on the combined EI-MS/ESI-MS/MS set outperforms models trained on either spectral mode individually

## Limitations

- Only about 30% of the compounds in the initial bitter and nonbitter sets had at least one identified mass spectrum, limiting the training data available for scaling experiments
- Nonbitter compound spectral representation in the training set is insufficient, so curve gains may be understated for the nonbitter class
- The combined EI-MS/ESI-MS/MS set is required — predictions on the individual EI-MS or ESI-MS/MS sets showed poor results, so subset sampling within a single mode is not representative

## Evidence

- [abstract] Random Forest trained on 5414 spectra: "We present BitterMasS, a Random Forest classifier that was trained on 5414 experimental mass spectra"
- [abstract] internal test set metrics: "achieving precision = 0.83 and recall = 0.90 for an internal test set"
- [discussion] combined set outperforms individual sets: "Prediction on the individual EI-MS and ESI-MS/MS sets showed poor results compared with the combined set."
- [discussion] limited spectral coverage of training compounds: "only 30% of the compounds in the initial bitter and nonbitter sets in the training set had at least one mass spectrum identified"
