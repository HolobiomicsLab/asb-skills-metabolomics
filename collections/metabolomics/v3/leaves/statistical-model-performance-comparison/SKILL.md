---
name: statistical-model-performance-comparison
description: Use when a log2-transformed metabolomics matrix (samples in rows, features
  in columns, batch labels in the first column) shows batch clustering (e.g. PCA by
  batch order) and several batch-correction models must be compared to pick the -fitting
  one.
when_to_use_negative:
- Datasets with more than 2000 features — the README states dbnormSCORE 'is suggested
  for less than 2000 features (variables) for better computational speed'
- Data that is not normalized/log2-transformed — the README requires data be 'normalized
  and scaled on the log2' before upload
- Datasets without batch-level labels in the first column, since the score is based
  on regression of metabolite intensity on batch level
edam_operation: null
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0203
tools:
- name: dbnorm
  role: R package whose dbnormSCORE, dbnormBer, dbnormBagging, dbnormPcom and dbnormNPcom
    functions compute adj-R² batch-dependency scores and apply the correction models
    being compared
  repo: https://github.com/NBDZ/dbnorm
- name: ComBat
  role: sva empirical Bayes location/scale batch-correction model applied in parametric
    (dbnormPcom) and nonparametric (dbnormNPcom) settings and scored against alternatives
  repo: https://github.com/NBDZ/dbnorm
- name: ber
  role: Linear-fitting L/S batch-correction model (and its bagging variant with n=150
    bootstrap samples) applied via dbnormBer/dbnormBagging and scored
  repo: https://github.com/NBDZ/dbnorm
- name: R
  role: Environment in which dbnormSCORE and the correction functions are run on CSV
    input
  repo: https://github.com/NBDZ/dbnorm
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41598-021-84824-3
    title: DBnorm as an R package for the comparison and selection of appropriate
      statistical methods for batch effect correction in metabolomic studies
  source_content_hash: sha256:33d9df91922b9610cb7520fd3311a944595d562abb6d78427acc7afc6b7e00eb
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/NBDZ/dbnorm
  tools:
  - ComBat
  - R
  - ber
  - dbnorm
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6875
n_claims: 16
---

# statistical-model-performance-comparison

## Summary

Scores and ranks batch-effect correction models (ber, bagging ber, parametric and nonparametric ComBat, lowess) by regressing each metabolite's log2-transformed intensity on batch level and reporting the maximum adjusted-R² (0–1) per model, so the model with the lowest residual batch-associated variability can be selected.

## When to use

Use when a log2-transformed metabolomics matrix (samples in rows, features in columns, batch labels in the first column) shows batch clustering (e.g. PCA by batch order) and several batch-correction models must be compared to pick the best-fitting one.

## When NOT to use

- Datasets with more than 2000 features — the README states dbnormSCORE 'is suggested for less than 2000 features (variables) for better computational speed'
- Data that is not normalized/log2-transformed — the README requires data be 'normalized and scaled on the log2' before upload
- Datasets without batch-level labels in the first column, since the score is based on regression of metabolite intensity on batch level

## Inputs

- CSV matrix of log2-transformed metabolite intensities (independent experiments in rows, features in columns) with batch levels in the first column
- Batch label vector (first column of the CSV)
- Optionally, corrected datasets from dbnormBer, dbnormBagging, dbnormPcom, dbnormNPcom

## Outputs

- Score table reporting the maximum adj-R² per correction model (raw and corrected)
- Correlation plot and score plot compiled into a PDF saved in the working directory
- CSV files (in a temporary Rtmpe folder) containing the two-vector adj-R² matrix per model

## How to apply

Load the CSV (read.csv with header=TRUE, row.names=1) and run dbnormSCORE(data), which computes, per metabolite, the regression of intensity on batch level in raw and corrected data and extracts the adjusted coefficient of determination (adj-R2, bounded 0–1); the score is the maximum adj-R² per model. Correct the data beforehand or within the workflow with dbnormBer/dbnormBagging (partial bagging, n=150 bootstrap samples), dbnormPcom (parametric ComBat via sva empirical Bayes L/S estimation), and dbnormNPcom (nonparametric ComBat). Collect the score table across raw plus corrected datasets and select the model with the lowest maximum adj-R² — in the SKIPOGH benchmark (1079 samples, 239 metabolites, 11 batches) ber and parametric ComBat fell below 0.01 while lowess left 0.78 and nonparametric ComBat 0.60. Outputs are correlation and score plots compiled into a PDF plus CSV files of per-metabolite adj-R² and the score table; the function is recommended for fewer than 2000 features for computational speed.

## Related tools

- **dbnorm** (R package whose dbnormSCORE, dbnormBer, dbnormBagging, dbnormPcom and dbnormNPcom functions compute adj-R² batch-dependency scores and apply the correction models being compared) — https://github.com/NBDZ/dbnorm
- **ComBat** (sva empirical Bayes location/scale batch-correction model applied in parametric (dbnormPcom) and nonparametric (dbnormNPcom) settings and scored against alternatives) — https://github.com/NBDZ/dbnorm
- **ber** (Linear-fitting L/S batch-correction model (and its bagging variant with n=150 bootstrap samples) applied via dbnormBer/dbnormBagging and scored) — https://github.com/NBDZ/dbnorm
- **R** (Environment in which dbnormSCORE and the correction functions are run on CSV input) — https://github.com/NBDZ/dbnorm

## Examples

```
data<-read.csv("path/to/mydata.csv",sep=",",header=T,row.names=1); library(dbnorm); dbnormSCORE(data)
```

## Evaluation signals

- A score table exists covering raw plus ber, bagging ber, parametric ComBat, nonparametric ComBat (and lowess) datasets, with every score on the 0–1 adj-R² scale
- On the SKIPOGH reference dataset the reported ranges reproduce: raw and lowess maximum adj-R² up to 0.78, nonparametric ComBat up to 0.60, and ber and parametric ComBat below 0.01
- Correlation plot and score plot PDF is generated in the working directory and CSV outputs are written to the Rtmpe temporary folder
- PCA of corrected data no longer clusters samples by batch order, whereas the raw-data PCA did cluster by batch

## Limitations

- dbnormSCORE is recommended for fewer than 2000 features (variables) for better computational speed
- dbnormSCORE quantifies removal of batch-associated variability but does not by itself confirm biological signal preservation; the article complemented it with creatinine correlation (r rising from 0.18 raw to 0.61 corrected) and CKD-EPI regression checks
- Bagging model uses partial bagging with n=150 bootstrap samples, a fixed setting not tuned per dataset
- Exact software versions of dbnorm, ComBat and ber are not specified

## Evidence

- [methods] This function gives a quick notification about the performance of the statistical models: "This function gives a quick notification about the performance of the statistical models"
- [methods] dbnormSCORE: "the adjusted coefficient of determination or Adjusted R-Squared is calculated for each variable estimated in a regression model for its dependency to the batch level in the raw data and treated data"
- [abstract] score range: "This score, which is denoted by adjusted- R squared (adj-R2), ranges from 0 to 1"
- [results] ber/ComBat performance: "the datasets corrected by ber and parametric Com Bat functions presented a similar performance in this study, with a maximum variability associated with the batch < 0.01 (1%)"
- [results] lowess residual: "the maximum variability detected for a metabolite was 0.78 (78%) for the lowess-corrected dataset"
- [readme] bagging size: "Pearson distance and average method for clustering were considered.Bagging model is performed using partial bagging with n=150 bootstrap samples"
- [readme] input format: "The input data must be in **.csv** format with the independent experiments in the rows and the features (variables) in the columns"
