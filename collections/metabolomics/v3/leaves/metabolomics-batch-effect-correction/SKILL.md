---
name: metabolomics-batch-effect-correction
description: Use when a log2-transformed LC–MS metabolomics matrix (.csv, samples
  in rows, batch labels in column 1) shows batch-dependent clustering (e.g., PCA grouping
  by batch order) and you need to compare batch-correction models quantitatively.
when_to_use_negative:
- Datasets with more than ~2000 features — dbnormSCORE and Visodbnorm are explicitly
  suggested for fewer than 2000 variables for computational speed
- Input is not a batch-annotated metabolomics matrix (no batch levels in the first
  column) or is not yet log2-scaled, violating dbnorm's stated input requirements
- Goal is QC-sample-injection-based drift correction or biological differential analysis
  (volcano plots, MSEA) rather than statistical-model batch adjustment and model selection
edam_operation: http://edamontology.org/operation_2935
edam_topics:
- http://edamontology.org/topic_3272
- http://edamontology.org/topic_0091
tools:
- name: dbnorm
  role: core R package providing dbnormBer, dbnormBagging, dbnormPcom, dbnormNPcom,
    Visodbnorm and dbnormSCORE for batch correction and adj-R² scoring
  repo: https://github.com/NBDZ/dbnorm
- name: ComBat
  role: empirical-Bayes location/scale (L/S) batch correction applied in parametric
    and nonparametric settings via sva
- name: ber
  role: two-stage linear-regression batch-effect model originally built for microarray
    data, applied via dbnormBer; installed from CRAN archive (version 4.0)
- name: R
  role: runtime environment for all dbnorm functions and data loading via read.csv
provenance:
  source_task_ids:
  - task_002
  - task_003
  source_papers:
  - doi: 10.1038/s41598-021-84824-3
    title: DBnorm as an R package for the comparison and selection of appropriate
      statistical methods for batch effect correction in metabolomic studies
  source_content_hash: sha256:a0f7d25b41604778a7cf1f43654fd930ed692974d08521f0aacb191b61d05e3d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/NBDZ/dbnorm
  tools:
  - ComBat
  - R
  - ber
  - dbnorm
  - sva
  edam_operation: http://edamontology.org/operation_2935
  edam_topics:
  - http://edamontology.org/topic_3272
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# metabolomics-batch-effect-correction

## Summary

Correct drift-across-batch in large metabolomics matrices with the dbnorm R package — ber/bagging-ber, parametric and nonparametric ComBat, and lowess — then score each correction with dbnormSCORE via the maximum metabolite-vs-batch adjusted-R² (0–1) to select the best-fitting model.

## When to use

Use when a log2-transformed LC–MS metabolomics matrix (.csv, samples in rows, batch labels in column 1) shows batch-dependent clustering (e.g., PCA grouping by batch order) and you need to compare batch-correction models quantitatively.

## When NOT to use

- Datasets with more than ~2000 features — dbnormSCORE and Visodbnorm are explicitly suggested for fewer than 2000 variables for computational speed
- Input is not a batch-annotated metabolomics matrix (no batch levels in the first column) or is not yet log2-scaled, violating dbnorm's stated input requirements
- Goal is QC-sample-injection-based drift correction or biological differential analysis (volcano plots, MSEA) rather than statistical-model batch adjustment and model selection

## Inputs

- log2-transformed metabolomics matrix in .csv format (experiments in rows, features in columns, batch levels in the first column)
- batch label vector per sample (first column of the matrix)
- optional: matrix with zero/NA missing values for emvd/emvf imputation

## Outputs

- corrected data matrix (.csv) per model: ber, bagging-ber, parametric ComBat, nonparametric ComBat
- dbnormSCORE table of maximum adjusted-R² (score 0–1) per correction model, exported as CSV
- PDF with correlation and score plots (plus PCA/Scree plots from Visodbnorm/dbnormBer etc.) saved in the working directory
- two-column matrix of adjusted-R² per variable for raw vs corrected data

## How to apply

Load the batch-annotated matrix with read.csv (batch levels in the first column, features in columns, data scaled on log2 as required by dbnorm), optionally impute zero/NA values first via emvd (experiment-wide lowest value) or emvf (per-feature lowest value). Run each correction model — dbnormBer (two-stage linear regression L/S fitting), dbnormBagging (partial bagging, n=150 bootstrap samples), dbnormPcom (parametric empirical-Bayes ComBat from sva) and dbnormNPcom (nonparametric ComBat) — then recompute the regression of each metabolite's intensity on batch level and extract the adjusted-R² per metabolite per dataset. Call dbnormSCORE, which compiles correlation and score plots into a PDF plus CSV outputs and reports one score per model equal to the maximum adj-R² (bounded 0–1); lower is better. Pick the model whose score drops residual batch-associated variability furthest toward 0 — in the SKIPOGH benchmark (1079 samples, 239 metabolites, 11 batches), ber and parametric ComBat reached <0.01 while lowess left 0.78 and nonparametric ComBat 0.60.

## Related tools

- **dbnorm** (core R package providing dbnormBer, dbnormBagging, dbnormPcom, dbnormNPcom, Visodbnorm and dbnormSCORE for batch correction and adj-R² scoring) — https://github.com/NBDZ/dbnorm
- **ComBat** (empirical-Bayes location/scale (L/S) batch correction applied in parametric and nonparametric settings via sva)
- **ber** (two-stage linear-regression batch-effect model originally built for microarray data, applied via dbnormBer; installed from CRAN archive (version 4.0))
- **R** (runtime environment for all dbnorm functions and data loading via read.csv)

## Examples

```
data<-read.csv("mydata.csv",sep=",",header=T,row.names=1); library(dbnorm); dbnormSCORE(data)
```

## Evaluation signals

- Score table exists for raw + corrected datasets and each score lies on the 0–1 adj-R² scale
- On SKIPOGH-like data, raw data should show >50% variability batch-explained for most metabolites, and ber/parametric ComBat corrected data should reach maximum adj-R² <0.01
- A model failing to remove drift is detectable as residual max adj-R² near the raw value (lowess: 0.78; nonparametric ComBat: 0.60)
- PDF (correlation/score plots) written to the working directory and corrected-matrix CSVs written to the R temporary folder, as documented in the README Value sections

## Limitations

- dbnormSCORE and Visodbnorm are recommended for <2000 features; larger datasets slow computation
- Lowess and nonparametric ComBat can leave substantial residual batch association (up to 0.78 and 0.60 respectively on SKIPOGH); e.g., xanthosine 5′-monophosphate was filtered out with lowess
- Software and dependency versions are only partially pinned (ber 4.0, NormalizeMets 0.25, metabolomics 0.1.4 from CRAN archive); exact dbnorm version numbers are not stated in the article
- Priors on sklearn-style cross-validation do not apply here — model choice is dataset-specific, which is why dbnorm computes per-dataset scores

## Evidence

- [abstract] dbnorm assigns a score to help users identify the best fitting model for each dataset. This score, which is denoted by adjusted- R squared (adj-R2), ranges from 0 to 1: [evidence span withheld — anchor status: not_found]
- [results] ber and parametric ComBat <0.01 residual variability: "the datasets corrected by ber and parametric Com Bat functions presented a similar performance in this study, with a maximum variability associated with the batch < 0.01 (1%)"
- [results] lowess-corrected SKIPOGH data showed a maximum residual variability of 0.78 (78%): "the maximum variability detected for a metabolite was 0.78 (78%) for the lowess-corrected dataset (Supplementary Table 2)"
- [readme] Data to be uploaded must be normalized and scaled on the log2 ... batch levels considered in the first column: "Data to be uploaded must be normalized and scaled on the log2 to account for the high abundance features"
- [readme] input format requirement: "The input data must be in **.csv** format with the independent experiments in the rows and the features (variables) in the columns, with the `batch` levels considered in the first column."
- [readme] dbnormSCORE recommended for less than 2000 features: "This function is suggested for less than 2000 features (variables) for better computational speed."
- [readme] Bagging partial n=150 bootstrap: "Bagging model is performed using partial bagging with n=150 bootstrap samples"
