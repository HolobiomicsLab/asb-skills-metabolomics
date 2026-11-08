---
name: random-forest-classifier-training
description: Use when you have experimental EI-MS or ESI-MS/MS spectra of labeled
  bitter/non-bitter compounds and need to train, select, and evaluate a Random Forest
  against XGBoost and a naive baseline.
when_to_use_negative:
- The chemical structures of the query compounds are known — use the structure-based
  BitterPredict classifier instead of spectrum-based prediction.
- Compounds have no experimental mass spectrum available (only ~30% of training compounds
  had at least one MassBank spectrum, so coverage is a prerequisite).
- You only need to evaluate an already-trained model on precomputed descriptors rather
  than train from binned spectra.
edam_operation: http://edamontology.org/operation_2454
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_3474
tools:
- name: BitterMasS
  role: Random Forest classifier trained on 5414 experimental mass spectra; the reference
    implementation and final prediction tool
  repo: Niv-Lab/BitterPredict1
- name: MassBank
  role: Source of experimental EI-MS and ESI-MS/MS positive-ion-mode spectra used
    to build the 0.1 m/z binned training descriptors
- name: XGBoost
  role: Comparator algorithm in the six-model dataset/algorithm grid-search selection;
    loses to Random Forest by ~0.01 balanced accuracy
- name: BitterPredict
  role: Structure-based bitterness classifier used as the alternative spectrum-structure-bitterness
    strategy benchmark
  repo: Niv-Lab/BitterPredict1
provenance:
  source_task_ids:
  - task_001
  - task_002
  source_papers:
  - doi: 10.1021/acs.jafc.3c09767
    title: 'BitterMasS: Predicting Bitterness from Mass Spectra'
  source_content_hash: sha256:3e378bc520957d96da835345dc79278cfc873a4167edacbcad6b83ea6beeb54f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Niv-Lab/BitterPredict1
  tools:
  - BitterMasS
  - BitterPredict
  - MassBank
  - XGBoost
  edam_operation: http://edamontology.org/operation_2454
  edam_topics:
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8461538461538461
n_claims: 13
---

# random-forest-classifier-training

## Summary

Trains a Random Forest classifier (BitterMasS) to predict bitterness directly from experimental mass spectra, using fixed-length 0.1 m/z binned spectral descriptors of EI-MS and ESI-MS/MS data and compound-level cross-validated splitting.

## When to use

Use when you have experimental EI-MS or ESI-MS/MS spectra of labeled bitter/non-bitter compounds and need to train, select, and evaluate a Random Forest against XGBoost and a naive baseline.

## When NOT to use

- The chemical structures of the query compounds are known — use the structure-based BitterPredict classifier instead of spectrum-based prediction.
- Compounds have no experimental mass spectrum available (only ~30% of training compounds had at least one MassBank spectrum, so coverage is a prerequisite).
- You only need to evaluate an already-trained model on precomputed descriptors rather than train from binned spectra.

## Inputs

- Experimental EI-MS spectra from MassBank (positive ion mode) for bitter and non-bitter compounds
- Experimental ESI-MS/MS positive-ion-mode spectra from MassBank for the same compound set
- Compound-level bitter/non-bitter labels (e.g., from BitterDB-derived sets)

## Outputs

- Trained BitterMasS Random Forest classifier on 0.1 m/z binned spectral descriptors
- balanced_accuracy_comparison.csv with balanced-accuracy values for 6 models (3 RF, 3 XGBoost across EI-only, ESI-only, combined sets)
- Aggregated internal-test precision, recall, and F1 over 20 train/test repetitions

## How to apply

Retrieve experimental EI-MS and ESI-MS/MS positive-ion-mode spectra from MassBank for the labeled compound set, then convert each spectrum into a fixed-length descriptor by binning peak intensities into 0.1 m/z bins and appending instrument-type descriptors. Partition compounds 80/20 at the compound level (all spectra of one compound stay in the same fold) and repeat the split 20 times. For algorithm/dataset selection, define hyperparameter grids for Random Forest and XGBoost and run fivefold cross-validated grid search on the EI-only, ESI-only, and combined feature matrices, selecting the best configuration per pair by balanced accuracy; the article chose Random Forest for its ~0.01 balanced-accuracy edge over XGBoost and the combined dataset over the individual sets. Train the selected RF per repetition, predict held-out test compounds, and aggregate precision, recall, and F1 over the 20 repetitions against the proportion-based naive baseline (precision/recall 0.67, F1 0.67, BA 0.51).

## Related tools

- **BitterMasS** (Random Forest classifier trained on 5414 experimental mass spectra; the reference implementation and final prediction tool) — Niv-Lab/BitterPredict1
- **MassBank** (Source of experimental EI-MS and ESI-MS/MS positive-ion-mode spectra used to build the 0.1 m/z binned training descriptors)
- **XGBoost** (Comparator algorithm in the six-model dataset/algorithm grid-search selection; loses to Random Forest by ~0.01 balanced accuracy)
- **BitterPredict** (Structure-based bitterness classifier used as the alternative spectrum-structure-bitterness strategy benchmark) — Niv-Lab/BitterPredict1

## Evaluation signals

- Internal-test precision = 0.83 and recall = 0.90 aggregated over the 20 compound-level 80/20 repetitions
- Model precision/recall/F1 clearly exceed the naive proportion baseline (precision/recall 0.67, F1 0.67, BA 0.51)
- Combined EI-MS + ESI-MS/MS balanced accuracy exceeds both individual-set results for each algorithm
- Random Forest balanced-accuracy advantage over XGBoost is approximately 0.01
- All spectra of a single compound remain in the same fold in every split (no compound leakage)

## Limitations

- Predictions are limited to EI-MS or ESI-MS/MS spectra obtained in positive ion mode.
- Only ~30% of the initial bitter and non-bitter compounds had at least one identified mass spectrum, limiting training data; non-bitter spectral representation is insufficient.
- No software/code availability, hosting location, or version number is provided for BitterMasS in the article.
- Prediction on the individual EI-MS and ESI-MS/MS sets showed poor results compared with the combined set, so single-instrument-type models should not be expected to perform well.

## Evidence

- [abstract] BitterMasS RF training and internal-test metrics: "We present BitterMasS, a Random Forest classifier that was trained on 5414 experimental mass spectra"
- [abstract] Internal-test performance target: "achieving precision = 0.83 and recall = 0.90 for an internal test set"
- [discussion] Combined dataset outperforms individual sets: "Prediction on the individual EI-MS and ESI-MS/MS sets showed poor results compared with the combined set."
- [discussion] Training data coverage limitation: "only 30% of the compounds in the initial bitter and nonbitter sets in the training set had at least one mass spectrum identified"
- [readme] BitterPredict scope (structure-based alternative): "BitterPredict is a classifier which predicts whether a compound is bitter or not, based on its chemical structure."
