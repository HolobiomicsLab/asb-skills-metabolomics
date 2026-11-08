---
name: splsda-feature-selection-metabolomics
description: Use when a samples x metabolites/proteins feature matrix (CSV) with a
  target file mapping each sample to predefined study groups (e.g. the three-group
  targeted metabolomics study ST000284) is available and the goal is supervised classification
  plus embedded feature selection.
when_to_use_negative:
- The analysis goal is unsupervised exploration of overall structure with no group
  labels — PCA is the POMAShiny panel for that.
- Features have not yet been preprocessed in POMAShiny (missing-value imputation,
  normalization and outlier removal), since the multivariate panel expects the cleaned
  matrix.
- No predefined study-group target exists (e.g. two-group ST000336-style binary design
  with no metadata file), so supervised discriminant analysis is not applicable.
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: POMAShiny
  role: web-based workflow whose statistical panel exposes sPLS-DA with tunable components,
    feature count, VIP cut-off and cross-validation; also provides interactive visualization
    of the fitted model
  repo: https://github.com/nutrimetabolomics/POMAShiny
- name: POMA
  role: R/Bioconductor package that implements the multivariate engine (including
    sPLS-DA via mixOmics) behind POMAShiny, enabling the same analysis outside the
    web environment
  repo: https://github.com/nutrimetabolomics/POMA
- name: plotly
  role: renders the interactive sample-score plots for the latent components produced
    by the sPLS-DA fit
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1371/journal.pcbi.1009148
    title: 'POMAShiny: A user-friendly web-based workflow for metabolomics and proteomics
      data analysis'
  source_content_hash: sha256:73b9075e6f5fe3887ffa2afac7b8305d5de2a99efd0e71f1230efc1c83932a92
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/nutrimetabolomics/POMAShiny
  tools:
  - POMA
  - POMAShiny
  - R
  - plotly
  - tidyverse
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 20
---

# splsda-feature-selection-metabolomics

## Summary

Apply sparse Partial Least Squares Discriminant Analysis (sPLS-DA) via the mixOmics-based POMA/POMAShiny workflow to select the metabolite (or protein) features that discriminate predefined study groups in a targeted omics data set, exporting the selected-feature list alongside sample scores on the first latent components.

## When to use

Use when a samples x metabolites/proteins feature matrix (CSV) with a target file mapping each sample to predefined study groups (e.g. the three-group targeted metabolomics study ST000284) is available and the goal is supervised classification plus embedded feature selection.

## When NOT to use

- The analysis goal is unsupervised exploration of overall structure with no group labels — PCA is the POMAShiny panel for that.
- Features have not yet been preprocessed in POMAShiny (missing-value imputation, normalization and outlier removal), since the multivariate panel expects the cleaned matrix.
- No predefined study-group target exists (e.g. two-group ST000336-style binary design with no metadata file), so supervised discriminant analysis is not applicable.

## Inputs

- samples x metabolites (or proteins) feature matrix CSV with one row per sample and one column per feature
- target (metadata) CSV mapping each sample to its predefined study group
- user-tuned parameters: number of latent components, number of features to select, VIP cut-off, cross-validation method (LOO or k-fold)

## Outputs

- sPLS-DA sample score table CSV (one row per sample; columns for the first two latent dimensions plus group label)
- sPLS-DA selected-feature table CSV (non-empty list of discriminative metabolites/proteins)
- interactive/plotly visualizations of sample separation on the latent components

## How to apply

Load the preprocessed feature matrix and its target/metadata CSV into POMAShiny (or call POMA directly from R), then fit a sparse PLS-DA model on the group-labeled data, tuning the number of components, the number of features to select, the VIP cut-off, and the cross-validation scheme (leave-one-out or k-fold). sPLS-DA performs embedded feature selection while classifying, so the discriminative metabolite subset is read directly from the fitted model. Export the sample scores on the first two latent components and the selected-feature table as CSVs; the selected-feature table should be a non-empty metabolite list and the score tables should show separation of the three study groups. This is the supervised branch of the multivariate panel, distinct from the unsupervised PCA, and complements the univariate and machine-learning panels offered by POMAShiny.

## Related tools

- **POMAShiny** (web-based workflow whose statistical panel exposes sPLS-DA with tunable components, feature count, VIP cut-off and cross-validation; also provides interactive visualization of the fitted model) — https://github.com/nutrimetabolomics/POMAShiny
- **POMA** (R/Bioconductor package that implements the multivariate engine (including sPLS-DA via mixOmics) behind POMAShiny, enabling the same analysis outside the web environment) — https://github.com/nutrimetabolomics/POMA
- **plotly** (renders the interactive sample-score plots for the latent components produced by the sPLS-DA fit)

## Examples

```
R: library(POMA); res <- Pomix::splsda(X = features_matrix, Y = target$group, ncomp = 2, keepX = c(5, 5))
```

## Evaluation signals

- Selected-feature table is non-empty and contains metabolite identifiers drawn from the input feature matrix columns.
- Score table contains exactly one row per sample with columns for the first two latent dimensions and the group label.
- Fitted model separates the three study groups on the first two latent components (expert review confirms the expected separation).
- Cross-validation prediction performance is reported for the chosen LOO or k-fold scheme.
- Number of selected features is consistent with the user-specified feature count / VIP cut-off, not larger than the input feature dimensionality.

## Limitations

- sPLS-DA is only as good as the preprocessing upstream: POMAShiny expects missing-value imputation, normalization and outlier removal to have been done in earlier panels (default 20% missing-per-group threshold for feature removal).
- POMAShiny requires exactly two CSV inputs (target and features); other formats must be converted before use.
- The web interface pins bs4Dash 0.5.0 and is not compatible with newer versions of that package, which can break local deployment.
- POMAShiny is an open-source, user-friendly interface but no benchmarking or quantitative evaluation against other tools is reported in the article.

## Evidence

- [results] POMAShiny implements PCA, PLS-DA and sPLS-DA through the mixOmics Bioconductor package, with tunable parameters such as number of components, number of features to select (sPLS-DA), VIP cut-off and cross-validation method (LOO or k-fold).: [evidence span withheld — anchor status: not_found]
- [intro] including univariate statistical methods, multivariate and dimension reduction methods, feature selection methods, regularized regression analysis approaches, machine learning classification: "including univariate statistical methods, multivariate and dimension reduction methods, feature selection methods, regularized regression analysis approaches, machine learning classification"
- [results] POMAShiny requires two comma-separated value (CSV) files as an input: the target (or metadata) file and the features file: "POMAShiny requires two comma-separated value (CSV) files as an input: the target (or metadata) file and the features file"
- [results] users are able to remove features of the data with more than a specific percentage of missing values in all the study groups (20% allowed by default): "users are able to remove features of the data with more than a specific percentage of missing values in all the study groups (20% allowed by default)"
- [results] The example data set ST000284 consists of a targeted metabolomics three-group study: "The example data set ST000284 consists of a targeted metabolomics three-group study"
- [readme] This tool is based on the POMA R/Bioconductor package, which increases the reproducibility and flexibility of the analysis outside the web environment.: "This tool is based on the POMA R/Bioconductor package, which increases the reproducibility and flexibility of the analysis outside the web environment."
