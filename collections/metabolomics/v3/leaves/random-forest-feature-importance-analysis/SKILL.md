---
name: random-forest-feature-importance-analysis
description: Use when a trained random forest (e.g. scikit-learn) combines multiple
  candidate-scoring features and you must determine which feature contributes most
  to ranking performance.
when_to_use_negative:
- You need feature importance for a model other than a tree-ensemble (e.g. linear
  models or neural networks like MS2Deepscore itself) — impurity-based feature_importances_
  is specific to random forests.
- You want model performance benchmarking (e.g. recall@1 vs cosine/modified cosine
  score) rather than attributing performance to individual input features.
- The candidate features have not yet been computed for the query spectra — extract
  importances only from a trained model over existing feature columns.
edam_operation: http://edamontology.org/operation_3936
edam_topics:
- http://edamontology.org/topic_0153
- http://edamontology.org/topic_3520
tools:
- name: MS2Query
  role: Provides the trained random forest re-ranking model whose five features (Spec2Vec
    similarity, precursor m/z values, average MS2Deepscore, average Tanimoto score)
    are analyzed for importance
  repo: https://github.com/iomega/ms2query
- name: MS2Deepscore
  role: Supplies the spectral similarity feature (average MS2Deepscore over 10 similar
    library molecules) expected to have the highest importance
- name: Spec2Vec
  role: Supplies one of the five spectral-similarity input features scored by the
    random forest
- name: scikit-learn
  role: Implements the random forest whose feature_importances_ attribute is extracted
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-023-37446-4
    title: 'MS2Query: reliable and scalable MS2 mass spectra-based analogue search'
  source_content_hash: sha256:e7665958e8c6d881ec56d3935d21411941f16b9069d445f67fb4590b9fc1c7f2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/iomega/ms2query
  tools:
  - MS2Deepscore
  - MS2Query
  - Spec2Vec
  - scikit-learn
  edam_operation: http://edamontology.org/operation_3936
  edam_topics:
  - http://edamontology.org/topic_0153
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# random-forest-feature-importance-analysis

## Summary

Quantify which input features drive a trained random forest's re-ranking performance by extracting scikit-learn's impurity-based feature_importances_ for each feature and ranking them. Applied to MS2Query's five analogue-search features to confirm the average MS2Deepscore over 10 similar library molecules is the dominant contributor.

## When to use

Use when a trained random forest (e.g. scikit-learn) combines multiple candidate-scoring features and you must determine which feature contributes most to ranking performance.

## When NOT to use

- You need feature importance for a model other than a tree-ensemble (e.g. linear models or neural networks like MS2Deepscore itself) — impurity-based feature_importances_ is specific to random forests.
- You want model performance benchmarking (e.g. recall@1 vs cosine/modified cosine score) rather than attributing performance to individual input features.
- The candidate features have not yet been computed for the query spectra — extract importances only from a trained model over existing feature columns.

## Inputs

- Zenodo deposit 10.5281/zenodo.6811540 (raw data, intermediate files and raw benchmarking results containing the five candidate-ranking features)
- Trained MS2Query random forest model files (scikit-learn) shipped alongside the library spectra

## Outputs

- feature_importances.csv: five Gini (impurity-based) importance values sorted in descending order, with a column naming each feature (Spec2Vec similarity, query precursor m/z, precursor m/z difference, average MS2Deepscore, average Tanimoto score)

## How to apply

Download the raw data, intermediate files and raw benchmarking results from Zenodo deposit 10.5281/zenodo.6811540, which contain MS2Query's five candidate-ranking features: Spec2Vec similarity, query precursor m/z, precursor m/z difference, the weighted average MS2Deepscore over 10 similar library molecules, and the average Tanimoto score. Load the trained random forest model (implemented with scikit-learn) shipped alongside the library spectra in the MS2Query model files, then extract the impurity-based feature_importances_ attribute for the five features. Write the five importance values sorted in descending order to feature_importances.csv, with a column naming each feature. Validate that the average MS2Deepscore feature has the highest Gini importance of the five and that all five features carry nonzero importance, consistent with the feature-importance analysis in Supplementary Table 1 and 2.

## Related tools

- **MS2Query** (Provides the trained random forest re-ranking model whose five features (Spec2Vec similarity, precursor m/z values, average MS2Deepscore, average Tanimoto score) are analyzed for importance) — https://github.com/iomega/ms2query
- **MS2Deepscore** (Supplies the spectral similarity feature (average MS2Deepscore over 10 similar library molecules) expected to have the highest importance)
- **Spec2Vec** (Supplies one of the five spectral-similarity input features scored by the random forest)
- **scikit-learn** (Implements the random forest whose feature_importances_ attribute is extracted)

## Examples

```
import joblib; rf = joblib.load('ms2query_random_forest.model'); import pandas as pd; pd.DataFrame({'feature': feature_names, 'importance': rf.feature_importances_}).sort_values('importance', ascending=False).to_csv('feature_importances.csv', index=False)
```

## Evaluation signals

- The average MS2Deepscore feature has the highest Gini importance among the five features.
- All five features have nonzero importance, consistent with the reported analysis in Supplementary Table 1 and 2.
- feature_importances.csv contains exactly five rows (one per feature) sorted in descending order of importance.
- The importance values sum to ~1, as expected for scikit-learn's normalized impurity-based feature_importances_.

## Limitations

- Supplementary Tables 1 and 2 containing the full feature-importance analysis are referenced but not included in the article text, so exact per-feature values must be recovered from the Zenodo deposit.
- Impurity-based (Gini) importances reflect training-time split quality and may not capture a feature's contribution to held-out re-ranking performance; Supplementary Note 3 records other features that were tested but its content is not included.
- Importances are specific to the trained model version shipped with the library spectra; retraining MS2Deepscore/Spec2Vec models (as done for benchmarking, excluding test spectra) could shift the ranking.

## Evidence

- [intro] First, potential candidates are selected based on MS2Deepscore, followed by re-ranking the spectra by using a random forest model.: "First, potential candidates are selected based on MS2Deepscore, followed by re-ranking the spectra by using a random forest model."
- [discussion] To optimally rank these 2000 spectra, MS2Query calculates 5 features which are combined by a random forest model.: "To optimally rank these 2000 spectra, MS2Query calculates 5 features which are combined by a random forest model."
- [discussion] Additional analysis of the feature importance indicates that each of the five features used contain relevant information for correctly ranking candidate structures (Supplementary Table 1 and 2).: "Additional analysis of the feature importance indicates that each of the five features used contain relevant information for correctly ranking candidate structures (Supplementary Table 1 and 2)."
- [discussion] The implementation of scikit-learn37 was used for the random forest model.: "The implementation of scikit-learn37 was used for the random forest model."
- [discussion] The raw data, intermediate files and raw results can be found on 10.5281/zenodo.6811540.: "The raw data, intermediate files and raw results can be found on 10.5281/zenodo.6811540."
