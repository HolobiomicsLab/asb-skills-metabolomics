---
name: cluster-label-table-export
description: Use when you have a tabular feature matrix (e.g. the iris dataset with
  150 samples and 4 morphological measurements) and need per-sample cluster membership
  labels from both a centroid-based (k-means) and a density-based (DBSCAN) method
  written to a shareable CSV.
when_to_use_negative:
- Input already contains cluster label columns (e.g. a pre-annotated species/category
  column) — only re-cluster when labels must be discovered, not copied
- Data is not a numeric feature matrix (e.g. raw optical recordings or single-column
  densitometry) — k-means and DBSCAN require the four-feature-style numeric table
- The goal is supervised prediction of known classes rather than unsupervised pattern
  discovery — use Instant Clue's supervised estimator/prediction pipeline instead
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_2269
tools:
- name: Instant Clue
  role: Reference interactive suite that implements k-means and DBSCAN clustering
    via drag & drop and predicts cluster membership of unseen data; this skill reproduces
    that clustering step as a script
  repo: https://github.com/hnolCol/instantclue
- name: scikit-learn
  role: Library on which Instant Clue builds its estimator/clustering functions; provides
    KMeans and DBSCAN estimators used to compute the labels
- name: pandas
  role: Assembles the per-sample label data frame (kmeans_label, dbscan_label) and
    writes iris_cluster_labels.csv
- name: numpy
  role: Numeric backbone for the feature matrix passed to the scikit-learn estimators
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41598-018-31154-6
    title: 'Instant Clue: A Software Suite for Interactive Data Visualization and
      Analysis'
  source_content_hash: sha256:d769169f3a82d17a19587c4ef06dbc96cd6f6f110b2e77568c8443c8acad0621
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
yes_rate: 0.4666666666666667
n_claims: 15
---

# cluster-label-table-export

## Summary

Fits unsupervised clustering estimators (scikit-learn k-means with a fixed k and DBSCAN with noise labeled -1) on a tabular feature matrix and exports the per-sample cluster assignments as a tidy CSV label table. It operationalizes Instant Clue's estimator/clustering capability, which is built on the scikit-learn library, as a reproducible script.

## When to use

Use when you have a tabular feature matrix (e.g. the iris dataset with 150 samples and 4 morphological measurements) and need per-sample cluster membership labels from both a centroid-based (k-means) and a density-based (DBSCAN) method written to a shareable CSV.

## When NOT to use

- Input already contains cluster label columns (e.g. a pre-annotated species/category column) — only re-cluster when labels must be discovered, not copied
- Data is not a numeric feature matrix (e.g. raw optical recordings or single-column densitometry) — k-means and DBSCAN require the four-feature-style numeric table
- The goal is supervised prediction of known classes rather than unsupervised pattern discovery — use Instant Clue's supervised estimator/prediction pipeline instead

## Inputs

- Tabular feature matrix (e.g. iris: 150 samples x 4 morphological measurements, from Excel/txt/csv/xml/gz/zip)
- Number of clusters k (e.g. 3 for the three iris species)
- DBSCAN density parameters (neighborhood radius / min points; noise points labeled -1)

## Outputs

- iris_cluster_labels.csv with exactly 150 rows, one per sample
- Two label columns per sample: kmeans_label and dbscan_label
- Per-sample identity column alongside the cluster assignments

## How to apply

Load the dataset as a pandas data frame in the same way Instant Clue ingests tabular files (Excel, .txt, .csv, .xml, .gz, .zip), mirroring its drag & drop ingestion pipeline. Fit scikit-learn's KMeans on the four measurement columns with k=3 to match the three iris species, collecting per-sample assignments. Fit scikit-learn's DBSCAN on the same feature matrix, treating noise points as label -1. Assemble a pandas data frame with one row per sample holding kmeans_label and dbscan_label columns alongside sample identity, and write it to iris_cluster_labels.csv. This mirrors how Instant Clue builds its estimator functions on scikit-learn and how the software uses clustering both to find underlying patterns and to predict cluster membership of unseen data.

## Related tools

- **Instant Clue** (Reference interactive suite that implements k-means and DBSCAN clustering via drag & drop and predicts cluster membership of unseen data; this skill reproduces that clustering step as a script) — https://github.com/hnolCol/instantclue
- **scikit-learn** (Library on which Instant Clue builds its estimator/clustering functions; provides KMeans and DBSCAN estimators used to compute the labels)
- **pandas** (Assembles the per-sample label data frame (kmeans_label, dbscan_label) and writes iris_cluster_labels.csv)
- **numpy** (Numeric backbone for the feature matrix passed to the scikit-learn estimators)

## Examples

```
from sklearn.cluster import KMeans, DBSCAN; kmeans_labels = KMeans(n_clusters=3).fit(X).labels_; dbscan_labels = DBSCAN().fit(X).labels_; df.to_csv('iris_cluster_labels.csv', index=False)
```

## Evaluation signals

- Output file iris_cluster_labels.csv exists with exactly 150 rows, one per sample
- Both kmeans_label and dbscan_label columns are present and populated for every sample
- kmeans_label resolves to exactly 3 distinct clusters, consistent with the three iris species
- DBSCAN output distinguishes density-based clusters from noise points labeled -1
- Labels are per-sample (row-aligned with the input feature matrix, no dropped or duplicated rows)

## Limitations

- k must be chosen a priori (k=3 to match the iris species structure); wrong k yields cluster counts inconsistent with known class structure
- DBSCAN labels noise points as -1; these rows are valid output but not members of any cluster
- Missing values must be handled before clustering — Instant Clue automatically filters data that activities cannot process, but the scripted export requires explicit pre-filtering
- Label integers are arbitrary (not species names); mapping to species requires post-hoc correspondence analysis

## Evidence

- [results] estimators built on scikit-learn: "Instant Clue offers several functions to establish an estimator for prediction, based on the scikit-learn library"
- [full_text] clustering capability including k-means and DBSCAN: [evidence span withheld — anchor status: not_found]
- [results] input file formats supported: "Data can be uploaded from various file types including Excel, tab delimited text (.txt) and csv-files (.csv), Extensible Markup Language (.xml) files as well as compressed files (.gz, .zip)"
- [results] missing-value handling caveat: "if an activity (each test is an activity) cannot handle missing values, the data are automatically filtered before submission to the specified activity, without changing the source data"
