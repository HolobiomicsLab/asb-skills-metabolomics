---
name: naive-baseline-classifier-comparison
description: Use when a binary classifier's precision/recall must be shown to exceed
  what the class proportions alone would achieve, e.g. reporting an internal test
  set for a spectral bitterness classifier.
when_to_use_negative:
- When the chemical structure is known — the article recommends BitterPredict (structure-based)
  rather than spectrum-based prediction; the spectrum–bitterness route is recommended
  only when 'the chemical structure of a compound is not yet unknown'.
- When spectra must be evaluated per instrument type — prediction on the individual
  EI-MS and ESI-MS/MS sets showed poor results compared with the combined set.
- When compounds lack any retrieved spectrum — only 30% of the initial bitter and
  nonbitter training compounds had at least one mass spectrum identified.
edam_topics:
- http://edamontology.org/topic_3318
tools:
- name: BitterMasS
  role: Random Forest classifier whose internal test precision/recall is benchmarked
    against the naive proportion-based baseline
- name: MassBank
  role: Source of the experimental EI-MS and ESI-MS/MS positive ion mode spectra used
    to build the training/test descriptors
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/acs.jafc.3c09767
    title: 'BitterMasS: Predicting Bitterness from Mass Spectra'
  source_content_hash: sha256:106a939d18638a49916e1495a151d1e56c7a5ad50aca8e94c26aef4e36cd4989
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BitterMasS
  - MassBank
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3318
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# naive-baseline-classifier-comparison

## Summary

Benchmark a supervised classifier's precision/recall against a naive proportion-based (majority-class) baseline computed from the same splits, as done when validating BitterMasS RF bitterness prediction on EI-MS + ESI-MS/MS MassBank descriptors.

## When to use

Use when a binary classifier's precision/recall must be shown to exceed what the class proportions alone would achieve, e.g. reporting an internal test set for a spectral bitterness classifier.

## When NOT to use

- When the chemical structure is known — the article recommends BitterPredict (structure-based) rather than spectrum-based prediction; the spectrum–bitterness route is recommended only when 'the chemical structure of a compound is not yet unknown'.
- When spectra must be evaluated per instrument type — prediction on the individual EI-MS and ESI-MS/MS sets showed poor results compared with the combined set.
- When compounds lack any retrieved spectrum — only 30% of the initial bitter and nonbitter training compounds had at least one mass spectrum identified.

## Inputs

- Fixed-length spectral descriptor matrix: peak intensities binned into 0.1 m/z bins plus instrument-type descriptors
- Bitter/non-bitter compound labels with compound-level fold assignments (80/20 split, 20 repetitions)
- Experimental EI-MS and ESI-MS/MS positive ion mode spectra retrieved from MassBank

## Outputs

- Aggregated model precision, recall, and F1 over 20 internal test repetitions (target: precision = 0.83, recall = 0.90)
- Naive proportion-based baseline metrics: baseline precision/recall = 0.67, F1 = 0.67, balanced accuracy = 0.51
- Per-repetition internal test predictions for held-out compounds

## How to apply

First define the evaluation splits exactly as the model sees them — here, compounds partitioned 80% train / 20% internal test at the compound level, repeated 20 times so all spectra of one compound stay in the same fold. Train the model (BitterMasS Random Forest on 0.1 m/z binned peak-intensity descriptors plus instrument-type descriptors, built from MassBank EI-MS and ESI-MS/MS positive ion mode spectra) on each training partition, and aggregate precision, recall, and F1 for held-out compounds across the 20 repetitions. Then compute the naive proportion-based baseline — predicting the majority/proportion class — on the identical folds, which in this case yields baseline precision/recall = 0.67, F1 = 0.67, and balanced accuracy = 0.51. Accept the model only if its aggregated metrics clearly exceed the baseline: BitterMasS's internal test precision = 0.83 and recall = 0.90 vs. the 0.67 baseline is the passing example. Run the baseline on the same combined EI-MS + ESI-MS/MS dataset used for the model, since prediction on the individual EI-MS and ESI-MS/MS sets showed poor results compared with the combined set.

## Related tools

- **BitterMasS** (Random Forest classifier whose internal test precision/recall is benchmarked against the naive proportion-based baseline)
- **MassBank** (Source of the experimental EI-MS and ESI-MS/MS positive ion mode spectra used to build the training/test descriptors)

## Evaluation signals

- Internal test precision = 0.83 and recall = 0.90, clearly exceeding the naive baseline precision/recall = 0.67 and F1 = 0.67.
- Baseline balanced accuracy = 0.51, confirming the naive predictor is near chance on this class distribution.
- All spectra of one compound remain in the same fold across the 20 repetitions of the 80/20 compound-level split.
- Model metrics are aggregated over the same 20 repetitions on the combined EI-MS + ESI-MS/MS dataset, not on individual instrument-type subsets.

## Limitations

- Only 30% of compounds in the initial bitter and nonbitter sets had at least one mass spectrum identified, limiting training data and thus the achievable baseline contrast.
- Nonbitter compound spectral representation in the training set is insufficient, which can inflate apparent recall of the bitter class over the baseline.
- No code or hosting location for the model itself was stated (the related BitterPredict repo notes 'full code will be available upon publication'), so the baseline comparison may not be independently reproducible without author input.

## Evidence

- [abstract] BitterMasS internal test metrics: "We present BitterMasS, a Random Forest classifier that was trained on 5414 experimental mass spectra"
- [abstract] BitterMasS internal test precision and recall: "achieving precision = 0.83 and recall = 0.90 for an internal test set"
- [discussion] Combined-set requirement for valid comparison: "Prediction on the individual EI-MS and ESI-MS/MS sets showed poor results compared with the combined set."
- [discussion] Recommendation of spectrum-based prediction only when structure is unknown: "the direct prediction of bitterness from the mass spectrum is recommended when the chemical structure of a compound is not yet unknown"
- [readme] Code availability caveat: "full code will be available upon publication."
