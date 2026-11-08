---
name: metabolomics-multivariate-score-extraction
description: Use when a preprocessed metabolomics/proteomics feature table (samples
  x metabolites, e.g. from ST000284-style targeted studies) must be reduced or classified,
  and per-sample PCA/PLS-DA scores plus a discriminative metabolite list are the required
  deliverables.
when_to_use_negative:
- Input is still raw spectral data (mzML/NetCDF or similar) that has not yet been
  imputed, normalized or scaled — POMAShiny's preprocessing panel (20% missing-value
  threshold, one of six imputation and six normalization methods, outlier removal)
  must run first.
- The analysis goal is purely univariate (e.g. fold-change or t-test per metabolite)
  with no need for dimension reduction or classification.
- Only two-group comparison without any interest in feature selection is required
  — simpler POMA tests may suffice and sPLS-DA tuning overhead is unnecessary.
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0121
tools:
- name: POMA
  role: R/Bioconductor package that implements PCA, PLS-DA and sPLS-DA (via mixOmics)
    for dimension reduction and classification of the preprocessed feature matrix
  repo: https://github.com/nutrimetabolomics/POMA
- name: POMAShiny
  role: Web interface that exposes the same PCA/PLS-DA/sPLS-DA analyses with tunable
    components, feature counts, VIP cut-off and CV method (LOO or k-fold); serves
    as the reference implementation to reproduce via POMA
  repo: https://github.com/nutrimetabolomics/POMAShiny
- name: tidyverse
  role: Used within POMAShiny's back-end for data manipulation of the target and feature
    tables before and after multivariate fitting
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1371/journal.pcbi.1009148
    title: 'POMAShiny: A user-friendly web-based workflow for metabolomics and proteomics
      data analysis'
  source_content_hash: sha256:56045dd1e96a91596ca6c3ea58dea10de2e17306fe5936cf27372f2c0b82a8dc
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/nutrimetabolomics/POMA
  tools:
  - POMA
  - POMAShiny
  - R
  - plotly
  - tidyverse
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9629629629629629
n_claims: 27
---

# metabolomics-multivariate-score-extraction

## Summary

Run PCA, PLS-DA and sPLS-DA on a preprocessed metabolomics/proteomics feature matrix via the POMA R/Bioconductor package (as used by POMAShiny), and export per-sample scores on the first two components plus the sPLS-DA selected-feature list.

## When to use

Use when a preprocessed metabolomics/proteomics feature table (samples x metabolites, e.g. from ST000284-style targeted studies) must be reduced or classified, and per-sample PCA/PLS-DA scores plus a discriminative metabolite list are the required deliverables.

## When NOT to use

- Input is still raw spectral data (mzML/NetCDF or similar) that has not yet been imputed, normalized or scaled — POMAShiny's preprocessing panel (20% missing-value threshold, one of six imputation and six normalization methods, outlier removal) must run first.
- The analysis goal is purely univariate (e.g. fold-change or t-test per metabolite) with no need for dimension reduction or classification.
- Only two-group comparison without any interest in feature selection is required — simpler POMA tests may suffice and sPLS-DA tuning overhead is unnecessary.

## Inputs

- Preprocessed samples x metabolites feature matrix (CSV)
- Target/metadata CSV mapping each sample to its study group

## Outputs

- PCA score table: one row per sample with scores on the first two dimensions plus group label (CSV)
- PLS-DA score table: one row per sample with scores on the first two latent components plus group label (CSV)
- sPLS-DA selected-feature table: non-empty list of metabolites selected as discriminative (CSV)

## How to apply

Load the preprocessed feature matrix (samples x metabolites, normalized/imputed via POMAShiny's preprocessing panel with autoscaling or one of its six scaling options) together with the target/metadata CSV mapping each sample to its study group. Fit PCA on the scaled matrix to obtain sample scores on the first two dimensions, then fit a PLS-DA classification model using the three-group (or two-group) target variable and extract latent-component scores. For feature selection, fit a sparse PLS-DA model with tunable parameters — number of components, number of features to select, VIP cut-off, and cross-validation method (LOO or k-fold) as configured in POMAShiny — and export the subset of metabolites selected as discriminative. The rationale is that POMAShiny implements these multivariate and dimension-reduction methods through the mixOmics-based POMA R/Bioconductor package, so running them via POMA outside the web environment preserves reproducibility and flexibility while matching the web-tool outputs.

## Related tools

- **POMA** (R/Bioconductor package that implements PCA, PLS-DA and sPLS-DA (via mixOmics) for dimension reduction and classification of the preprocessed feature matrix) — https://github.com/nutrimetabolomics/POMA
- **POMAShiny** (Web interface that exposes the same PCA/PLS-DA/sPLS-DA analyses with tunable components, feature counts, VIP cut-off and CV method (LOO or k-fold); serves as the reference implementation to reproduce via POMA) — https://github.com/nutrimetabolomics/POMAShiny
- **tidyverse** (Used within POMAShiny's back-end for data manipulation of the target and feature tables before and after multivariate fitting)

## Examples

```
Rscript -e "library(POMA); scores <- POMA::poma_pca(X, Y); write.csv(scores, 'pca_scores.csv')"
```

## Evaluation signals

- Each score file contains exactly one row per sample with columns for the first two dimensions and the group label.
- The sPLS-DA selected-feature table is non-empty and its metabolites match the feature names in the input matrix.
- PCA and PLS-DA score tables reproduce the expected three-group separation seen in the POMAShiny ST000284 example.
- Imputation, normalization and outlier parameters match those configured in the POMAShiny preprocessing panel (e.g. 20% missing-value threshold, chosen scaling method).
- Cross-validation choice (LOO or k-fold) and sPLS-DA tuning parameters (components, selected features, VIP cut-off) are recorded and reproducible via the POMA package outside the web app.

## Limitations

- POMAShiny was built with bs4Dash version 0.5.0 and is not compatible with newer versions due to substantial breaking changes in the API; running locally requires pinning the old bs4Dash or using the Docker image pcastellanoescuder/pomashiny:v.1.1.14.
- No benchmarking or quantitative evaluation of POMAShiny against other tools is reported, so score outputs should be sanity-checked against the web app rather than assumed superior.
- The article states no specific software version number for POMAShiny or POMA, so exact package versions must be captured separately for reproducibility.

## Evidence

- [results] PCA, PLS-DA and sPLS-DA through the mixOmics Bioconductor package, with tunable parameters such as number of components, number of features to select (sPLS-DA), VIP cut-off and cross-validation method (LOO or k-fold): [evidence span withheld — anchor status: not_found]
- [abstract] it is based on the POMA R/Bioconductor package, which increases the reproducibility and flexibility of analyses outside the web environment: "it is based on the POMA R/Bioconductor package, which increases the reproducibility and flexibility of analyses outside the web environment"
- [results] The example data set ST000284 consists of a targeted metabolomics three-group study: "The example data set ST000284 consists of a targeted metabolomics three-group study"
- [results] users are able to remove features of the data with more than a specific percentage of missing values in all the study groups (20% allowed by default): "users are able to remove features of the data with more than a specific percentage of missing values in all the study groups (20% allowed by default)"
- [readme] NOTE: POMAShiny was built with `bs4Dash` version 0.5.0 and is not compatible with newer versions of `bs4Dash` due to substantial breaking changes in the API.: "NOTE: *POMAShiny was built with `bs4Dash` version 0.5.0 and is not compatible with newer versions of `bs4Dash` due to substantial breaking changes in the API."
