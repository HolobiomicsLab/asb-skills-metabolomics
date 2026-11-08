---
name: dbscan-density-based-clustering
description: Use when you have an unlabeled tabular feature matrix (e.g. 150 samples
  × 4 iris measurements) and need density-based cluster assignment without pre-specifying
  k, with noise points flagged as -1.
when_to_use_negative:
- The data has known ground-truth labels and the goal is supervised classification
  — Instant Clue's estimator pipeline with feature selection and grid search is the
  appropriate path instead
- The analysis requires an exact number of clusters (e.g. exactly k=3 species); DBSCAN
  does not guarantee a target k, so k-means is the correct estimator for that constraint
- The feature matrix lacks meaningful density structure (e.g. uniformly distributed
  points), which would leave most or all samples labeled -1 as noise
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3473
tools:
- name: scikit-learn
  role: Provides the DBSCAN and k-means estimators used to generate per-sample cluster
    labels on the iris feature matrix
- name: Instant Clue
  role: Reference GUI whose drag & drop clustering workflow (k-means and DBSCAN on
    the iris example) this skill reproduces programmatically
  repo: https://github.com/hnolCol/instantclue
- name: Python >=3.4
  role: Runtime for running the scikit-learn clustering scripts (Instant Clue requires
    a working Python version >=3.4)
  repo: https://github.com/hnolCol/instantclue
- name: pandas
  role: Assembles the per-sample data frame with kmeans_label and dbscan_label columns
    and writes iris_cluster_labels.csv
- name: numpy
  role: Numeric backend for the feature matrix consumed by the scikit-learn estimators
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41598-018-31154-6
    title: 'Instant Clue: A Software Suite for Interactive Data Visualization and
      Analysis'
  source_content_hash: sha256:5be6e1e0ec2c197065dcc4c0b07dd5225ff2150d36ee2c22a289850525059657
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/hnolCol/instantclue
  tools:
  - Instant Clue
  - Mass-Suite (MSS)
  - Python
  - Python >=3.4
  - numpy
  - pandas
  - scikit-learn
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3473
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# dbscan-density-based-clustering

## Summary

Applies density-based clustering (DBSCAN) alongside k-means to discover underlying patterns in tabular data, mirroring the unsupervised clustering Instant Clue exposes via drag & drop on its bundled iris example data, using the scikit-learn library.

## When to use

Use when you have an unlabeled tabular feature matrix (e.g. 150 samples × 4 iris measurements) and need density-based cluster assignment without pre-specifying k, with noise points flagged as -1.

## When NOT to use

- The data has known ground-truth labels and the goal is supervised classification — Instant Clue's estimator pipeline with feature selection and grid search is the appropriate path instead
- The analysis requires an exact number of clusters (e.g. exactly k=3 species); DBSCAN does not guarantee a target k, so k-means is the correct estimator for that constraint
- The feature matrix lacks meaningful density structure (e.g. uniformly distributed points), which would leave most or all samples labeled -1 as noise

## Inputs

- Tabular data frame of numeric features (e.g. iris: 150 samples × 4 morphological measurements) loaded from Excel, txt, csv, xml, gz or zip
- Optional DBSCAN parameters (eps, min_samples) supplied by the analyst

## Outputs

- iris_cluster_labels.csv with 150 rows and per-sample sample identity, kmeans_label, and dbscan_label columns
- DBSCAN label vector with noise points labeled -1

## How to apply

Load the tabular data (Excel, txt, csv, xml, gz or zip are acceptable input formats in the Instant Clue ecosystem) as a pandas data frame; for the iris example this is 150 samples with 4 morphological features. Fit DBSCAN from scikit-learn — the same library Instant Clue builds its estimator/clustering functions on — on the feature matrix, collecting per-sample labels where noise points receive label -1. In parallel fit k-means with k=3 (matching the three iris species) to provide a k-fixed contrast that validates the density structure. Assemble a pandas data frame with one row per sample holding kmeans_label and dbscan_label columns alongside sample identity, and write it to iris_cluster_labels.csv. Rationale: Instant Clue demonstrates this pattern on iris (Fig. S5 shows raw PCA and k-means output), and its clustering algorithms can also predict cluster membership of unseen data, so labels should be produced in a form re-usable for prediction.

## Related tools

- **scikit-learn** (Provides the DBSCAN and k-means estimators used to generate per-sample cluster labels on the iris feature matrix)
- **Instant Clue** (Reference GUI whose drag & drop clustering workflow (k-means and DBSCAN on the iris example) this skill reproduces programmatically) — https://github.com/hnolCol/instantclue
- **Python >=3.4** (Runtime for running the scikit-learn clustering scripts (Instant Clue requires a working Python version >=3.4)) — https://github.com/hnolCol/instantclue
- **pandas** (Assembles the per-sample data frame with kmeans_label and dbscan_label columns and writes iris_cluster_labels.csv)
- **numpy** (Numeric backend for the feature matrix consumed by the scikit-learn estimators)

## Examples

```
from sklearn.cluster import DBSCAN; dbscan_labels = DBSCAN().fit_predict(iris_features)
```

## Evaluation signals

- iris_cluster_labels.csv exists and contains exactly 150 rows (one per iris sample)
- Output contains both kmeans_label and dbscan_label columns populated for every sample
- The kmeans_label column resolves to exactly 3 distinct clusters, consistent with the three iris species
- DBSCAN output uses -1 as the noise-point label and integer labels for density-connected clusters
- Labels are re-usable for prediction of unseen samples, consistent with Instant Clue's stated ability to predict cluster membership of new data

## Limitations

- DBSCAN label count is data-dependent — unlike the k-means run with k=3, DBSCAN may return a different number of clusters or mark many samples as noise (-1) depending on eps/min_samples
- Activities that cannot handle missing values need automatic pre-filtering before clustering; in Instant Clue, 'if an activity (each test is an activity) cannot handle missing values, the data are automatically filtered before submission to the specified activity'
- The statistical-test panel of Instant Clue was still being extended at publication, so some downstream statistical activities may not be available

## Evidence

- [results] scikit-learn estimators underpin Instant Clue clustering: "Instant Clue offers several functions to establish an estimator for prediction, based on the scikit-learn library"
- [abstract] clustering is part of Instant Clue's statistical portfolio: "it offers a comprehensive portfolio of statistical tools for systematic analysis such as dimensional reduction, (un)-supervised learning, clustering, multi-block (omics) integration and curve fitting"
- [intro] Python runtime requirement for running the code: "the python source code is also freely available making Instant Clue editable and available on any platform with a working Python version (>=3.4)"
- [readme] how to set up the Python environment to run clustering code from the source: "Open terminal (mac) or command line tool (windows) and navigate to instantclue/src/main/python"
