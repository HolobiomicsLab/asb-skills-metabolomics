---
name: balanced-accuracy-model-evaluation
description: 'Decision support for choosing among multiple classifier–dataset combinations:
  assemble binned feature matrices for each spectral dataset, run fivefold CV grid
  search per pair, and rank by balanced accuracy on held-out test splits.'
when_to_use_negative:
- When the goal is structure-based bitterness prediction (BitterPredict from chemical
  structure) rather than spectrum-based prediction and model selection.
- When only a single dataset and a single algorithm are available — the comparison
  logic requires at least two candidate algorithm–dataset pairs.
- When the spectral input is already annotated to structures; the paper's direct spectrum–bitterness
  strategy is recommended when 'the chemical structure of a compound is not yet unknown'.
edam_operation: http://edamontology.org/operation_3925
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: BitterMasS
  role: Reference Random Forest pipeline whose training/evaluation loop is reproduced
    when comparing the six candidate models
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1021/acs.jafc.3c09767
    title: 'BitterMasS: Predicting Bitterness from Mass Spectra'
  source_content_hash: sha256:3feab17f5d9ce3f708658b191b70c088135c88db395e5a10ddc30b95cdd94a8e
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BitterMasS
  edam_operation: http://edamontology.org/operation_3925
  edam_topics:
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# balanced-accuracy-model-evaluation

## Summary

A systematic model-selection procedure that trains candidate classifier–dataset pairs (Random Forest vs XGBoost on EI-MS, ESI-MS/MS, and combined spectral sets) under fivefold cross-validated grid search and compares them by held-out balanced accuracy. It grounds the algorithm and dataset choice (Random Forest, combined set) in reproducible numeric evidence.

## When to use

Decision support for choosing among multiple classifier–dataset combinations: assemble binned feature matrices for each spectral dataset, run fivefold CV grid search per pair, and rank by balanced accuracy on held-out test splits.

## When NOT to use

- When the goal is structure-based bitterness prediction (BitterPredict from chemical structure) rather than spectrum-based prediction and model selection.
- When only a single dataset and a single algorithm are available — the comparison logic requires at least two candidate algorithm–dataset pairs.
- When the spectral input is already annotated to structures; the paper's direct spectrum–bitterness strategy is recommended when 'the chemical structure of a compound is not yet unknown'.

## Inputs

- Peak-intensity feature matrices binned onto a shared 0.1 m/z grid (EI-MS, ESI-MS/MS, and combined sets)
- Training and held-out test splits for each of the three datasets
- Hyperparameter grids for Random Forest and XGBoost

## Outputs

- balanced_accuracy_comparison.csv (balanced accuracy per model)
- Six best-configured trained models (three Random Forest, three XGBoost)
- Selected algorithm and dataset (Random Forest on the combined EI-MS + ESI-MS/MS set)

## How to apply

Binning the EI-MS and ESI-MS/MS peak intensities onto a shared 0.1 m/z grid produces three candidate feature matrices (EI-only, ESI-only, combined). For each dataset, define hyperparameter grids for Random Forest and XGBoost and run fivefold cross-validated grid search to select the best configuration per algorithm–dataset pair, then train the six best-configured models on the training splits. Evaluate each model on its held-out test split, recording balanced accuracy (the metric the paper treats as primary) and compiling the values into balanced_accuracy_comparison.csv. The decision rule used in BitterMasS: the combined EI-MS + ESI-MS/MS set is selected if it beats both individual sets, and Random Forest is selected if its balanced-accuracy advantage over XGBoost is real (here, roughly 0.01). Use the BitterMasS Random Forest pipeline as the reference implementation so results are reproducible.

## Related tools

- **BitterMasS** (Reference Random Forest pipeline whose training/evaluation loop is reproduced when comparing the six candidate models)

## Evaluation signals

- The combined-set balanced accuracy must exceed the balanced accuracy of both individual EI-MS and ESI-MS/MS models.
- The Random Forest balanced-accuracy advantage over XGBoost should be approximately 0.01, matching the reported finding.
- balanced_accuracy_comparison.csv contains exactly six rows — three Random Forest and three XGBoost — covering the EI-MS, ESI-MS/MS, and combined sets.
- Every reported balanced accuracy comes from a held-out test split, not the fivefold cross-validation splits used for hyperparameter selection.
- Individual-set predictions are expected to be poor relative to the combined set, consistent with the paper's discussion.

## Limitations

- Mass spectral data were available for only ~30% of the initial bitter and nonbitter training compounds, and nonbitter compound spectral representation was insufficient, which limits the models being compared.
- No hyperparameters, version numbers, or code-hosting location for BitterMasS are stated, so exact reproduction of the grid search depends on the reference pipeline.
- BitterMasS in its current implementation predicts only from EI-MS or ESI-MS/MS spectra obtained in positive ion mode.

## Evidence

- [discussion] Combined-set superiority: "Prediction on the individual EI-MS and ESI-MS/MS sets showed poor results compared with the combined set."
- [abstract] Random Forest as the reference classifier: "We present BitterMasS, a Random Forest classifier that was trained on 5414 experimental mass spectra"
- [readme] Model basis of the compared tools: "BitterPredict is a classifier which predicts whether a compound is bitter or not, based on its chemical structure."
- [discussion] Scope of spectra supported: "the model makes predictions based on EI-MS or ESI-MS/MS spectra obtained in positive ion mode."
