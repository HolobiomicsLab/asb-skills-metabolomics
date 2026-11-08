---
name: benchmark-dataset-clustering-reconstruction
description: Use when a task asks to identify underlying patterns in an unlabeled
  tabular dataset (e.g., iris with 150 samples x 4 morphological features) by fitting
  k-means and/or DBSCAN and exporting per-sample cluster assignments.
when_to_use_negative:
- When the goal is supervised classification with known species/labels — use Instant
  Clue's scikit-learn estimator/prediction pipeline instead of unsupervised clustering
- When the input is already a cluster-label table or a precomputed clustering result
  — re-fitting k-means/DBSCAN would duplicate the analysis
- When the dataset has many missing values in the feature matrix — Instant Clue notes
  activities that cannot handle missing values require pre-filtering before clustering
edam_operation: http://edamontology.org/operation_3432
tools:
- name: scikit-learn
  role: Provides the KMeans and DBSCAN estimators Instant Clue's clustering functions
    build on
- name: Instant Clue
  role: Reference desktop application demonstrating drag & drop unsupervised clustering
    and cluster-membership prediction on the iris example
  repo: https://github.com/hnolCol/instantclue
- name: pandas
  role: Assembles the per-sample label data frame and writes iris_cluster_labels.csv
- name: Python
  role: Runtime environment; Instant Clue itself requires Python >= 3.4 and installs
    from requirements.txt in a virtualenv
  repo: https://github.com/hnolCol/instantclue
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41598-018-31154-6
    title: 'Instant Clue: A Software Suite for Interactive Data Visualization and
      Analysis'
  source_content_hash: sha256:8bdb9646cd3e85f00aff8c94cf94804b2936214b7818590bb89976574f8fdd4b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/hnolCol/instantclue
  tools:
  - Instant Clue
  - Python
  - numpy
  - pandas
  - scikit-learn
  edam_operation: http://edamontology.org/operation_3432
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# benchmark-dataset-clustering-reconstruction

## Summary

Reproduces the unsupervised clustering capability demonstrated by Instant Clue (k-means and DBSCAN via scikit-learn) on a tabular benchmark dataset such as the 150-sample, 4-feature iris example, emitting per-sample cluster labels for algorithm comparison.

## When to use

Use when a task asks to identify underlying patterns in an unlabeled tabular dataset (e.g., iris with 150 samples x 4 morphological features) by fitting k-means and/or DBSCAN and exporting per-sample cluster assignments.

## When NOT to use

- When the goal is supervised classification with known species/labels — use Instant Clue's scikit-learn estimator/prediction pipeline instead of unsupervised clustering
- When the input is already a cluster-label table or a precomputed clustering result — re-fitting k-means/DBSCAN would duplicate the analysis
- When the dataset has many missing values in the feature matrix — Instant Clue notes activities that cannot handle missing values require pre-filtering before clustering

## Inputs

- Tabular data frame of unlabeled samples (iris: 150 samples x 4 morphological features, loadable from Excel, .txt, .csv, .xml, .gz or .zip per Instant Clue's supported file types)
- Chosen cluster count for k-means (k=3 for the iris species structure)

## Outputs

- iris_cluster_labels.csv with 150 rows and columns kmeans_label and dbscan_label keyed by sample identity
- DBSCAN noise-point assignments (label -1) distinguishing density-based outliers from k-means partitions

## How to apply

Load the tabular dataset (Excel/txt/csv per Instant Clue's supported inputs) into a pandas data frame and extract the numeric feature matrix — for iris, the 4 morphological measurements. Fit k-means from scikit-learn (the same library Instant Clue builds its estimator functions on) with k=3, chosen to match the three iris species, and collect per-sample assignments. Fit DBSCAN on the same feature matrix for a density-based view, treating label -1 as noise points. Assemble a single data frame with one row per sample containing kmeans_label and dbscan_label alongside sample identity, and write it to iris_cluster_labels.csv. Rationale: Instant Clue demonstrates that its clustering can also predict cluster membership of unseen data, so keeping labels sample-keyed supports later membership prediction checks.

## Related tools

- **scikit-learn** (Provides the KMeans and DBSCAN estimators Instant Clue's clustering functions build on)
- **Instant Clue** (Reference desktop application demonstrating drag & drop unsupervised clustering and cluster-membership prediction on the iris example) — https://github.com/hnolCol/instantclue
- **pandas** (Assembles the per-sample label data frame and writes iris_cluster_labels.csv)
- **Python** (Runtime environment; Instant Clue itself requires Python >= 3.4 and installs from requirements.txt in a virtualenv) — https://github.com/hnolCol/instantclue

## Examples

```
from sklearn.cluster import KMeans, DBSCAN; km = KMeans(n_clusters=3).fit_predict(X); db = DBSCAN().fit_predict(X)
```

## Evaluation signals

- iris_cluster_labels.csv exists and contains exactly 150 rows, one per iris sample
- Output contains the two required columns kmeans_label and dbscan_label populated for every sample
- kmeans_label resolves to exactly 3 distinct clusters, consistent with the three iris species
- DBSCAN noise points carry label -1, distinguishing them from density-based cluster members
- k-means assignments cross-tabulated against any available species labels should show cluster/species agreement consistent with the iris structure

## Limitations

- k-means requires k to be fixed a priori (k=3 was chosen because the iris dataset has three species); other datasets need their own k decision, e.g., via the exhaustive grid searching over estimator parameters that Instant Clue exposes
- DBSCAN may mark genuine samples as noise (label -1), so its labels are not directly comparable 1:1 with k-means partitions
- Activities that cannot handle missing values require automatic pre-filtering of rows before clustering; the source data should remain unchanged

## Evidence

- [abstract] portfolio of statistical tools: "it offers a comprehensive portfolio of statistical tools for systematic analysis such as dimensional reduction, (un)-supervised learning, clustering"
- [results] scikit-learn estimators: "Instant Clue offers several functions to establish an estimator for prediction, based on the scikit-learn library"
- [results] grid search tuning: "Users can optimize pre-processing of data, feature selection/reduction and estimator parameters using exhaustive grid searching over given parameters"
- [readme] Python-based application: "Instant Clue is a Python based desktop application (GUI) using the PyQt6 (since 0.12.0) library for data visualization and analysis."
