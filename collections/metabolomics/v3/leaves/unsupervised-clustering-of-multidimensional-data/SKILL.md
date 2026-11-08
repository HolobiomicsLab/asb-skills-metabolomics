---
name: unsupervised-clustering-of-multidimensional-data
description: Use when an unlabeled feature table (rows = samples, columns = numerical
  measurements) needs pattern discovery, e.g. grouping samples by multiple morphological
  or omics features without ground-truth labels.
when_to_use_negative:
- The samples already carry class labels and the goal is supervised prediction rather
  than pattern discovery — Instant Clue's estimator pipeline with grid search is the
  appropriate path there
- The table is not a numerical feature matrix (e.g. only categorical annotation columns);
  k-means and DBSCAN require numerical measurements
- The task is dimensionality reduction or visualization of the data (e.g. PCA) rather
  than cluster membership assignment
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_0092
tools:
- name: Instant Clue
  role: GUI application whose clustering functions (k-means, DBSCAN) via drag & drop
    this workflow reproduces programmatically; can also predict cluster membership
    of unseen data
  repo: https://github.com/hnolCol/instantclue
- name: scikit-learn
  role: Library underlying Instant Clue's estimator/clustering functions; source of
    the KMeans (k=3) and DBSCAN implementations used here
- name: pandas
  role: Assemble the per-sample data frame with kmeans_label and dbscan_label columns
    and write iris_cluster_labels.csv
- name: Python >=3.4
  role: Runtime for the clustering script; Instant Clue's source requires any platform
    with a working Python version (>=3.4)
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41598-018-31154-6
    title: 'Instant Clue: A Software Suite for Interactive Data Visualization and
      Analysis'
  source_content_hash: sha256:ecf66458b9ae10487ebf3356fbe3cd66700b9a7027802b78404b2eeecd81c0ba
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/hnolCol/instantclue
  tools:
  - Instant Clue
  - Python
  - Python >=3.4
  - numpy
  - pandas
  - scikit-learn
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# unsupervised-clustering-of-multidimensional-data

## Summary

Applies unsupervised clustering (k-means and DBSCAN, the same estimator style Instant Clue builds on scikit-learn) to a multidimensional tabular dataset such as the 150-sample, 4-feature iris data to identify underlying patterns and assign cluster memberships.

## When to use

Use when an unlabeled feature table (rows = samples, columns = numerical measurements) needs pattern discovery, e.g. grouping samples by multiple morphological or omics features without ground-truth labels.

## When NOT to use

- The samples already carry class labels and the goal is supervised prediction rather than pattern discovery — Instant Clue's estimator pipeline with grid search is the appropriate path there
- The table is not a numerical feature matrix (e.g. only categorical annotation columns); k-means and DBSCAN require numerical measurements
- The task is dimensionality reduction or visualization of the data (e.g. PCA) rather than cluster membership assignment

## Inputs

- Tabular feature table (e.g. Excel, .txt, .csv, .xml, .gz or .zip) with numerical measurement columns per sample
- Iris example dataset: 150 samples x 4 morphological features (mirroring Instant Clue's included Example Data)
- Expected number of groups (k=3 for the three iris species) for k-means

## Outputs

- iris_cluster_labels.csv with one row per sample and columns kmeans_label and dbscan_label alongside sample identity
- Per-sample DBSCAN labels including -1 noise assignments

## How to apply

Load the tabular data frame (Instant Clue ingests Excel, .txt, .csv, .xml, .gz and .zip) and extract the numerical feature matrix — for the iris example, the 150 samples x 4 morphological measurements. Fit k-means from scikit-learn (the library Instant Clue bases its estimator functions on) with k set to the expected group count (k=3 for the three iris species) and collect per-sample cluster assignments. Fit DBSCAN on the same feature matrix for a density-based view, where noise points receive the label -1; compare the two partitions to judge cluster robustness. Assemble a pandas data frame with one row per sample containing kmeans_label and dbscan_label columns alongside sample identity, and write it to iris_cluster_labels.csv. Validation: the file must have 150 rows, both cluster-label columns populated for every sample, and the k-means column must resolve to exactly 3 distinct clusters consistent with the iris species structure.

## Related tools

- **Instant Clue** (GUI application whose clustering functions (k-means, DBSCAN) via drag & drop this workflow reproduces programmatically; can also predict cluster membership of unseen data) — https://github.com/hnolCol/instantclue
- **scikit-learn** (Library underlying Instant Clue's estimator/clustering functions; source of the KMeans (k=3) and DBSCAN implementations used here)
- **pandas** (Assemble the per-sample data frame with kmeans_label and dbscan_label columns and write iris_cluster_labels.csv)
- **Python >=3.4** (Runtime for the clustering script; Instant Clue's source requires any platform with a working Python version (>=3.4))

## Examples

```
from sklearn.cluster import KMeans; labels = KMeans(n_clusters=3).fit(iris_df.iloc[:, :4]).labels_
```

## Evaluation signals

- Output file iris_cluster_labels.csv exists with exactly 150 rows matching the iris sample count
- Both kmeans_label and dbscan_label columns are populated for every sample (no missing assignments)
- The k-means column resolves to exactly 3 distinct clusters, consistent with the three iris species structure
- DBSCAN labels form a sensible density partition with noise points explicitly labeled -1
- Per-sample assignments in the two columns can be cross-compared to check whether k-means and DBSCAN agree on the same underlying groups

## Limitations

- k-means requires choosing k in advance (k=3 matches the three iris species, but an incorrect k misrepresents structure); DBSCAN's density parameters determine which points are flagged as noise (-1)
- Activities that cannot handle missing values require automatic pre-filtering of the data before submission, which can change the effective sample set
- Instant Clue is desktop-oriented (Windows/Mac executables, PyQt6 GUI); the script-based route used here runs outside the drag & drop application

## Evidence

- [abstract] it offers a comprehensive portfolio of statistical tools for systematic analysis such as dimensional reduction, (un)-supervised learning, clustering, multi-block (omics) integration and curve fitting: "it offers a comprehensive portfolio of statistical tools for systematic analysis such as dimensional reduction, (un)-supervised learning, clustering"
- [results] Instant Clue offers several functions to establish an estimator for prediction, based on the scikit-learn library: "Instant Clue offers several functions to establish an estimator for prediction, based on the scikit-learn library"
- [intro] the python source code is also freely available making Instant Clue editable and available on any platform with a working Python version (>=3.4): "the python source code is also freely available making Instant Clue editable and available on any platform with a working Python version (>=3.4)"
- [readme] Instant Clue is a Python based desktop application (GUI) using the PyQt6 (since 0.12.0) library for data visualization and analysis.: "Instant Clue is a Python based desktop application (GUI) using the PyQt6 (since 0.12.0) library for data visualization and analysis."
