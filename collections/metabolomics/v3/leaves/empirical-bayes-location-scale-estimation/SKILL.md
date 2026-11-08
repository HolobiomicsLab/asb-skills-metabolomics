---
name: empirical-bayes-location-scale-estimation
description: Use when a log2-normalized metabolomics dataset in CSV format (samples
  in rows, features in columns, batch levels in the first column) shows across-batch
  signal drift and you need to correct it and compare empirical-Bayes correction models.
when_to_use_negative:
- The input matrix has no batch annotation in the first column — L/S estimation requires
  batch labels and cannot be run on unbatched data
- The dataset contains more than 2000 features — dbnorm's correction/scoring functions
  (Visodbnorm, dbnormSCORE) are suggested only for fewer than 2000 variables for computational
  speed; use the decomposed dbnormBer/dbnormBagging/dbnormPcom/dbnormNPcom functions
  instead
- The data have not been normalized and log2-scaled — running the models on raw unscaled
  intensities violates the documented input requirement and can let high-abundance
  features dominate technical heterogeneity estimates
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: dbnorm
  role: R package providing the correction functions (dbnormPcom, dbnormNPcom, dbnormBer,
    dbnormBagging) and the dbnormSCORE adj-R² model-comparison scoring
  repo: github.com/NBDZ/dbnorm
- name: ComBat
  role: Empirical Bayes L/S correction model run by dbnorm in parametric (dbnormPcom)
    and nonparametric (dbnormNPcom) settings via the sva package
- name: ber
  role: Two-stage linear-regression L/S correction model for across-batch drift (dbnormBer),
    originally developed for microarray data
- name: R
  role: Execution environment for dbnorm and all dependent CRAN/Bioconductor packages
provenance:
  source_task_ids:
  - task_003
  - task_002
  source_papers:
  - doi: 10.1038/s41598-021-84824-3
    title: DBnorm as an R package for the comparison and selection of appropriate
      statistical methods for batch effect correction in metabolomic studies
  source_content_hash: sha256:fa37e3682fe9380c800326ef1b17c0303f9797d7e4115eff52e4cf4950fe3f94
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ComBat
  - R
  - ber
  - dbnorm
  - sva
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# empirical-bayes-location-scale-estimation

## Summary

Empirical Bayes location/scale (L/S) batch-effect correction for metabolomics matrices via the dbnorm R package, which runs ComBat in parametric and nonparametric settings (sva) alongside the two-stage linear-regression ber and bagging-ber (n=150 bootstrap) models, and scores each model's residual batch dependence with maximum adjusted-R².

## When to use

Use when a log2-normalized metabolomics dataset in CSV format (samples in rows, features in columns, batch levels in the first column) shows across-batch signal drift and you need to correct it and compare empirical-Bayes correction models.

## When NOT to use

- The input matrix has no batch annotation in the first column — L/S estimation requires batch labels and cannot be run on unbatched data
- The dataset contains more than 2000 features — dbnorm's correction/scoring functions (Visodbnorm, dbnormSCORE) are suggested only for fewer than 2000 variables for computational speed; use the decomposed dbnormBer/dbnormBagging/dbnormPcom/dbnormNPcom functions instead
- The data have not been normalized and log2-scaled — running the models on raw unscaled intensities violates the documented input requirement and can let high-abundance features dominate technical heterogeneity estimates

## Inputs

- CSV metabolomics matrix, samples in rows, features in columns, batch levels in the first column, normalized and log2-transformed
- Batch level labels as a factor/character first column defining analytical runs

## Outputs

- Corrected data matrices (CSV) for ber, ber-bagging, parametric ComBat, and nonparametric ComBat
- Adjusted-R² tables per metabolite and per model, plus a max-adj-R² score table bounded to the 0–1 scale
- PCA, Scree, and correlation plots compiled into a PDF, with corrected CSVs written to the Rtmpe temporary directory and RLA plots shown in the RStudio Viewer

## How to apply

Load the batch-annotated matrix with read.csv (rows = independent experiments, columns = features, batch levels in column 1) and ensure data are normalized and log2-scaled before correction, since scaling prevents high-abundance features from masking technical heterogeneity. Apply the empirical-Bayes L/S correction models via dbnormPcom and dbnormNPcom (parametric and nonparametric ComBat, Johnson et al. 2007, implemented in sva) and via dbnormBer and dbnormBagging (the two-stage linear-regression ber procedure of Giordan 2013, with bagging performed as partial bagging over n=150 bootstrap samples). For each metabolite, regress its intensity on the batch level in raw and corrected data and extract adjusted-R²; dbnormSCORE then compiles a 0–1 score per model based on the maximum batch-explained variability, with correlation and score plots compiled to PDF and corrected matrices to CSV. Judge model fit by the residual score: lower maximum adjusted-R² means better removal of across-batch drift (in the SKIPOGH benchmark of 1079 samples, 239 metabolites, 11 batches, ber and parametric ComBat dropped below 0.01, lowess left 0.78 and nonparametric ComBat 0.60). Decision point: choose the model whose adj-R² score best fits your data structure rather than applying one model blindly.

## Related tools

- **dbnorm** (R package providing the correction functions (dbnormPcom, dbnormNPcom, dbnormBer, dbnormBagging) and the dbnormSCORE adj-R² model-comparison scoring) — github.com/NBDZ/dbnorm
- **ComBat** (Empirical Bayes L/S correction model run by dbnorm in parametric (dbnormPcom) and nonparametric (dbnormNPcom) settings via the sva package)
- **ber** (Two-stage linear-regression L/S correction model for across-batch drift (dbnormBer), originally developed for microarray data)
- **R** (Execution environment for dbnorm and all dependent CRAN/Bioconductor packages)

## Examples

```
data<-read.csv("path/to/mydata.csv",sep=",",header=T,row.names=1); library(dbnorm); dbnormSCORE(data)
```

## Evaluation signals

- All adjusted-R² scores are bounded to the 0–1 scale, with per-model scores equal to the maximum metabolite-vs-batch regression adj-R²
- On SKIPOGH-like data (1079 samples, 239 metabolites, 11 batches), ber and parametric ComBat corrected matrices should reach maximum residual batch variability < 0.01 (1%)
- Nonparametric ComBat-corrected data should not exceed a residual maximum adj-R² of ~0.60, and lowess ~0.78 — scores near the raw level (>50% batch-explained) indicate the correction failed
- Expected outputs exist: PDF compiled plots in the working directory, corrected .csv files in a folder initiated with Rtmpe, and PCA of corrected data no longer clustering samples by batch order

## Limitations

- Visodbnorm and dbnormSCORE are suggested for datasets with fewer than 2000 features (variables) for better computational speed
- Lowess and nonparametric ComBat corrections left residual batch association in the SKIPOGH evaluation, while ber/bagging-ber and parametric ComBat removed it — model choice is data-dependent
- Bagging is performed only as partial bagging with a fixed n=150 bootstrap samples; no configuration of bootstrap size is exposed
- Version numbers of dbnorm, ComBat, and ber dependencies are not precisely stated beyond the ber v4.0 and dbnorm V-0.2.2 mentions, so reproducibility requires pinning the archive installs from CRAN

## Evidence

- [abstract] ComBat is also a function established in this category that uses the empirical Bayes methods (EB) framework for estimation of the L/S parameters that represent batch effects: [evidence span withheld — anchor status: not_found]
- [results] the datasets corrected by ber and parametric ComBat functions presented a similar performance in this study, with a maximum variability associated with the batch < 0.01 (1%): "the datasets corrected by ber and parametric Com Bat functions presented a similar performance in this study, with a maximum variability associated with the batch < 0.01 (1%)"
- [discussion] We provide "dbnorm" as an efficient and user-friendly tool for the removal of drift across batches.: "We provide “dbnorm” as an efficient and user-friendly tool for the removal of drift across batches."
- [other] This function gives a quick notification about the performance of the statistical models ... in accommodating technical variability.: [evidence span withheld — anchor status: not_found]
- [readme] The input data must be in .csv format with the independent experiments in the rows and the features (variables) in the columns, with the batch levels considered in the first column.: "The input data must be in **.csv** format with the independent experiments in the rows and the features (variables) in the columns, with the `batch` levels considered in the first column."
