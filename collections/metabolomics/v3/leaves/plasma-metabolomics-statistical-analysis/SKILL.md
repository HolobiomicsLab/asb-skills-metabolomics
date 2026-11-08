---
name: plasma-metabolomics-statistical-analysis
description: Use when a log2-transformed LC-MS plasma metabolomics feature table has
  both a batch-corrected and raw version plus matched clinical variables, and you
  must quantify whether batch correction improved biological validity (e.g., correlation
  with a clinical analyte or regression on an outcome).
when_to_use_negative:
- Input is raw unnormalized/unlogged intensity data — dbnorm requires data normalized
  and scaled on log2 before correction and comparison
- No independent clinical/reference measurement or outcome exists to validate correction
  quality — the skill's rationale depends on comparing against an external gold standard
- Goal is model selection among correction methods by residual batch variability alone
  (use dbnormSCORE adj-R2 scoring instead), not biological validity assessment against
  clinical variables
edam_operation: http://edamontology.org/operation_3657
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_2269
tools:
- name: dbnorm
  role: R package providing batch effect correction models (ber, bagging ber, parametric
    and nonparametric ComBat, lowess/two-stage) whose corrected outputs are compared
    on correlation and regression performance
  repo: github.com/NBDZ/dbnorm
- name: R
  role: Environment for correlation (Fisher's z CI), lm regression of CKD-EPI on creatinine
    adjusted for age and sex, and limma-based testing with Bonferroni correction
- name: ber
  role: Batch-effect removal model (linear fitting of L/S parameters, originally for
    microarray) whose corrected creatinine intensities feed the correlation/regression
    validation
- name: ComBat
  role: Empirical-Bayes batch correction (parametric and nonparametric variants from
    the sva package) whose corrected outputs are compared in the same validation table
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41598-021-84824-3
    title: DBnorm as an R package for the comparison and selection of appropriate
      statistical methods for batch effect correction in metabolomic studies
  source_content_hash: sha256:b6ea02e4ee76a015e5b4448f679f2e6e4d4de44a3639741e570876b664c670b4
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ComBat
  - R
  - ber
  - dbnorm
  edam_operation: http://edamontology.org/operation_3657
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# plasma-metabolomics-statistical-analysis

## Summary

Statistical validation of batch effect correction in LC-MS plasma metabolomics by correlating metabolite intensities against independent clinical measurements and clinical outcomes (e.g., LC-MS creatinine vs clinical routine creatinine and CKD-EPI) before and after dbnorm batch correction.

## When to use

Use when a log2-transformed LC-MS plasma metabolomics feature table has both a batch-corrected and raw version plus matched clinical variables, and you must quantify whether batch correction improved biological validity (e.g., correlation with a clinical analyte or regression on an outcome).

## When NOT to use

- Input is raw unnormalized/unlogged intensity data — dbnorm requires data normalized and scaled on log2 before correction and comparison
- No independent clinical/reference measurement or outcome exists to validate correction quality — the skill's rationale depends on comparing against an external gold standard
- Goal is model selection among correction methods by residual batch variability alone (use dbnormSCORE adj-R2 scoring instead), not biological validity assessment against clinical variables

## Inputs

- Log2-transformed LC-MS plasma metabolomics .csv feature table (samples in rows, features in columns, batch levels in first column)
- Batch-corrected dataset versions from dbnorm statistical models (lowess, ber, bagging ber, parametric ComBat, nonparametric ComBat)
- Matched clinical measurements table (e.g., clinical routine creatinine, CKD-EPI, age, sex per participant)

## Outputs

- Comparison table of Pearson r, Fisher's z with 95% CI, and Bonferroni-corrected regression coefficients/p-values per batch-correction model (e.g., creatinine_correlation_regression_stats.csv)

## How to apply

Load the log2-transformed metabolomics matrix (samples in rows, features in columns, batch levels in the first column, .csv format) together with matched clinical covariates; for SKIPOGH this is the creatinine feature plus clinical creatinine, CKD-EPI, age and sex for 934 participants with available CKD-EPI. Retrieve raw and corrected intensities from each dbnorm-corrected dataset version (lowess/two-stage, ber, bagging ber, parametric ComBat, nonparametric ComBat) generated via dbnormBer, dbnormBagging, dbnormPcom or dbnormNPcom. For each version compute Pearson's r between the LC-MS feature and its clinical counterpart, transforming r to Fisher's z to derive 95% CI at alpha < 0.05 (SKIPOGH expectation: raw r ≈ 0.18, 95% CI 0.12–0.24; lowess r ≈ 0.5, Fisher's z 0.54, CI 0.48–0.60; ber/ComBat-corrected r ≈ 0.61). Then fit a multiple linear regression of the clinical outcome (CKD-EPI) on the metabolite adjusted for age and sex per data version using R lm/limma, applying Bonferroni correction as the metabolite-selection criterion. Assemble all Pearson r, Fisher's z CI, and regression coefficients/p-values into a comparison table (e.g., creatinine_correlation_regression_stats.csv) and confirm the raw-to-corrected progression reproduces the reported improvement.

## Related tools

- **dbnorm** (R package providing batch effect correction models (ber, bagging ber, parametric and nonparametric ComBat, lowess/two-stage) whose corrected outputs are compared on correlation and regression performance) — github.com/NBDZ/dbnorm
- **R** (Environment for correlation (Fisher's z CI), lm regression of CKD-EPI on creatinine adjusted for age and sex, and limma-based testing with Bonferroni correction)
- **ber** (Batch-effect removal model (linear fitting of L/S parameters, originally for microarray) whose corrected creatinine intensities feed the correlation/regression validation)
- **ComBat** (Empirical-Bayes batch correction (parametric and nonparametric variants from the sva package) whose corrected outputs are compared in the same validation table)

## Examples

```
data<-read.csv("path/to/skipogh_log2.csv",sep=",",header=T,row.names=1); corrected<-dbnormBer(data); cor.test(corrected$creatinine, clinical$creatinine, method="pearson")
```

## Evaluation signals

- Pearson r between LC-MS and clinical creatinine progresses from ~0.18 (raw, 95% CI 0.12–0.24) to ~0.5 (lowess) to ~0.61 (ber/parametric or nonparametric ComBat-corrected)
- CKD-EPI–creatinine multiple regression (adjusted for age and sex) is non-significant on raw data but Bonferroni-corrected p < 0.05 after appropriate correction
- Comparison CSV contains per-model rows with Pearson r, Fisher's z and 95% CI, and regression coefficient with Bonferroni-corrected p-value for each correction version
- Input matrices conform to dbnorm conventions: .csv, samples in rows, features in columns, batch levels in the first column, log2-scaled values
- Regression performed on the full 934 SKIPOGH participants with available CKD-EPI, matching the article's cohort subset

## Limitations

- Residual batch association persists in some corrected versions — lowess-corrected SKIPOGH data showed up to 0.78 (78%) residual batch-explained variability and nonparametric ComBat 0.60 (60%), vs < 0.01 for ber and parametric ComBat, so correlation gains vary by model
- dbnorm's Visodbnorm/dbnormSCORE functions are suggested for fewer than 2000 features for computational speed
- One metabolite (xanthosine 5'-monophosphate) was filtered out with lowess, so not all features survive every correction model

## Evidence

- [results] The correlation coefficient (r) changed from 0.18 (Fisher’s z = 0.18 with a 95% interval level (CI), 0.12 to 0.24) observed in the raw data: "The correlation coefficient (r) changed from 0.18 (Fisher’s z = 0.18 with a 95% interval level (CI), 0.12 to 0.24) observed in the raw data"
- [results] a multiple regression model was applied to the raw, lowess- and ber-corrected data for 934 participants: "a multiple regression model was applied to the raw, lowess- and ber-corrected data for 934 participants"
- [results] We first evaluated the correlation between creatinine levels, as determined by the targeted LC–MS/MS metabolomics experiment and those measured in the clinical laboratory: "We first evaluated the correlation between creatinine levels, as determined by the targeted LC–MS/MS metabolomics experiment and those measured in the clinical laboratory"
- [readme] Data to be uploaded must be normalized and scaled on the log2 to account for the high abundance features: "Data to be uploaded must be normalized and scaled on the log2 to account for the high abundance features (variables) by which technical heterogeneity might be overlooked."
- [results] the datasets corrected by ber and parametric Com Bat functions presented a similar performance in this study, with a maximum variability associated with the batch < 0.01 (1%): "the datasets corrected by ber and parametric Com Bat functions presented a similar performance in this study, with a maximum variability associated with the batch < 0.01 (1%)"
