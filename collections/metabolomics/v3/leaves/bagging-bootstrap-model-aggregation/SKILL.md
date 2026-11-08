---
name: bagging-bootstrap-model-aggregation
description: Use when correcting across-batch signal drift in a log2-transformed metabolomics
  feature table and you want to stabilize the ber (two-stage linear L/S fitting) model
  via partial bagging with n=150 bootstrap samples, or benchmark bagged ber against
  ComBat variants.
when_to_use_negative:
- Dataset has fewer metabolites than the batch structure can support or is not normalized/scaled
  on log2 — dbnorm requires log2-scaled input to account for high-abundance features
- Goal is supervised differential analysis rather than batch drift removal — use volcano
  plot / MSEA workflow instead
- Dataset exceeds roughly 2000 features — dbnormSCORE and Visodbnorm are suggested
  for less than 2000 variables for computational speed
edam_operation: http://edamontology.org/operation_3658
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0203
tools:
- name: dbnorm
  role: R package providing dbnormBagging, which applies ber-bagging (partial bagging,
    n=150 bootstrap samples) for across-batches signal-drift normalization, plus dbnormSCORE
    for model comparison
  repo: github.com/NBDZ/dbnorm
- name: ber
  role: Base batch-effect removal model (linear fitting of location and scale parameters,
    originally for microarray data) that is aggregated via bootstrapping in the bagging
    variant
- name: ComBat
  role: Empirical Bayes L/S correction from the sva package; run in parametric and
    nonparametric settings as benchmark comparators to the bagged ber result
- name: R
  role: Environment in which dbnorm and its dependencies (sva, limma, pcaMethods,
    bootstrap, caret) are executed
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41598-021-84824-3
    title: DBnorm as an R package for the comparison and selection of appropriate
      statistical methods for batch effect correction in metabolomic studies
  source_content_hash: sha256:7fdc597f9f5d47f1ce42b32b2b870f3ce98e5cf7cdbe3a694a49116e210badad
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ComBat
  - R
  - ber
  - dbnorm
  edam_operation: http://edamontology.org/operation_3658
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# bagging-bootstrap-model-aggregation

## Summary

Apply bagging (bootstrap aggregation) to the ber batch-effect correction model, as implemented in the dbnorm package via dbnormBagging, to stabilize correction of across-batch signal drift in metabolomics data and compare it against ber, parametric ComBat, and nonparametric ComBat using the dbnormSCORE adjusted-R2 metric.

## When to use

Use when correcting across-batch signal drift in a log2-transformed metabolomics feature table and you want to stabilize the ber (two-stage linear L/S fitting) model via partial bagging with n=150 bootstrap samples, or benchmark bagged ber against ComBat variants.

## When NOT to use

- Dataset has fewer metabolites than the batch structure can support or is not normalized/scaled on log2 — dbnorm requires log2-scaled input to account for high-abundance features
- Goal is supervised differential analysis rather than batch drift removal — use volcano plot / MSEA workflow instead
- Dataset exceeds roughly 2000 features — dbnormSCORE and Visodbnorm are suggested for less than 2000 variables for computational speed

## Inputs

- Log2-transformed metabolomics intensity matrix as .csv, samples in rows, features in columns, batch levels in the first column
- Batch labels for each sample (first column of the CSV)
- dbnorm R package (v0.2.2) with dependencies from CRAN and Bioconductor, including sva, ber, NormalizeMets, and metabolomics archives

## Outputs

- Bagged-ber-corrected metabolomics matrix (.csv written to the Rtmp temporary folder)
- Adjusted R-squared value per metabolite for the bagged-ber corrected data (two-column matrix)
- Score table reporting the maximum adj-R2 (0-1) per correction model, including bagged ber
- PCA, Scree, and Correlation diagnostic plots compiled into a PDF in the working directory

## How to apply

Load a log2-transformed metabolomics matrix in .csv format with samples in rows, metabolites in columns, and batch levels in the first column. Run dbnormBagging(data), which performs batch effect correction using ber-bagging with partial bagging (bootstrap sample size n = 150), then recompute the regression of each metabolite intensity on batch level and extract adjusted R-squared per metabolite. Run dbnormSCORE(data) to obtain, for each model including bagged ber, a 0-to-1 score equal to the maximum adj-R2 explained by batch; lower scores indicate better drift removal. In the SKIPOGH benchmark (1079 samples, 239 metabolites, 11 batches), bagged ber behaved like ber and parametric ComBat, dropping maximum residual batch-associated variability below 0.01 (1%), whereas lowess left 0.78 and nonparametric ComBat left 0.60. Use the per-model score table plus PCA/correlation plots compiled into PDF and CSV outputs to decide whether bagging improved stability over plain ber.

## Related tools

- **dbnorm** (R package providing dbnormBagging, which applies ber-bagging (partial bagging, n=150 bootstrap samples) for across-batches signal-drift normalization, plus dbnormSCORE for model comparison) — github.com/NBDZ/dbnorm
- **ber** (Base batch-effect removal model (linear fitting of location and scale parameters, originally for microarray data) that is aggregated via bootstrapping in the bagging variant)
- **ComBat** (Empirical Bayes L/S correction from the sva package; run in parametric and nonparametric settings as benchmark comparators to the bagged ber result)
- **R** (Environment in which dbnorm and its dependencies (sva, limma, pcaMethods, bootstrap, caret) are executed)

## Examples

```
library(dbnorm); data <- read.csv("skiogH_log2.csv", header=TRUE, row.names=1); dbnormBagging(data); dbnormSCORE(data)
```

## Evaluation signals

- dbnormSCORE output exists and the bagged-ber maximum adj-R2 falls below 0.01 (1%) as reported for SKIPOGH, matching ber and parametric ComBat performance
- All reported scores lie on the bounded 0-1 adjusted-R2 scale
- Corrected data show no residual association of metabolite intensities to batch level, unlike lowess (residual max 0.78) and nonparametric ComBat (residual max 0.60)
- PDF of PCA/Scree/Correlation plots is written to the working directory and the corrected-data CSV is saved in the Rtmp folder, confirming dbnormBagging completed
- PCA of corrected data no longer clusters samples strictly by batch order, in contrast to raw SKIPOGH data where samples clustered by batch

## Limitations

- dbnormSCORE, Visodbnorm, and related functions are suggested for less than 2000 features for better computational speed; larger feature sets may be slow
- Input must already be normalized and log2-scaled; dbnorm does not handle raw, unscaled intensities
- Software version of dbnorm and precise versions of ber and ComBat are not specified in the article; dependencies must be installed from CRAN archives at pinned versions (ber 4.0, NormalizeMets 0.25, metabolomics 0.1.4)
- Bagging variant uses partial bagging with n=150 bootstrap samples; no adaptive tuning of this parameter is described

## Evidence

- [readme] Bagging model is performed using partial bagging with n=150 bootstrap samples: "Bagging model is performed using partial bagging with n=150 bootstrap samples"
- [results] ber and parametric ComBat achieved maximum residual batch variability < 0.01 (1%) in SKIPOGH: "the datasets corrected by ber and parametric Com Bat functions presented a similar performance in this study, with a maximum variability associated with the batch < 0.01 (1%)"
- [abstract] dbnorm assigns a score (adj-R2, 0 to 1) to help users identify the best fitting model for each dataset: [evidence span withheld — anchor status: not_found]
- [abstract] no association was present in parametric ComBat-, ber- and bagging ber-corrected data: [evidence span withheld — anchor status: not_found]
- [readme] Data to be uploaded must be normalized and scaled on the log2: "Data to be uploaded must be normalized and scaled on the log2 to account for the high abundance features (variables)"
