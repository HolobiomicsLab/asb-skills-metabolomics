---
name: multivariate-feature-standardization
description: Use when a numerical feature matrix has columns on heterogeneous scales
  (e.g. iris-like morphological measurements) and PCA, clustering or another scikit-learn-based
  activity is about to be run on the raw columns.
when_to_use_negative:
- Columns are already on a common scale or the analysis is scale-invariant (e.g. rank-based
  or correlation-based methods), so z-scoring would add no benefit
- The table still contains missing values that you intend to impute first — the automatic
  pre-filter would silently drop rows before standardization
- The goal is batch-effect correction of expression data rather than plain per-feature
  scaling — use ComBat/pyComBat (bundled via the combat package) instead
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_0605
tools:
- name: Instant Clue
  role: Desktop GUI where normalization and transformation are applied per column
    via the context menu before statistical activities such as PCA (drag & drop)
  repo: https://github.com/hnolCol/instantclue
- name: scikit-learn
  role: Underlying library providing the estimator machinery (e.g. StandardScaler
    for z-score, PCA) that Instant Clue's statistical activities build on
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41598-018-31154-6
    title: 'Instant Clue: A Software Suite for Interactive Data Visualization and
      Analysis'
  source_content_hash: sha256:5dfa24b7ca5bc579a0213d6593b28b134ffb3bcb972889bf20ad02acd1ff192a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/hnolCol/instantclue
  tools:
  - Instant Clue
  - Python >=3.4
  - scikit-learn
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_0605
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# multivariate-feature-standardization

## Summary

Standardizes each numerical feature of a multivariate table (e.g. z-score per column) so that variables measured on different scales contribute equally to downstream decomposition methods such as principal component analysis.

## When to use

Use when a numerical feature matrix has columns on heterogeneous scales (e.g. iris-like morphological measurements) and PCA, clustering or another scikit-learn-based activity is about to be run on the raw columns.

## When NOT to use

- Columns are already on a common scale or the analysis is scale-invariant (e.g. rank-based or correlation-based methods), so z-scoring would add no benefit
- The table still contains missing values that you intend to impute first — the automatic pre-filter would silently drop rows before standardization
- The goal is batch-effect correction of expression data rather than plain per-feature scaling — use ComBat/pyComBat (bundled via the combat package) instead

## Inputs

- Tabular feature matrix (e.g. Excel, .txt, .csv, .xml, .gz/.zip upload) with mixed-scale numerical columns
- Iris-style dataset: 150 samples x 4 morphological features, 3 species labels

## Outputs

- Standardized feature matrix (per-column z-scored values) feeding into PCA or other scikit-learn estimators
- Column-level normalization/transformation applied via the context menu without altering the source data table

## How to apply

Load the tabular dataset (Excel, .txt, .csv, .xml, .gz or .zip uploads are supported) and select the numerical columns to be scaled. Apply per-column normalization/transformation using the column context-menu activities — in the reference workflow this is a z-score standardization of the four iris morphological features so all variables contribute equally to the decomposition. Instant Clue implements such activities on top of the scikit-learn library, so an equivalent script would instantiate a StandardScaler (z-score) over the feature matrix before fitting an estimator. Decide between mean-centering only and full z-scaling depending on whether the raw variances of the columns differ; standardize before, not after, the PCA fit. Note that activities that cannot handle missing values trigger automatic pre-filtering of rows without changing the source data, so expect row counts to shrink if NaNs are present.

## Related tools

- **Instant Clue** (Desktop GUI where normalization and transformation are applied per column via the context menu before statistical activities such as PCA (drag & drop)) — https://github.com/hnolCol/instantclue
- **scikit-learn** (Underlying library providing the estimator machinery (e.g. StandardScaler for z-score, PCA) that Instant Clue's statistical activities build on)

## Examples

```
from sklearn.preprocessing import StandardScaler; X_std = StandardScaler().fit_transform(df[['sepal_length','sepal_width','petal_length','petal_width']])
```

## Evaluation signals

- After z-scoring, each standardized column has mean ~0 and standard deviation ~1 (all four iris features scaled comparably)
- Row count is unchanged (150 iris samples) unless rows were auto-filtered due to missing values — verify which activities triggered pre-filtering
- The source data table remains unmodified: normalization is applied as an activity, not in place ('without changing the source data')
- If the standardized matrix feeds PCA, PC1 explained-variance ratio should fall in the expected ~0.90-0.95 range for standardized iris data

## Limitations

- Activities that cannot handle missing values cause automatic pre-filtering of data before submission, so rows with NaNs are dropped rather than imputed
- Instant Clue's combat dependency (pyComBat) is not in requirements.txt because of strict pandas/numpy versioning requirements; the README recommends installing combat first and ignoring the numpy/pandas version-mismatch warning
- Building custom executables with PyInstaller requires optimization; custom hooks must be requested from the developers

## Evidence

- [results] per-column normalization precedes decomposition: "Activities are applied on columns in the dataset using the context menu and cover basic steps such as sorting, string splitting and replacement, normalization and transformation"
- [results] missing-value pre-filtering: "if an activity (each test is an activity) cannot handle missing values, the data are automatically filtered before submission to the specified activity, without changing the source data"
- [intro] PCA as target statistical test: "Offering a repertoire of statistical tests like ANOVA, regressions, curve fitting and principal component analysis as well as facilitating calculation of metrics like area under the curve (AUC)"
- [readme] combat versioning limitation: "the package [combat](https://github.com/epigenelabs/pyComBat/tree/master/combat) is not in the requirements table because it has very specific versioning requirements of pandas and numpy"
