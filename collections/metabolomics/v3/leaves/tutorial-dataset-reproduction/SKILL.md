---
name: tutorial-dataset-reproduction
description: Use when a task asks to reproduce a bundled tutorial or canonical benchmark
  dataset (e.g. iris from Fisher 1936) with a published tool's statistical workflow,
  such as unsupervised PCA on small feature tables.
when_to_use_negative:
- The goal is interactive visual exploration inside the Instant Clue GUI itself rather
  than a scriptable, file-based reproduction of its PCA activity.
- The input is already a projected/reduced feature table (e.g. existing PC scores)
  rather than raw morphological or omics feature columns.
- Supervised analysis is needed — PCA is an unsupervised dimensional-reduction activity,
  not a classifier.
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3316
tools:
- name: Instant Clue
  role: reference implementation whose drag & drop PCA activity this reproduction
    mirrors; executables and source available on GitHub
  repo: https://github.com/hnolCol/instantclue
- name: scikit-learn
  role: PCA estimator library that Instant Clue uses for its statistical analysis
    activities; used here to fit the full-rank 4-component model
- name: Python >=3.4
  role: runtime for the reproduction script, matching the minimum Python version Instant
    Clue requires
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41598-018-31154-6
    title: 'Instant Clue: A Software Suite for Interactive Data Visualization and
      Analysis'
  source_content_hash: sha256:e21c3cdf9098501e1fad8d470c7011dd93f1aad77ed1e943dc64491a238f085b
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
  - http://edamontology.org/topic_3316
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# tutorial-dataset-reproduction

## Summary

Reproduces a documented tutorial analysis — here the Fisher 1936 iris dataset (150 samples × 4 morphological features, 3 species) — end-to-end in code, mirroring how Instant Clue performs drag & drop principal component analysis, and writes a validated scores table (pca_scores_table.csv).

## When to use

Use when a task asks to reproduce a bundled tutorial or canonical benchmark dataset (e.g. iris from Fisher 1936) with a published tool's statistical workflow, such as unsupervised PCA on small feature tables.

## When NOT to use

- The goal is interactive visual exploration inside the Instant Clue GUI itself rather than a scriptable, file-based reproduction of its PCA activity.
- The input is already a projected/reduced feature table (e.g. existing PC scores) rather than raw morphological or omics feature columns.
- Supervised analysis is needed — PCA is an unsupervised dimensional-reduction activity, not a classifier.

## Inputs

- iris dataset (150 samples × 4 morphological features, 3 species) from the public Fisher 1936 source or the bundled tutorial copy
- Excel/tab-delimited text (.txt)/csv/.xml/.gz/.zip file ingested via pandas

## Outputs

- pca_scores_table.csv with exactly 150 data rows, PC1–PC4 score columns, species label, and an explained_variance_ratio row/field
- per-component explained-variance ratios

## How to apply

Load the iris dataset (150 samples × 4 morphological features, 3 species) from the public Fisher 1936 source or the bundled tutorial copy using pandas. Optionally z-score standardize the four numerical features via the context-menu-style normalization Instant Clue applies on columns, so all variables contribute equally to the decomposition. Fit a full-rank PCA (all 4 components) with scikit-learn's PCA estimator — the library Instant Clue uses for its statistical analysis activities — then project samples onto PC1–PC4 and record per-component explained-variance ratios. Write a single table with one row per iris sample, columns for PC1–PC4 scores plus the species label and an explained_variance_ratio field, saved as pca_scores_table.csv. Conceptually this mirrors how in Instant Clue principal component analysis is initialized by a single drag & drop event to identify underlying patterns unsupervised, without writing code.

## Related tools

- **Instant Clue** (reference implementation whose drag & drop PCA activity this reproduction mirrors; executables and source available on GitHub) — https://github.com/hnolCol/instantclue
- **scikit-learn** (PCA estimator library that Instant Clue uses for its statistical analysis activities; used here to fit the full-rank 4-component model)
- **Python >=3.4** (runtime for the reproduction script, matching the minimum Python version Instant Clue requires)

## Examples

```
from sklearn.decomposition import PCA; from sklearn.preprocessing import StandardScaler; Z = StandardScaler().fit_transform(df.iloc[:, :4]); scores = PCA(n_components=4).fit_transform(Z)
```

## Evaluation signals

- pca_scores_table.csv exists with exactly 150 data rows and 4 principal-component score columns
- PC1 explained-variance ratio falls in the expected range ~0.90–0.95 for the iris data
- Output table carries the species label per row and an explained_variance_ratio field
- Component count equals full rank (4) so per-component ratios sum to ~1.0
- Any remaining numeric plausibility judgments are deferred to expert_review

## Limitations

- PC1 explained-variance varies with pre-processing: standardization is optional, so the ~0.90–0.95 range assumes the standardized iris data; unstandardized features shift ratios.
- Instant Clue's panel of statistical tests and additional activities are still being extended, so not every published-planned activity can be reproduced.
- pyComBat/combat is not in the requirements table due to specific pandas/numpy versioning requirements — install it first and ignore the numpy/pandas version-mismatch warning when setting up the source environment.

## Evidence

- [abstract] PCA initialized by drag & drop, unsupervised, no code: "Instant Clue combines the power of visual and statistical analytics using a straight forward drag & drop approach making the software highly intuitive"
- [results] scikit-learn underlies estimators: "Instant Clue offers several functions to establish an estimator for prediction, based on the scikit-learn library"
- [results] column activities include normalization: "Activities are applied on columns in the dataset using the context menu and cover basic steps such as sorting, string splitting and replacement, normalization and transformation"
- [results] supported input file types: "Data can be uploaded from various file types including Excel, tab delimited text (.txt) and csv-files (.csv), Extensible Markup Language (.xml) files as well as compressed files (.gz, .zip)"
- [intro] Python >=3.4 requirement: "the python source code is also freely available making Instant Clue editable and available on any platform with a working Python version (>=3.4)"
- [methods] combat not in requirements: "the package [combat](https://github.com/epigenelabs/pyComBat/tree/master/combat) is not in the requirements table"
- [discussion] planned extension of statistical tests: "we are aiming to extend the panel of statistical tests and will add further activities that will be beneficial for systematic data interpretation"
