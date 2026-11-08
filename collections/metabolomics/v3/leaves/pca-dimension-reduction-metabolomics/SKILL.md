---
name: pca-dimension-reduction-metabolomics
description: Use when the analysis goal is unsupervised exploration of a samples x
  metabolites (or proteins) matrix — e.g. the ST000284 targeted metabolomics three-group
  study — to reduce its dimensions and visualize sample separation before supervised
  modelling.
when_to_use_negative:
- The analysis goal is supervised class discrimination or feature selection — use
  PLS-DA or sPLS-DA instead, since PCA is unsupervised.
- The input is raw vendor spectral data (mzML etc.); the article directs such processing
  to the RforMassSpectrometry packages before migrating to POMA for statistical analysis.
- The feature matrix has not yet passed missing-value imputation and normalization
  — run the POMAShiny preprocessing panel first rather than applying PCA directly.
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0092
tools:
- name: POMAShiny
  role: Web front-end hosting the PCA dimension-reduction panel for metabolomics/proteomics
    data; supports interactive PCA visualization.
  repo: https://github.com/nutrimetabolomics/POMAShiny
- name: POMA
  role: R/Bioconductor package providing the multivariate PCA engine used by POMAShiny,
    enabling reproducible PCA outside the web environment.
  repo: https://github.com/nutrimetabolomics/POMA
- name: plotly
  role: Interactive plotting library used by POMAShiny to render the two-dimensional
    PCA score plots.
- name: tidyverse
  role: R package family used in POMAShiny's back-end for data manipulation and visualization
    prior to PCA.
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1371/journal.pcbi.1009148
    title: 'POMAShiny: A user-friendly web-based workflow for metabolomics and proteomics
      data analysis'
  source_content_hash: sha256:ff44a451c4ed03744ca6912ae8469ca1dd790721e9fe4e1776377a747fb46f1a
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
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8125
n_claims: 16
---

# pca-dimension-reduction-metabolomics

## Summary

Apply PCA to scaled metabolomics or proteomics feature matrices for unsupervised sample-dimension reduction and two-dimensional sample-separation visualization, as implemented in POMAShiny via the POMA R/Bioconductor package.

## When to use

Use when the analysis goal is unsupervised exploration of a samples x metabolites (or proteins) matrix — e.g. the ST000284 targeted metabolomics three-group study — to reduce its dimensions and visualize sample separation before supervised modelling.

## When NOT to use

- The analysis goal is supervised class discrimination or feature selection — use PLS-DA or sPLS-DA instead, since PCA is unsupervised.
- The input is raw vendor spectral data (mzML etc.); the article directs such processing to the RforMassSpectrometry packages before migrating to POMA for statistical analysis.
- The feature matrix has not yet passed missing-value imputation and normalization — run the POMAShiny preprocessing panel first rather than applying PCA directly.

## Inputs

- Preprocessed samples x metabolites feature matrix (CSV) with samples as rows and metabolites as columns
- Target/metadata CSV mapping each sample to its study-group label (e.g. three-group ST000284 layout)
- Scale/normalization choice from POMAShiny's normalization panel (autoscaling, level scaling, log scaling, log transformation, vast scaling, log pareto scaling)

## Outputs

- PCA score table CSV: one row per sample with columns for the first two principal-component scores plus the group label
- Two-dimensional sample-separation plot (interactive, e.g. via plotly in POMAShiny)

## How to apply

Load the preprocessed feature matrix (samples as rows, metabolites as columns) together with the target/metadata CSV that maps each sample to its study group; in the article's ST000284 example this is a targeted metabolomics three-group study. Run PCA on the scaled feature matrix using POMA, which wraps the multivariate modelling engine used by POMAShiny's statistical-analysis panel. Scaling should follow POMAShiny's normalization options (autoscaling, level scaling, log scaling, log transformation, vast scaling, log pareto scaling), and the feature matrix should first pass through the missing-value imputation panel (zero, half-minimum, median, mean, minimum or k-nearest-neighbours imputation), with features carrying more than 20% missing values removed by default. Extract each sample's scores on the first two principal components and write the score table (one row per sample, columns for the two dimensions plus the group label) as CSV; in the POMAShiny GUI this is the EDA/unsupervised view that 'highly recommended to perform an EDA before any statistical analysis'. Validate that the score file contains one row per sample and that the first two dimensions give the expected three-group separation reviewed by an expert.

## Related tools

- **POMAShiny** (Web front-end hosting the PCA dimension-reduction panel for metabolomics/proteomics data; supports interactive PCA visualization.) — https://github.com/nutrimetabolomics/POMAShiny
- **POMA** (R/Bioconductor package providing the multivariate PCA engine used by POMAShiny, enabling reproducible PCA outside the web environment.) — https://github.com/nutrimetabolomics/POMA
- **plotly** (Interactive plotting library used by POMAShiny to render the two-dimensional PCA score plots.)
- **tidyverse** (R package family used in POMAShiny's back-end for data manipulation and visualization prior to PCA.)

## Examples

```
shiny::runApp(appDir = "app")
```

## Evaluation signals

- The PCA score CSV contains exactly one row per sample from the input feature matrix, with columns for the first two dimensions and the study-group label.
- The first two principal components reproduce the expected three-group separation on the ST000284 example, as confirmed by expert review of the score plot.
- Features carrying more than the default 20% missing values were removed before PCA, and the remaining matrix was normalized using one of the six POMAShiny scaling methods.
- The score table is derived from the scaled (not raw) feature matrix, consistent with the chosen normalization method.
- Re-running the workflow yields a stable PCA score table, reflecting the POMA-based reproducibility emphasized by the article.

## Limitations

- PCA is unsupervised and cannot perform class discrimination or embedded feature selection — for that use PLS-DA or sPLS-DA.
- The POMAShiny article reports no benchmarking or quantitative evaluation of its PCA results against other tools.
- Feature-removal and imputation defaults (20% missing-value threshold; zero/half-minimum/median/mean/minimum/kNN imputation) shape the input to PCA, so different preprocessing choices will change the PCA outcome.
- No specific software version number is stated for POMAShiny or POMA in the article.

## Evidence

- [results] POMAShiny implements PCA, PLS-DA and sPLS-DA through the mixOmics Bioconductor package: [evidence span withheld — anchor status: not_found]
- [results] unsupervised exploratory analysis recommended before statistical analysis: "it is highly recommended to perform an EDA before any statistical analysis"
- [results] six available normalization methods: "The methods available for this purpose are autoscaling, level scaling, log scaling, log transformation, vast scaling, and log pareto scaling"
- [results] default missing-value threshold of 20%: "remove features of the data with more than a specific percentage of missing values in all the study groups (20% allowed by default)"
- [results] POMAShiny requires two CSV input files: "POMAShiny requires two comma-separated value (CSV) files as an input: the target (or metadata) file and the features file"
- [readme] POMAShiny is based on POMA R/Bioconductor package for reproducibility outside the web environment: "This tool is based on the **POMA** R/Bioconductor package, which increases the reproducibility and flexibility of the analysis outside the web environment."
- [intro] POMAShiny offers multivariate and dimension reduction methods: "including univariate statistical methods, multivariate and dimension reduction methods, feature selection methods, regularized regression analysis approaches, machine learning classification"
