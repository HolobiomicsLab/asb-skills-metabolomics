---
name: plsda-supervised-classification-metabolomics
description: Use when a preprocessed samples-by-features metabolomics/proteomics matrix
  with group labels (e.g. a two- or three-group study like ST000284) must be classified
  by group and its discriminative features identified via PLS-DA/sPLS-DA.
when_to_use_negative:
- When the goal is unsupervised exploration without predefined group labels — use
  PCA instead, which POMAShiny also provides through mixOmics
- When the input matrix still contains missing values or is not yet imputed/normalized
  — POMAShiny requires preprocessing (missing-value imputation and normalization)
  before the statistical analysis panel
- When the analysis is a pure regression task rather than classification into predefined
  study groups
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: POMAShiny
  role: Web application exposing the PLS-DA/sPLS-DA classification workflow with tunable
    components, feature counts, VIP cut-off and cross-validation panels
  repo: https://github.com/nutrimetabolomics/POMAShiny
- name: POMA
  role: R/Bioconductor back-end that implements the multivariate classification, enabling
    the same PLS-DA/sPLS-DA analysis outside the web environment for reproducibility
  repo: https://github.com/nutrimetabolomics/POMA
- name: mixOmics
  role: Underlying engine invoked by POMA for PLS-DA and sPLS-DA modeling
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1371/journal.pcbi.1009148
    title: 'POMAShiny: A user-friendly web-based workflow for metabolomics and proteomics
      data analysis'
  source_content_hash: sha256:b4c206a38561c817c6af4b8f2f57ba4e3940c0dc9ff8c602c84a8cd477251440
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/nutrimetabolomics/POMAShiny
  tools:
  - POMA
  - POMAShiny
  - R
  - mixOmics
  - plotly
  - tidyverse
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.84
n_claims: 25
---

# plsda-supervised-classification-metabolomics

## Summary

Apply partial least squares discriminant analysis (PLS-DA) and its sparse variant (sPLS-DA) to metabolomics or proteomics feature matrices to classify samples into predefined study groups and select discriminative features, as implemented in POMAShiny via the POMA R/Bioconductor package (which wraps mixOmics).

## When to use

Use when a preprocessed samples-by-features metabolomics/proteomics matrix with group labels (e.g. a two- or three-group study like ST000284) must be classified by group and its discriminative features identified via PLS-DA/sPLS-DA.

## When NOT to use

- When the goal is unsupervised exploration without predefined group labels — use PCA instead, which POMAShiny also provides through mixOmics
- When the input matrix still contains missing values or is not yet imputed/normalized — POMAShiny requires preprocessing (missing-value imputation and normalization) before the statistical analysis panel
- When the analysis is a pure regression task rather than classification into predefined study groups

## Inputs

- Preprocessed samples-by-features CSV matrix (metabolomics or proteomics feature intensities, imputed and normalized, e.g. autoscaled)
- Target/metadata CSV mapping each sample to its predefined study group (e.g. the three-group ST000284 study or two-group ST000336 study)

## Outputs

- PLS-DA sample score table with coordinates on the first two latent components per sample plus group label
- sPLS-DA selected-feature table: the subset of metabolites identified as discriminative between the study groups
- Two-dimensional PLS-DA/sPLS-DA score plots visualizing group separation

## How to apply

Load the preprocessed feature matrix (samples as rows, metabolites/proteins as columns) together with the target/metadata CSV mapping each sample to its study group. Fit a PLS-DA model on the normalized data with the group variable as the supervised response, extracting sample scores on the first two latent components for two-dimensional visualization of group separation. For embedded feature selection, fit a sparse PLS-DA (sPLS-DA) model and tune the number of components, the number of features selected per component, the VIP cut-off, and the cross-validation scheme (leave-one-out or k-fold). Export the sPLS-DA-selected metabolite list as the discriminative feature set. Validate that the score tables show the expected separation of the predefined groups (e.g. the three groups of ST000284) and that the selected-feature table is a non-empty metabolite list.

## Related tools

- **POMAShiny** (Web application exposing the PLS-DA/sPLS-DA classification workflow with tunable components, feature counts, VIP cut-off and cross-validation panels) — https://github.com/nutrimetabolomics/POMAShiny
- **POMA** (R/Bioconductor back-end that implements the multivariate classification, enabling the same PLS-DA/sPLS-DA analysis outside the web environment for reproducibility) — https://github.com/nutrimetabolomics/POMA
- **mixOmics** (Underlying engine invoked by POMA for PLS-DA and sPLS-DA modeling)

## Examples

```
Rscript -e 'library(POMA); pomadata(target_file, features_file); PomaUnivariate(); PomaMultivariate(method = "splsda", ncomp = 2, keepN = 10, cv = "loo")'
```

## Evaluation signals

- Score table row count equals number of input samples, with columns for the first two latent dimensions and the group label
- PLS-DA score plot shows visually separated clusters corresponding to the predefined study groups (e.g. the three groups of ST000284)
- sPLS-DA selected-feature table is a non-empty list of metabolites
- Cross-validation (LOO or k-fold) classification performance is reported with the chosen number of components and features
- Feature selection is consistent across repeated runs at fixed sPLS-DA parameters (number of components, keepN, VIP cut-off)

## Limitations

- Supervised model performance must be assessed by cross-validation (LOO or k-fold); POMAShiny provides this, but classification accuracy is not a substitute for independent validation
- sPLS-DA feature selection depends on user-chosen parameters (number of components, number of features, VIP cut-off) and different settings yield different feature sets
- POMAShiny requires two specific CSV inputs (features file and target/metadata file) in the expected format; other formats must first be converted

## Evidence

- [methods] POMAShiny implements PCA, PLS-DA and sPLS-DA through the mixOmics Bioconductor package, with tunable parameters such as number of components, number of features to select (sPLS-DA), VIP cut-off and cross-validation method (LOO or k-fold).: [evidence span withheld — anchor status: not_found]
- [intro] including univariate statistical methods, multivariate and dimension reduction methods, feature selection methods, regularized regression analysis approaches, machine learning classification: "including univariate statistical methods, multivariate and dimension reduction methods, feature selection methods, regularized regression analysis approaches, machine learning classification"
- [results] The example data set ST000284 consists of a targeted metabolomics three-group study: "The example data set ST000284 consists of a targeted metabolomics three-group study"
- [readme] it is based on the POMA R/Bioconductor package, which increases the reproducibility and flexibility of analyses outside the web environment: "This tool is based on the [**POMA**](http://pcastellanoescuder.github.io/POMA/) R/Bioconductor package, which increases the reproducibility and flexibility of the analysis outside the web environment"
