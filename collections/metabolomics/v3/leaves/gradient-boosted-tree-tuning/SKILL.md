---
name: gradient-boosted-tree-tuning
description: Use when several candidate classifiers (e.g. Random Forest vs XGBoost)
  and several feature-matrix variants (e.g. EI-only, ESI-only, combined 0.1 m/z-binned
  spectra) must be compared to justify one production model.
when_to_use_negative:
- The goal is predicting bitterness from chemical structure rather than from mass
  spectra — BitterPredict (structure-based classifier) is the appropriate tool in
  that case
- Only a single algorithm or a single feature-matrix variant is of interest, so no
  comparative selection between Random Forest, XGBoost, and EI/ESI/combined datasets
  is needed
- Input spectra are not EI-MS or positive-ion-mode ESI-MS/MS, which are the only modes
  the current BitterMasS implementation supports
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_0092
tools:
- name: BitterMasS
  role: Reference Random Forest pipeline whose trained models serve as the benchmark
    against which XGBoost configurations are compared during algorithm and dataset
    selection
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1021/acs.jafc.3c09767
    title: 'BitterMasS: Predicting Bitterness from Mass Spectra'
  source_content_hash: sha256:dc13cb034082627239dc6813eb2d89bd5e4af54fd8d01051693b50162a76ddf4
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BitterMasS
  edam_operation: http://edamontology.org/operation_2990
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# gradient-boosted-tree-tuning

## Summary

Systematic algorithm-and-dataset selection for spectral classification: train Random Forest and XGBoost over multiple binned feature-matrix variants (EI-MS, ESI-MS/MS, combined) with fivefold cross-validated grid search, then pick the winner by held-out balanced accuracy.

## When to use

Use when several candidate classifiers (e.g. Random Forest vs XGBoost) and several feature-matrix variants (e.g. EI-only, ESI-only, combined 0.1 m/z-binned spectra) must be compared to justify one production model.

## When NOT to use

- The goal is predicting bitterness from chemical structure rather than from mass spectra — BitterPredict (structure-based classifier) is the appropriate tool in that case
- Only a single algorithm or a single feature-matrix variant is of interest, so no comparative selection between Random Forest, XGBoost, and EI/ESI/combined datasets is needed
- Input spectra are not EI-MS or positive-ion-mode ESI-MS/MS, which are the only modes the current BitterMasS implementation supports

## Inputs

- Binned peak-intensity feature matrices for EI-MS, ESI-MS/MS, and the combined set (shared 0.1 m/z grid)
- Bitter/non-bitter class labels for the training compounds (5414 experimental mass spectra)
- Hyperparameter grids for Random Forest and XGBoost
- Train/test splits for each of the three datasets

## Outputs

- Six best-configured trained models (three Random Forest, three XGBoost) over EI-MS, ESI-MS/MS, and combined sets
- balanced_accuracy_comparison.csv with held-out balanced accuracy per algorithm-dataset pair
- Selected production algorithm and dataset combination (Random Forest on the combined EI-MS + ESI-MS/MS set)

## How to apply

First assemble the feature matrices by binning peak intensities of the EI-MS and ESI-MS/MS spectra onto a shared 0.1 m/z grid, yielding three datasets: EI-only, ESI-MS/MS-only, and combined. Define hyperparameter grids for Random Forest and XGBoost and run fivefold cross-validated grid search on each of the three datasets to select the best configuration per algorithm-dataset pair. Train the six best-configured models (three Random Forest, three XGBoost) on the training splits, using the BitterMasS Random Forest pipeline as the reference implementation. Evaluate each model on its held-out test split using balanced accuracy, and compile the results into balanced_accuracy_comparison.csv. Select the combination only if the combined-set models outperform both individual-set models and the chosen algorithm's advantage is consistent (here Random Forest exceeded XGBoost by roughly 0.01 balanced accuracy).

## Related tools

- **BitterMasS** (Reference Random Forest pipeline whose trained models serve as the benchmark against which XGBoost configurations are compared during algorithm and dataset selection)

## Evaluation signals

- The combined-set balanced accuracy must exceed both the individual EI-MS and ESI-MS/MS results, mirroring the reported finding that individual-set prediction was poor
- The Random Forest advantage over XGBoost should be approximately 0.01 balanced accuracy, consistent with its selection on a slight performance edge
- Exactly six best-configured models are produced: three Random Forest and three XGBoost, one per dataset variant
- All balanced-accuracy values are recorded in balanced_accuracy_comparison.csv with one row per algorithm-dataset pair
- Each model is tuned by fivefold cross-validated grid search before final training on the training split

## Limitations

- Only 30% of the compounds in the initial bitter and nonbitter training sets had at least one identified mass spectrum, limiting the spectral training data
- Nonbitter compound spectral representation in the training set is insufficient, so class-imbalance effects may bias balanced-accuracy comparisons
- The article provides no model retraining details, hyperparameters, or explicit algorithm-choice justification in the discussion section, so the tuning grids must be reconstructed independently
- The Random Forest advantage over XGBoost is slight (~0.01), so the choice is not robust to small changes in splits or hyperparameter grids

## Evidence

- [abstract] Random Forest classifier trained on 5414 experimental mass spectra supports the reference pipeline: "We present BitterMasS, a Random Forest classifier that was trained on 5414 experimental mass spectra"
- [discussion] Combined dataset outperforms individual EI-MS and ESI-MS/MS sets — key validation invariant: "Prediction on the individual EI-MS and ESI-MS/MS sets showed poor results compared with the combined set."
- [discussion] Scope limited to EI-MS and positive-mode ESI-MS/MS spectra: "In the current implementation, the model makes predictions based on EI-MS or ESI-MS/MS spectra obtained in positive ion mode."
- [readme] README distinguishes structure-based BitterPredict from the spectral approach: "BitterPredict is a classifier which predicts whether a compound is bitter or not, based on its chemical structure."
