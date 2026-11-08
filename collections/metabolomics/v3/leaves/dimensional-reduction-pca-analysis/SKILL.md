---
name: dimensional-reduction-pca-analysis
description: Use when the goal is unsupervised dimensional reduction of a tabular
  feature matrix (e.g., iris 4 morphological features across 3 species) to reveal
  underlying patterns or produce PC1-PCn score tables with explained-variance ratios.
when_to_use_negative:
- The analysis goal is supervised learning or prediction — use Instant Clue's interactive
  prediction pipeline (pre-processing, feature selection, estimator, grid search)
  instead of unsupervised PCA
- The task is chart annotation, subsetting via the scatter-plot selection tool, or
  figure/subplot composition rather than dimensionality reduction
- The dataset requires batch-effect correction (e.g., pyComBat) before decomposition,
  which has versioning requirements outside this skill's scope
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_2269
tools:
- name: Instant Clue
  role: Desktop GUI where PCA is initialized by a single drag & drop event from the
    Analysis toolbox for unsupervised pattern discovery
  repo: https://github.com/hnolCol/instantclue
- name: scikit-learn
  role: Library Instant Clue uses for its statistical analysis activities; provides
    the PCA estimator fitted on the feature matrix
- name: Python >=3.4
  role: Runtime platform required to run Instant Clue's Python source and the PCA
    script
  repo: https://github.com/hnolCol/instantclue
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41598-018-31154-6
    title: 'Instant Clue: A Software Suite for Interactive Data Visualization and
      Analysis'
  source_content_hash: sha256:917de56015c882ac059c6738a600cd3ff7e86cc5d4c2a18d4dca69274fc9e3ed
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
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# dimensional-reduction-pca-analysis

## Summary

Performs unsupervised principal component analysis on high-dimensional data (e.g., the 150-sample x 4-feature iris dataset) using scikit-learn's PCA estimator, mirroring Instant Clue's one-drag-and-drop PCA/cluster activity so users can identify patterns without writing code.

## When to use

Use when the goal is unsupervised dimensional reduction of a tabular feature matrix (e.g., iris 4 morphological features across 3 species) to reveal underlying patterns or produce PC1-PCn score tables with explained-variance ratios.

## When NOT to use

- The analysis goal is supervised learning or prediction — use Instant Clue's interactive prediction pipeline (pre-processing, feature selection, estimator, grid search) instead of unsupervised PCA
- The task is chart annotation, subsetting via the scatter-plot selection tool, or figure/subplot composition rather than dimensionality reduction
- The dataset requires batch-effect correction (e.g., pyComBat) before decomposition, which has versioning requirements outside this skill's scope

## Inputs

- Tabular dataset with numerical feature columns (e.g., iris: 150 samples x 4 morphological features, .csv/.txt/.xlsx, Fisher 1936 source or bundled tutorial copy)
- Optional species/label column carried through unsupervised projection

## Outputs

- pca_scores_table.csv with one row per sample, PC1-PC4 score columns, species label, and an explained_variance_ratio field
- Per-component explained-variance ratios (PC1 expected ~0.90-0.95 for iris)

## How to apply

Load the tabular dataset (csv/txt/xlsx) with pandas, optionally z-score-standardize the numerical feature columns so all variables contribute equally — matching the normalization activities Instant Clue applies on columns via the context menu. Fit a full-rank PCA (all 4 components for iris) with scikit-learn's PCA estimator, the library Instant Clue uses for its statistical-analysis activities. Project samples onto the components, record per-component explained_variance_ratio, and write a single table with one row per sample (PC1-PC4 scores plus species label) as pca_scores_table.csv. In the Instant Clue GUI, the equivalent is a single drag & drop of the PCA activity from the Analysis toolbox, which internally relies on scikit-learn estimators.

## Related tools

- **Instant Clue** (Desktop GUI where PCA is initialized by a single drag & drop event from the Analysis toolbox for unsupervised pattern discovery) — https://github.com/hnolCol/instantclue
- **scikit-learn** (Library Instant Clue uses for its statistical analysis activities; provides the PCA estimator fitted on the feature matrix)
- **Python >=3.4** (Runtime platform required to run Instant Clue's Python source and the PCA script) — https://github.com/hnolCol/instantclue

## Examples

```
from sklearn.decomposition import PCA; X4 = PCA(n_components=4).fit_transform(X)
```

## Evaluation signals

- Output file pca_scores_table.csv exists with exactly 150 data rows for the iris dataset
- The table contains exactly 4 principal-component score columns (PC1-PC4)
- PC1 explained-variance ratio falls in the expected range ~0.90-0.95 for iris
- Explained-variance ratios are recorded per component and sum to at most 1.0
- The species label column is preserved row-aligned with the PC scores; remaining numeric plausibility judgments go to expert_review

## Limitations

- Activities that cannot handle missing values trigger automatic pre-filtering of data before submission, without changing the source data — rows with NaNs may be silently dropped
- The panel of statistical tests is not yet fully implemented; further activities for systematic data interpretation are planned but not added
- Running Instant Clue from source requires installing combat (pyComBat) first due to specific pandas/numpy versioning requirements, and creating executables with PyInstaller 'requires some optimization'

## Evidence

- [results] scikit-learn estimator basis: "Instant Clue offers several functions to establish an estimator for prediction, based on the scikit-learn library"
- [results] drag & drop analysis trigger: "statistical tests are enabled by a drag & drop event from the Analysis toolbox onto generated charts"
- [results] input file formats: "Data can be uploaded from various file types including Excel, tab delimited text (.txt) and csv-files (.csv)"
- [intro] PCA among offered tests: "Offering a repertoire of statistical tests like ANOVA, regressions, curve fitting and principal component analysis"
- [results] column normalization activities: "Activities are applied on columns in the dataset using the context menu and cover basic steps such as sorting, string splitting and replacement, normalization"
- [results] missing-value handling: "if an activity (each test is an activity) cannot handle missing values, the data are automatically filtered before submission to the specified activity"
