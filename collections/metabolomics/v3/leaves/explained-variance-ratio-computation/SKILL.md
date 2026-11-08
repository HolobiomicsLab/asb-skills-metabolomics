---
name: explained-variance-ratio-computation
description: Use when a fitted PCA decomposition must be quantified — after projecting
  samples onto PCs, to record how much variance each component explains and validate
  that the component captures the expected share of high-dimensional feature variance.
when_to_use_negative:
- The task is supervised — Instant Clue separates (un)-supervised learning, and explained-variance
  ratios are only defined for unsupervised decompositions like PCA, not for prediction
  pipelines with estimator/grid-search steps
- The input is already a PCA score table or the components' explained variance is
  already reported, so recomputation would be redundant
- The data contain many missing values that cannot be handled — Instant Clue automatically
  filters such rows before activities, which would change the variance base; resolve
  imputation/filtering first
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3474
tools:
- name: scikit-learn
  role: PCA estimator whose fit on the standardized feature matrix yields the explained_variance_ratio_
    per principal component
- name: Instant Clue
  role: Desktop GUI that exposes PCA via drag & drop onto charts and applies column
    normalization/transformation via the context menu, the workflow this computation
    mirrors
  repo: https://github.com/hnolCol/instantclue
- name: Python >=3.4
  role: Runtime for the pandas/scikit-learn script that loads the iris data, fits
    PCA, and writes pca_scores_table.csv
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41598-018-31154-6
    title: 'Instant Clue: A Software Suite for Interactive Data Visualization and
      Analysis'
  source_content_hash: sha256:c74fb5266db15c4955699462d208673f4ecda42a801de65d38dd89ea0bed3d94
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
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# explained-variance-ratio-computation

## Summary

Compute and report the per-component explained-variance ratios after fitting a principal component analysis on a feature matrix, so that the fraction of dataset variance captured by each principal component (e.g. PC1 on Fisher's iris data, expected ~0.90-0.95) can be used to judge how much dimensionality reduction is justified.

## When to use

Use when a fitted PCA decomposition must be quantified — after projecting samples onto PCs, to record how much variance each component explains and validate that the leading component captures the expected share of high-dimensional feature variance.

## When NOT to use

- The task is supervised — Instant Clue separates (un)-supervised learning, and explained-variance ratios are only defined for unsupervised decompositions like PCA, not for prediction pipelines with estimator/grid-search steps
- The input is already a PCA score table or the components' explained variance is already reported, so recomputation would be redundant
- The data contain many missing values that cannot be handled — Instant Clue automatically filters such rows before activities, which would change the variance base; resolve imputation/filtering first

## Inputs

- Feature matrix (samples x numerical features), e.g. the iris dataset's 150 samples x 4 morphological features loaded from the Fisher 1936 source or the bundled Tutorial_Data_01.xlsx via pandas
- Optionally z-score standardized numerical columns (Instant Clue applies normalization/transformation on columns via the context menu)
- Fitted scikit-learn PCA estimator (full-rank, one component per input feature)

## Outputs

- pca_scores_table.csv: one row per sample with PC1-PC4 score columns, the class label column (e.g. species), and an explained_variance_ratio row/field
- Per-component explained-variance ratios (e.g. PC1 ~0.90-0.95 for iris)

## How to apply

Fit a full-rank PCA (all components, e.g. all 4 for the 150-sample x 4-feature iris matrix) with scikit-learn's PCA estimator, which is the library Instant Clue uses for its statistical analysis activities; optionally z-score standardize the numerical columns beforehand so all variables contribute equally to the decomposition. Project the samples onto the principal components and read off the explained_variance_ratio_ attribute to obtain one ratio per component. Write the per-sample PC1-PC4 score columns plus the class label (e.g. iris species) into a single table (pca_scores_table.csv) with the explained_variance_ratio recorded as an accompanying row/field. Validate by checking that the leading component's ratio falls in the expected range (~0.90-0.95 for iris) and that the output has the expected row and column counts; any remaining numeric plausibility judgments are deferred to expert_review. Rationale: Instant Clue exposes PCA (and cluster analysis) via a single drag & drop event for unsupervised pattern discovery, and this skill supplies the variance-accounted-for metric that makes such unsupervised reductions interpretable.

## Related tools

- **scikit-learn** (PCA estimator whose fit on the standardized feature matrix yields the explained_variance_ratio_ per principal component)
- **Instant Clue** (Desktop GUI that exposes PCA via drag & drop onto charts and applies column normalization/transformation via the context menu, the workflow this computation mirrors) — https://github.com/hnolCol/instantclue
- **Python >=3.4** (Runtime for the pandas/scikit-learn script that loads the iris data, fits PCA, and writes pca_scores_table.csv)

## Examples

```
from sklearn.decomposition import PCA; pca = PCA(n_components=4).fit(X); ratios = pca.explained_variance_ratio_; scores = pca.transform(X)
```

## Evaluation signals

- pca_scores_table.csv exists with exactly 150 data rows (one per iris sample) and 4 principal-component score columns
- PC1 explained-variance ratio falls in the expected ~0.90-0.95 range for the iris data
- Ratios are non-negative and sum to ~1 across all 4 full-rank components
- Species label column is carried through unchanged alongside the PC scores
- Any remaining numeric plausibility judgments are routed to expert_review rather than auto-accepted

## Limitations

- The expected explained-variance range (~0.90-0.95 for PC1) is dataset-specific to iris; other datasets need their own plausibility bounds
- Without z-score standardization, features with larger scale dominate the decomposition, distorting the variance split
- Instant Clue's statistical-test panel is still being extended ('we are aiming to extend the panel of statistical tests'), so GUI-exposed dimensionality-reduction options may not cover every estimator variant available in scikit-learn

## Evidence

- [results] PCA drag & drop: [evidence span withheld — anchor status: not_found]
- [results] scikit-learn basis: "Instant Clue offers several functions to establish an estimator for prediction, based on the scikit-learn library"
- [results] column activities: "Activities are applied on columns in the dataset using the context menu and cover basic steps such as sorting, string splitting and replacement, normalization and transformation"
- [intro] PCA in objective list: "Offering a repertoire of statistical tests like ANOVA, regressions, curve fitting and principal component analysis"
- [results] missing-value pre-filter: "if an activity (each test is an activity) cannot handle missing values, the data are automatically filtered before submission to the specified activity"
