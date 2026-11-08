---
name: high-dimensional-data-reduction-visualization
description: Use when a numeric feature matrix (e.g. 150 samples x 4 morphological
  iris measurements, or high-dimensional omics columns) should be projected onto principal
  components to identify unsupervised patterns and visualize sample structure.
when_to_use_negative:
- The analysis goal is supervised (prediction or classification using the label as
  target) rather than unsupervised pattern discovery — use Instant Clue's estimator/prediction
  pipeline instead.
- The data contain batch effects that must be corrected first — apply a ComBat-style
  method (combat/pyComBat) before reduction.
- The matrix has only one or two variables, so a dimensional-reduction step adds no
  information.
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: Instant Clue
  role: Desktop GUI in which PCA (and cluster analysis) is initialized by a single
    drag & drop event for unsupervised dimensional reduction
  repo: https://github.com/hnolCol/instantclue
- name: scikit-learn
  role: Library supplying the PCA estimator and other statistical functions underlying
    Instant Clue's analysis activities
- name: Python >=3.4
  role: Runtime required to run Instant Clue from source on any platform
  repo: https://github.com/hnolCol/instantclue
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41598-018-31154-6
    title: 'Instant Clue: A Software Suite for Interactive Data Visualization and
      Analysis'
  source_content_hash: sha256:b3ce8a15c76b322173ccf8f7bc0939844fb6878aee4114b5dc7f25ffa95f5e79
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/hnolCol/instantclue
  tools:
  - Instant Clue
  - Python >=3.4
  - scikit-learn
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# high-dimensional-data-reduction-visualization

## Summary

Unsupervised dimensional reduction (principal component analysis) of a high-dimensional feature table, initialized by a single drag & drop event in Instant Clue and backed by scikit-learn estimators, to reveal underlying patterns without writing code.

## When to use

Use when a numeric feature matrix (e.g. 150 samples x 4 morphological iris measurements, or high-dimensional omics columns) should be projected onto principal components to identify unsupervised patterns and visualize sample structure.

## When NOT to use

- The analysis goal is supervised (prediction or classification using the label as target) rather than unsupervised pattern discovery — use Instant Clue's estimator/prediction pipeline instead.
- The data contain batch effects that must be corrected first — apply a ComBat-style method (combat/pyComBat) before reduction.
- The matrix has only one or two variables, so a dimensional-reduction step adds no information.

## Inputs

- Numeric feature matrix with one row per sample and continuous columns (e.g. 150 iris samples x 4 morphological features)
- Categorical label column (e.g. iris species) for coloring/annotating the PCA projection
- Data file in Excel, .txt, .csv, .xml, .gz or .zip format as supported by Instant Clue upload

## Outputs

- Per-sample principal-component score table (PC1-PC4 columns plus species label) saved as pca_scores_table.csv
- Explained-variance-ratio values per component (expected PC1 ~0.90-0.95 for iris)
- Drag & drop PCA chart for visual inspection and candidate annotation

## How to apply

Load the feature table (Instant Clue accepts Excel, .txt, .csv, .xml, .gz and .zip inputs) and optionally z-score standardize the numeric columns via the context-menu normalization activities so all variables contribute equally to the decomposition. Trigger the PCA activity by a single drag & drop of the analysis onto the chart, which instantiates the scikit-learn-based estimator underlying Instant Clue's statistical functions. Fit a full-rank decomposition (all components, e.g. PC1-PC4 for iris), project samples, and record per-component explained-variance ratios. Export a scores table with one row per sample, columns PC1-PC4 plus the species/category label and an explained_variance_ratio field (e.g. pca_scores_table.csv). Validate by checking the row/column schema and that the PC1 explained-variance ratio falls in the expected range (~0.90-0.95 for iris); plausibility judgments beyond that go to expert review.

## Related tools

- **Instant Clue** (Desktop GUI in which PCA (and cluster analysis) is initialized by a single drag & drop event for unsupervised dimensional reduction) — https://github.com/hnolCol/instantclue
- **scikit-learn** (Library supplying the PCA estimator and other statistical functions underlying Instant Clue's analysis activities)
- **Python >=3.4** (Runtime required to run Instant Clue from source on any platform) — https://github.com/hnolCol/instantclue

## Examples

```
from sklearn.decomposition import PCA; scores = PCA(n_components=4).fit_transform(X); ratios = PCA(n_components=4).fit(X).explained_variance_ratio_
```

## Evaluation signals

- Output file pca_scores_table.csv exists with exactly 150 data rows (one per iris sample) and 4 principal-component score columns (PC1-PC4).
- PC1 explained-variance ratio falls in the expected ~0.90-0.95 range for the iris dataset.
- An explained_variance_ratio field/row is present alongside the per-sample scores.
- The PCA was reached without code, via a single drag & drop event, mirroring Instant Clue's intended interaction model.
- Remaining numeric plausibility judgments are deferred to expert_review rather than silently accepted.

## Limitations

- Activities that cannot handle missing values trigger automatic pre-filtering before submission, which can silently change the analyzed sample set (source data are unchanged).
- The panel of statistical tests and supporting activities was still being extended at publication, so some tests may be unavailable.
- z-score standardization of columns is effectively required when features are on different scales, otherwise variables contribute unequally to the decomposition.
- Running Instant Clue from source requires a Python >=3.4 virtual environment; combat must be installed first and outside requirements.txt because of specific pandas/numpy versioning requirements.

## Evidence

- [abstract] Instant Clue combines visual and statistical analytics via drag & drop: "Instant Clue combines the power of visual and statistical analytics using a straight forward drag & drop approach making the software highly intuitive"
- [results] Statistical engine is scikit-learn: "Instant Clue offers several functions to establish an estimator for prediction, based on the scikit-learn library"
- [results] Column-level pre-processing activities including normalization: "Activities are applied on columns in the dataset using the context menu and cover basic steps such as sorting, string splitting and replacement, normalization and transformation"
- [results] Input file formats supported by Instant Clue: "Data can be uploaded from various file types including Excel, tab delimited text (.txt) and csv-files (.csv), Extensible Markup Language (.xml) files as well as compressed files (.gz, .zip)"
- [readme] Instant Clue is a Python desktop GUI for visualization and analysis: "Instant Clue is a Python based desktop application (GUI) using the PyQt6 (since 0.12.0) library for data visualization and analysis"
