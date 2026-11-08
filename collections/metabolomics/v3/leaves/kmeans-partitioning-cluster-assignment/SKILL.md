---
name: kmeans-partitioning-cluster-assignment
description: 'Use when a small, fully numeric tabular dataset (like the iris example:
  150 samples × 4 morphological features) lacks labels and the goal is to discover
  whether samples group into a known or expected number of clusters (k=3).'
when_to_use_negative:
- Sample labels are already known and the goal is supervised prediction rather than
  unsupervised pattern discovery — use Instant Clue's prediction/estimator pipeline
  instead.
- The input is not a numeric tabular feature matrix (e.g. raw omics reads or images
  needing prior feature extraction), since the activity operates on data-frame columns.
- The dataset has many missing values in the feature columns; activities that cannot
  handle missing values trigger automatic pre-filtering that would silently drop rows
  before clustering.
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_2269
tools:
- name: scikit-learn
  role: Provides the KMeans and DBSCAN estimators used for the per-sample cluster
    assignment, the same library Instant Clue bases its estimator functions on
- name: pandas
  role: Assembles the one-row-per-sample label table and writes iris_cluster_labels.csv
- name: Instant Clue
  role: Reference GUI application that performs the same k-means/DBSCAN clustering
    via drag & drop on ingested tabular data
  repo: https://github.com/hnolCol/instantclue
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41598-018-31154-6
    title: 'Instant Clue: A Software Suite for Interactive Data Visualization and
      Analysis'
  source_content_hash: sha256:2f100dae5072efc4adb1704b55dde0e4a138aa566d2b0f7ac91a99aa4bb16b0b
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
  edam_topics:
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5882352941176471
n_claims: 17
---

# kmeans-partitioning-cluster-assignment

## Summary

Unsupervised partitioning of a tabular feature matrix (e.g. the 150-sample, 4-feature iris dataset) into k clusters with scikit-learn's k-means, mirroring the clustering activity Instant Clue exposes via drag & drop, optionally complemented by DBSCAN for density-based assignment.

## When to use

Use when a small, fully numeric tabular dataset (like the iris example: 150 samples × 4 morphological features) lacks labels and the goal is to discover whether samples group into a known or expected number of clusters (k=3).

## When NOT to use

- Sample labels are already known and the goal is supervised prediction rather than unsupervised pattern discovery — use Instant Clue's prediction/estimator pipeline instead.
- The input is not a numeric tabular feature matrix (e.g. raw omics reads or images needing prior feature extraction), since the activity operates on data-frame columns.
- The dataset has many missing values in the feature columns; activities that cannot handle missing values trigger automatic pre-filtering that would silently drop rows before clustering.

## Inputs

- Tabular feature matrix as data frame (e.g. iris: 150 samples × 4 morphological measurements), loadable from .xlsx/.txt/.csv/.xml/.gz/.zip
- Prespecified cluster count k (e.g. k=3 matching the three iris species)

## Outputs

- iris_cluster_labels.csv with 150 rows and columns sample identity, kmeans_label, dbscan_label

## How to apply

Load the tabular data (Excel, txt, csv, xml, gz, zip are ingestible formats in the Instant Clue pipeline) into a pandas data frame and keep only the numeric feature columns. Fit scikit-learn's KMeans with n_clusters=3 — chosen to match the three iris species — and collect per-sample cluster assignments; this mirrors how Instant Clue builds its estimator functions on the scikit-learn library. Optionally fit DBSCAN on the same feature matrix as a density-based cross-check, where noise points receive label -1. Assemble a pandas data frame with one row per sample holding kmeans_label and dbscan_label alongside sample identity, and write it to iris_cluster_labels.csv. The rationale is that Instant Clue's unsupervised learning capability exists precisely to identify underlying patterns in high-dimensional data without a computational background, so the scripted equivalent should stay simple and reproducible.

## Related tools

- **scikit-learn** (Provides the KMeans and DBSCAN estimators used for the per-sample cluster assignment, the same library Instant Clue bases its estimator functions on)
- **pandas** (Assembles the one-row-per-sample label table and writes iris_cluster_labels.csv)
- **Instant Clue** (Reference GUI application that performs the same k-means/DBSCAN clustering via drag & drop on ingested tabular data) — https://github.com/hnolCol/instantclue

## Examples

```
from sklearn.cluster import KMeans, DBSCAN; km_labels = KMeans(n_clusters=3).fit_predict(iris.data); db_labels = DBSCAN().fit_predict(iris.data)
```

## Evaluation signals

- Output file iris_cluster_labels.csv exists and contains exactly 150 rows, one per iris sample.
- Both kmeans_label and dbscan_label columns are populated for every sample (no NaNs).
- The kmeans_label column resolves to exactly 3 distinct clusters, consistent with the three-species iris structure.
- DBSCAN labels are integers with noise points coded as -1.
- Cluster assignments can be cross-tabulated against species identity to confirm agreement of the partition with the known group structure.

## Limitations

- k-means requires k to be specified in advance; choosing k=3 here is justified only because the iris species count is known, and it does not generalize to datasets without prior cluster-count knowledge.
- DBSCAN may assign noise points label -1, which must be handled separately when comparing partitions.
- If an activity cannot handle missing values, Instant Clue automatically pre-filters data before submission, so rows with NaNs may be silently excluded from clustering.
- Reproducing Instant Clue from source requires installing pyComBat/combat first because of its specific pandas/numpy versioning requirements, before running pip install -r requirements.txt.

## Evidence

- [results] scikit-learn clustering basis: "Instant Clue offers several functions to establish an estimator for prediction, based on the scikit-learn library"
- [abstract] clustering in the statistical portfolio: "it offers a comprehensive portfolio of statistical tools for systematic analysis such as dimensional reduction, (un)-supervised learning, clustering, multi-block (omics) integration"
- [results] ingestible file formats: "Data can be uploaded from various file types including Excel, tab delimited text (.txt) and csv-files (.csv), Extensible Markup Language (.xml) files as well as compressed files (.gz, .zip)"
- [readme] Instant Clue implementation stack: "Instant Clue is a Python based desktop application (GUI) using the PyQt6 (since 0.12.0) library for data visualization and analysis."
