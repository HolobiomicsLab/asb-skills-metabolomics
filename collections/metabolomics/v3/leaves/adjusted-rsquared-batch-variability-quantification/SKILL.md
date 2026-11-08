---
name: adjusted-rsquared-batch-variability-quantification
description: Use when you have a batch-corrected (or raw) metabolomics matrix with
  a batch-level column and need to measure per-metabolite residual association with
  batch to select the correction model.
when_to_use_negative:
- 'The dataset has more than 2000 features: dbnormSCORE is only suggested for less
  than 2000 features for better computational speed — decompose the data or use separate
  per-model functions instead.'
- The goal is batch correction itself rather than quantifying residual batch association
  — use Visodbnorm/dbnormBer/dbnormBagging/dbnormPcom/dbnormNPcom instead of the scoring
  step.
- The input data has not been normalized and log2-scaled; the README states data 'must
  be normalized and scaled on the log2' before upload.
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: dbnorm
  role: R package providing dbnormSCORE, which fits per-variable regressions of intensity
    on batch level, computes adjusted R-squared, and converts the maximum adj-R2 into
    a 0–1 model-selection score with correlation and score plots.
  repo: https://github.com/NBDZ/dbnorm
- name: R
  role: Runtime environment in which stats::lm and one-way ANOVA regress citrate (or
    any feature) intensity on the batch factor; equivalent to the dbnorm internal
    regression for a manual per-metabolite analysis.
- name: ComBat (sva)
  role: One of the correction models whose output adj-R2 score is compared; in SKIPOGH
    the parametric version reduced maximum residual batch adj-R2 to < 0.01 while the
    nonparametric version left 0.60.
- name: ber
  role: Linear L/S batch-correction model whose corrected SKIPOGH data achieved maximum
    batch adj-R2 < 0.01, making it a top-scoring candidate in the adj-R2 comparison.
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41598-021-84824-3
    title: DBnorm as an R package for the comparison and selection of appropriate
      statistical methods for batch effect correction in metabolomic studies
  source_content_hash: sha256:36a2c332082a6491af221a8813604b16df7923582431db8a2e3c3a8a4144554e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/NBDZ/dbnorm
  tools:
  - ComBat (sva)
  - R
  - ber
  - dbnorm
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# adjusted-rsquared-batch-variability-quantification

## Summary

Quantify residual batch dependency of each metabolite by regressing log2-transformed feature intensity on the analytical batch level factor and reporting adjusted R-squared, the dbnorm scoring metric (0–1) used to compare batch-effect correction models such as lowess, ber, bagging ber, and parametric/non-parametric ComBat.

## When to use

Use when you have a batch-corrected (or raw) metabolomics matrix with a batch-level column and need to measure per-metabolite residual association with batch to select the best correction model.

## When NOT to use

- The dataset has more than 2000 features: dbnormSCORE is only suggested for less than 2000 features for better computational speed — decompose the data or use separate per-model functions instead.
- The goal is batch correction itself rather than quantifying residual batch association — use Visodbnorm/dbnormBer/dbnormBagging/dbnormPcom/dbnormNPcom instead of the scoring step.
- The input data has not been normalized and log2-scaled; the README states data 'must be normalized and scaled on the log2' before upload.

## Inputs

- log2-normalized metabolomics intensity matrix as .csv (rows = samples, columns = features) with the batch level factor in the first column
- optional named metabolite feature column to interrogate individually (e.g. the citrate column of the lowess-corrected SKIPOGH matrix)

## Outputs

- per-metabolite adjusted R-squared (adj-R2, range 0–1) of the regression of feature intensity on batch level
- a dbnorm score based on the maximum adj-R2 for each correction model (lowess/ber/bagging ber/parametric ComBat/nonparametric ComBat), presented as a score table and PDF correlation/score plots
- CSV files: per-metabolite batch-association table (e.g. batch_association_citrate.csv) and two-column adj-R2 vectors written to an Rtmpe temporary directory

## How to apply

Load the log2-normalized data as .csv with independent experiments in rows, features in columns, and the batch levels in the first column (e.g. the lowess-corrected SKIPOGH matrix). For each metabolite column, fit a linear regression of intensity on the batch-level factor (stats::lm, citrate ~ batch in the SKIPOGH example) and extract the adjusted coefficient of determination (adj-R2). Optionally confirm magnitude with a one-way ANOVA of citrate ~ batch. dbnorm computes this score over all variables via dbnormSCORE, ranging from 0 to 1; the model whose maximum adj-R2 is lowest is judged the best fit — e.g. ber and parametric ComBat reached < 0.01 in SKIPOGH while lowess left a maximum of 0.78 and nonparametric ComBat 0.60. Write per-metabolite results (metabolite, adj-R2) to batch_association_citrate.csv, or run dbnormSCORE directly, which emits correlation and score plots as PDF and the adj-R2 vectors as .csv in an Rtmpe temporary folder. The workflow is recommended for datasets with fewer than 2000 features for computational speed.

## Related tools

- **dbnorm** (R package providing dbnormSCORE, which fits per-variable regressions of intensity on batch level, computes adjusted R-squared, and converts the maximum adj-R2 into a 0–1 model-selection score with correlation and score plots.) — https://github.com/NBDZ/dbnorm
- **R** (Runtime environment in which stats::lm and one-way ANOVA regress citrate (or any feature) intensity on the batch factor; equivalent to the dbnorm internal regression for a manual per-metabolite analysis.)
- **ComBat (sva)** (One of the correction models whose output adj-R2 score is compared; in SKIPOGH the parametric version reduced maximum residual batch adj-R2 to < 0.01 while the nonparametric version left 0.60.)
- **ber** (Linear L/S batch-correction model whose corrected SKIPOGH data achieved maximum batch adj-R2 < 0.01, making it a top-scoring candidate in the adj-R2 comparison.)

## Examples

```
data <- read.csv("lowess_corrected_skipogh.csv", sep = ",", header = TRUE, row.names = 1); summary(lm(citrate ~ batch, data = data))$adj.r.squared
```

## Evaluation signals

- Adjusted R-squared values must fall in [0, 1]; the dbnorm score is defined on that range ('This score, which is denoted by adjusted- R squared (adj-R2), ranges from 0 to 1').
- For the SKIPOGH lowess-corrected dataset the maximum residual adj-R2 should be close to 0.78 — an individual metabolite such as citrate reproducing near this ceiling confirms the residual batch shift.
- Correctly applied parametric ComBat and ber corrections should yield maximum batch adj-R2 < 0.01; values above that indicate the correction was not applied or batch labels were misaligned.
- The regression must use the batch-level factor taken from the first column of the .csv; running on data without batch labels or on uncorrected raw data by mistake will shift adj-R2 upward (raw SKIPOGH showed > 50% of variability explained by batch for most metabolites).
- dbnormSCORE should write a PDF of correlation/score plots to the working directory and .csv adj-R2 tables to an Rtmpe folder; absence of these artifacts indicates the function did not complete.

## Limitations

- The adjusted R-squared reflects only batch-level linear association; it cannot detect nonlinear or interaction residual drift, and 'some residual association was observed in lowess- and nonparametric ComBat-corrected data' even after correction.
- dbnormSCORE and Visodbnorm are recommended only for datasets with fewer than 2000 features for computational speed.
- Data must be log2-normalized and scaled before upload; 'Data to be uploaded must be normalized and scaled on the log2 to account for the high abundance features (variables)'.
- Input format is strict: .csv with batch levels in the first column; other layouts require reformatting before the regression or dbnormSCORE run.

## Evidence

- [abstract] score definition: [evidence span withheld — anchor status: not_found]
- [results] lowess residual magnitude: "the maximum variability detected for a metabolite was 0.78 (78%) for the lowess-corrected dataset (Supplementary Table 2)"
- [results] best-performing models: "the datasets corrected by ber and parametric Com Bat functions presented a similar performance in this study, with a maximum variability associated with the batch < 0.01 (1%)"
- [readme] input format: "The input data must be in **.csv** format with the independent experiments in the rows and the features (variables) in the columns, with the `batch` levels considered in the first column."
- [readme] feature-count constraint: "This function is suggested for less than 2000 features (variables) for better computational speed."
