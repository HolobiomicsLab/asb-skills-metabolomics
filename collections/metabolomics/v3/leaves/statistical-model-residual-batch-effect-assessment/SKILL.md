---
name: statistical-model-residual-batch-effect-assessment
description: Use when after applying a batch correction model (ber, parametric/non-parametric
  ComBat, bagging ber or lowess) to a log2-scaled metabolomics matrix, you must decide
  whether residual intensity differences across analytical batches (e.
when_to_use_negative:
- When the data matrix has not yet undergone any batch correction — use raw-data PCA/batch
  profiling (e.g. dbnorm's PCA and ProfPlotraw) instead, since this skill assumes
  corrected data are already available.
- When the goal is to select the best model across all metabolites at once; use dbnormSCORE,
  which calculates the score for the maximum adjusted R-squared over all features,
  rather than inspecting a single metabolite's regression.
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: dbnorm
  role: R package that provides the corrected datasets, batch-level annotations and
    the adjusted-R2-based score (via dbnormSCORE) against which the per-metabolite
    batch regression is compared.
  repo: https://github.com/NBDZ/dbnorm
- name: R (stats::lm)
  role: Performs the linear regression of each metabolite's intensity on the batch
    factor to compute the adjusted R-squared of residual batch association.
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41598-021-84824-3
    title: DBnorm as an R package for the comparison and selection of appropriate
      statistical methods for batch effect correction in metabolomic studies
  source_content_hash: sha256:192836a535c96da39366b5238a04fd5b3585fec52d496b7ecb7259ccbf8915e3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/NBDZ/dbnorm
  tools:
  - R
  - R (stats::lm)
  - dbnorm
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# statistical-model-residual-batch-effect-assessment

## Summary

Quantifies how much batch-level variance remains in metabolomics signal after a batch-effect correction model (e.g. lowess) by regressing each metabolite's intensity on the batch factor and comparing adjusted R-squared against a per-model maximum residual score.

## When to use

Use when, after applying a batch correction model (ber, parametric/non-parametric ComBat, bagging ber or lowess) to a log2-scaled metabolomics matrix, you must decide whether residual intensity differences across analytical batches (e.g. citrate in SKIPOGH) persist or whether the model removed the drift.

## When NOT to use

- When the data matrix has not yet undergone any batch correction — use raw-data PCA/batch profiling (e.g. dbnorm's PCA and ProfPlotraw) instead, since this skill assumes corrected data are already available.
- When the goal is to select the best model across all metabolites at once; use dbnormSCORE, which calculates the score for the maximum adjusted R-squared over all features, rather than inspecting a single metabolite's regression.

## Inputs

- Log2-transformed, batch-corrected metabolomics matrix in .csv format (rows = samples/experiments, columns = metabolite features) with the batch-level factor in the first column
- Corrected data files produced by dbnorm (e.g. the .csv outputs from dbnormBer, dbnormPcom, dbnormNPcom, dbnormBagging or Visodbnorm stored in the Rtmp temporary directory)
- Metabolite feature of interest (e.g. citrate column) and the batch factor column

## Outputs

- Adjusted R-squared value of the metabolite ~ batch regression per corrected dataset
- CSV file (e.g. batch_association_citrate.csv) listing metabolite name, adjusted R-squared and batch regression result
- Comparison of the per-metabolite adjusted R-squared against the maximum residual variability score for each dbnorm model (e.g. 0.78 for lowess, <0.01 for ber/parametric ComBat, 0.60 for non-parametric ComBat)

## How to apply

Load the corrected data matrix (e.g. lowess-corrected SKIPOGH data, log2-transformed, with batch levels in the first column) into R and select the metabolite feature of interest, such as the citrate column. Regress the metabolite's intensity on the batch-level factor using stats::lm (citrate ~ batch) and record the adjusted R-squared of the batch term; optionally confirm magnitude with a one-way ANOVA of citrate ~ batch. Compare this adjusted R-squared against the maximum residual variability score reported by dbnormSCORE for each model (e.g. lowess-corrected SKIPOGH reaches 0.78, i.e. 78%, while ber and parametric ComBat stay below 0.01). Write the per-metabolite result (metabolite name, adjusted R-squared, batch regression) to a CSV such as batch_association_citrate.csv to document which model left a batch association. The rationale is that dbnorm computes this adjusted-R2-based score to flag which model fits the data structure best, so any metabolite whose batch regression exceeds a small threshold indicates the correction failed to fully remove the drift.

## Related tools

- **dbnorm** (R package that provides the corrected datasets, batch-level annotations and the adjusted-R2-based score (via dbnormSCORE) against which the per-metabolite batch regression is compared.) — https://github.com/NBDZ/dbnorm
- **R (stats::lm)** (Performs the linear regression of each metabolite's intensity on the batch factor to compute the adjusted R-squared of residual batch association.)

## Examples

```
library(dbnorm); data <- read.csv('lowess_corrected_skipogh.csv', row.names=1); m <- lm(citrate ~ batch, data=data); summary(m)$adj.r.squared
```

## Evaluation signals

- Adjusted R-squared of the metabolite ~ batch regression should match the model's reported maximum residual variability (e.g. ≈0.78 for lowess-corrected SKIPOGH citrate, <0.01 for ber and parametric ComBat, 0.60 for non-parametric ComBat).
- The output CSV must contain the metabolite name, the computed adjusted R-squared and the batch regression result for every corrected dataset assessed.
- If the corrected data were produced by dbnorm, the residual association should disappear (adjusted R-squared near 0) for parametric ComBat, ber and bagging ber, matching the article's finding of 'no association was present in parametric ComBat-, ber- and bagging ber-corrected data'.
- Input data must be log2-scaled with batch levels in the first column, per dbnorm's README requirement; otherwise the regression design is invalid.
- Consistency check: the per-metabolite adjusted R-squared should not exceed the model's maximum residual variability score computed by dbnormSCORE for that dataset.

## Limitations

- dbnorm's dbnormSCORE and Visodbnorm functions are suggested for datasets with fewer than 2000 features for computational speed, so very large feature counts may slow down or break the scoring workflow.
- The adjusted R-squared score only signals how well the batch factor explains a metabolite's intensity; it does not by itself prove the correction was inappropriate for biological variation, so results should be interpreted alongside biological correlation checks (e.g. LC-MS creatinine vs clinical creatinine).
- Lowess and non-parametric ComBat are known to leave residual batch association (e.g. 0.78 and 0.60 max variability respectively in SKIPOGH), so a single metabolite's residual R-squared can be high without indicating the model itself is misapplied.

## Evidence

- [other] The source card confirms the workflow: extract the citrate column, regress it on the batch factor with R's stats::lm, compute adjusted R-squared, and write the result to batch_association_citrate.csv.: [evidence span withheld — anchor status: not_found]
- [readme] dbnorm's SCORE function provides the adjusted R-squared score used for comparison.: "the adjusted coefficient of determination or Adjusted R-Squared is calculated for each variable estimated in a regression model for its dependency to the batch level in the raw data and treated data"
- [results] dbnorm's overall score is based on the maximum adjusted R-squared across metabolites.: [evidence span withheld — anchor status: not_found]
- [results] The article reports the expected residual variability thresholds per model.: "the maximum variability detected for a metabolite was 0.78 (78%) for the lowess-corrected dataset (Supplementary Table 2)"
- [readme] README specifies the required input format for dbnorm-corrected data.: "The input data must be in **.csv** format with the independent experiments in the rows and the features (variables) in the columns, with the `batch` levels considered in the first column."
