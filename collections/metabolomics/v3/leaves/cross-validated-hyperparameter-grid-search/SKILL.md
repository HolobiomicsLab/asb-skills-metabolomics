---
name: cross-validated-hyperparameter-grid-search
description: Use when you must choose both a classifier (e.g., Random Forest vs XGBoost)
  and a feature-set variant (e.g., EI-only vs ESI-only vs combined binned mass spectra)
  and need fair, cross-validated hyperparameter tuning plus a single held-out metric
  for the decision.
when_to_use_negative:
- The goal is bitterness prediction on new spectra rather than algorithm/dataset selection
  — use the already-selected BitterMasS Random Forest classifier instead of rerunning
  the comparison
- Input is chemical structure descriptors rather than mass spectra — the structure-based
  path (BitterPredict) applies, not this spectral model-selection workflow
- Only one algorithm and one feature set are candidates, so there is no six-way grid-search
  comparison to perform
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_0092
tools:
- name: BitterMasS
  role: Reference Random Forest pipeline whose training/evaluation setup the six-model
    comparison reproduces; the chosen outcome is the BitterMasS Random Forest classifier
    on the combined EI-MS + ESI-MS/MS dataset
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1021/acs.jafc.3c09767
    title: 'BitterMasS: Predicting Bitterness from Mass Spectra'
  source_content_hash: sha256:f7caa1c081e91857a2f4c6541c37a501035da9375d63de353917c60041688f79
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BitterMasS
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# cross-validated-hyperparameter-grid-search

## Summary

A systematic model-selection procedure in which Random Forest and XGBoost classifiers, each tuned by fivefold cross-validated grid search, are trained on three binned mass-spectral feature sets (EI-MS, ESI-MS/MS, and the combined set) and compared by balanced accuracy — the approach used to select the algorithm and dataset behind BitterMasS.

## When to use

Use when you must choose both a classifier (e.g., Random Forest vs XGBoost) and a feature-set variant (e.g., EI-only vs ESI-only vs combined binned mass spectra) and need fair, cross-validated hyperparameter tuning plus a single held-out metric for the decision.

## When NOT to use

- The goal is bitterness prediction on new spectra rather than algorithm/dataset selection — use the already-selected BitterMasS Random Forest classifier instead of rerunning the comparison
- Input is chemical structure descriptors rather than mass spectra — the structure-based path (BitterPredict) applies, not this spectral model-selection workflow
- Only one algorithm and one feature set are candidates, so there is no six-way grid-search comparison to perform

## Inputs

- Binned mass-spectral feature matrices on a shared 0.1 m/z grid: EI-MS only, ESI-MS/MS only, and combined EI-MS + ESI-MS/MS (bitter vs nonbitter labels)
- Train/test splits for each of the three datasets
- Hyperparameter grids for Random Forest and XGBoost

## Outputs

- Best hyperparameter configuration per algorithm–dataset pair (6 pairs from 5-fold CV grid search)
- Six trained models (3 Random Forest, 3 XGBoost) evaluated on held-out test splits
- balanced_accuracy_comparison.csv tabulating balanced accuracy for all six model–dataset combinations

## How to apply

First bin the peak intensities of each spectrum (EI-MS and ESI-MS/MS positive ion mode) onto a shared 0.1 m/z grid, producing three feature matrices: EI-only, ESI-only, and combined. Define hyperparameter grids for both Random Forest and XGBoost and run a fivefold cross-validated grid search on each of the three datasets, yielding a best configuration for each of the six algorithm–dataset pairs. Train the six best-configured models on the training splits (the BitterMasS Random Forest pipeline serves as the reference implementation) and evaluate each on its held-out test split using balanced accuracy as the sole comparison metric. Compile the six balanced-accuracy values into balanced_accuracy_comparison.csv and apply the validation rule: the combined-set model must outperform both individual-set models, and the Random Forest advantage over XGBoost should be roughly 0.01 — the reported basis for choosing Random Forest on the combined EI-MS + ESI-MS/MS data.

## Related tools

- **BitterMasS** (Reference Random Forest pipeline whose training/evaluation setup the six-model comparison reproduces; the chosen outcome is the BitterMasS Random Forest classifier on the combined EI-MS + ESI-MS/MS dataset)

## Examples

```
from sklearn.model_selection import GridSearchCV; gs = GridSearchCV(RandomForestClassifier(random_state=0), param_grid, cv=5, scoring='balanced_accuracy'); gs.fit(X_binned, y)  # run per RF/XGBoost x {EI, ESI, combined}
```

## Evaluation signals

- balanced_accuracy_comparison.csv exists and contains balanced-accuracy values for all six algorithm–dataset pairs
- The combined-set balanced accuracy exceeds both the individual EI-MS and ESI-MS/MS results, matching the reported finding that individual-set prediction was poor compared with the combined set
- The Random Forest advantage over XGBoost is approximately 0.01, consistent with the article's selection rationale
- Each best configuration comes from a fivefold cross-validated grid search, not from default hyperparameters
- The selected model's internal test performance is in the range of BitterMasS's reported precision = 0.83 and recall = 0.90

## Limitations

- Mass spectral data were available for only ~30% of the initial bitter and nonbitter training compounds, limiting the feature matrices the grid search can draw on
- Nonbitter compound spectral representation in the training set is acknowledged as insufficient, so the comparison metric may understate achievable performance
- No hyperparameter values, model versions, or code-hosting details are published for BitterMasS in the article, so the grid definitions must be reconstructed

## Evidence

- [methods] RF chosen over XGBoost and combined dataset over separate sets: "Prediction on the individual EI-MS and ESI-MS/MS sets showed poor results compared with the combined set."
- [abstract] BitterMasS is a Random Forest classifier trained on experimental mass spectra: "We present BitterMasS, a Random Forest classifier that was trained on 5414 experimental mass spectra"
- [full_text] Task card workflow defining the 0.1 m/z binning and fivefold CV grid search: [evidence span withheld — anchor status: not_found]
