---
name: multiple-regression-egfr-creatinine-association
description: Use when a targeted LC-MS metabolomics feature (e.g. creatinine) must
  be validated against an external clinical trait (CKD-EPI/eGFR, clinical routine
  creatinine) to check whether batch correction improves the biological signal.
when_to_use_negative:
- No paired clinical outcome (CKD-EPI or clinical creatinine) is available for the
  metabolomics participants, so the association model cannot be fit
- The goal is batch correction/model selection itself rather than post-correction
  validation — use Visodbnorm or dbnormSCORE instead
- Dataset has more than 2000 features, where dbnorm's Visodbnorm/dbnormSCORE functions
  are not recommended for computational speed
edam_operation: http://edamontology.org/operation_3656
edam_topics:
- http://edamontology.org/topic_2269
tools:
- name: dbnorm
  role: R package supplying the batch-correction models (lowess, ber, bagging ber,
    parametric and nonparametric ComBat) whose corrected creatinine values are regressed
    on CKD-EPI
  repo: https://github.com/NBDZ/dbnorm
- name: R (stats::lm)
  role: Fits the multiple linear regression of CKD-EPI on creatinine adjusted for
    age and sex per data version
- name: limma
  role: R/Bioconductor package used alongside lm for the regression analysis and significance
    testing
- name: ber
  role: Two-stage batch-correction model (installed at version 4.0) whose corrected
    data are one regression input
- name: ComBat
  role: Empirical-Bayes batch correction (parametric and nonparametric, via sva) supplying
    alternative corrected creatinine intensities
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41598-021-84824-3
    title: DBnorm as an R package for the comparison and selection of appropriate
      statistical methods for batch effect correction in metabolomic studies
  source_content_hash: sha256:9aa259ee5b7ba9981ac22e524afb306d9064103fb8ae2228cc0d2a3e704e05dd
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/NBDZ/dbnorm
  tools:
  - ComBat
  - R
  - R (stats::lm)
  - ber
  - dbnorm
  - limma
  edam_operation: http://edamontology.org/operation_3656
  edam_topics:
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8695652173913043
n_claims: 23
---

# multiple-regression-egfr-creatinine-association

## Summary

Validate batch-effect correction in LC-MS metabolomics by regressing a clinical renal outcome (CKD-EPI) on the LC-MS creatinine feature adjusted for age and sex, comparing raw vs. dbnorm-corrected SKIPOGH datasets. It tests whether statistical batch correction restores a biologically expected metabolite–clinical association that is masked in raw data.

## When to use

Use when a targeted LC-MS metabolomics feature (e.g. creatinine) must be validated against an external clinical trait (CKD-EPI/eGFR, clinical routine creatinine) to check whether batch correction improves the biological signal.

## When NOT to use

- No paired clinical outcome (CKD-EPI or clinical creatinine) is available for the metabolomics participants, so the association model cannot be fit
- The goal is batch correction/model selection itself rather than post-correction validation — use Visodbnorm or dbnormSCORE instead
- Dataset has more than 2000 features, where dbnorm's Visodbnorm/dbnormSCORE functions are not recommended for computational speed

## Inputs

- Log2-transformed SKIPOGH plasma metabolomics data matrix in .csv format (samples in rows, features in columns, batch level in the first column)
- LC-MS/MS creatinine feature intensities (raw and corrected by lowess, ber, bagging ber, parametric ComBat, nonparametric ComBat)
- Clinical covariate table: clinical routine creatinine, CKD-EPI, age and sex for the 934 participants with available CKD-EPI

## Outputs

- Per-model multiple regression coefficients and Bonferroni-corrected p-values for CKD-EPI ~ creatinine + age + sex
- Pearson correlation coefficients with Fisher's z 95% confidence intervals comparing LC-MS vs. clinical creatinine per data version
- Comparison table creatinine_correlation_regression_stats.csv

## How to apply

Start from the log2-transformed metabolomics data (.csv, samples in rows, features in columns, batch level in the first column) merged with clinical covariates — clinical routine creatinine, CKD-EPI, age, and sex for the 934 SKIPOGH participants with available CKD-EPI. Obtain raw and corrected creatinine intensities from each batch-correction model in dbnorm (lowess via Visodbnorm, ber via dbnormBer, ber-bagging via dbnormBagging, parametric ComBat via dbnormPcom, nonparametric ComBat via dbnormNPcom). For each data version fit lm(CKD_EPI ~ creatinine + age + sex) with the R stats package (using limma utilities) and apply Bonferroni correction as the significance criterion for metabolite selection. Compute Pearson's r (with Fisher's z 95% CI) between LC-MS and clinical creatinine per version as a parallel check. Assemble Pearson r, Fisher's z CI, and regression coefficients/Bonferroni-corrected p-values into a comparison table; the rationale is that raw data mask the creatinine–renal-function association while appropriate correction (ber, parametric ComBat) recovers it.

## Related tools

- **dbnorm** (R package supplying the batch-correction models (lowess, ber, bagging ber, parametric and nonparametric ComBat) whose corrected creatinine values are regressed on CKD-EPI) — https://github.com/NBDZ/dbnorm
- **R (stats::lm)** (Fits the multiple linear regression of CKD-EPI on creatinine adjusted for age and sex per data version)
- **limma** (R/Bioconductor package used alongside lm for the regression analysis and significance testing)
- **ber** (Two-stage batch-correction model (installed at version 4.0) whose corrected data are one regression input)
- **ComBat** (Empirical-Bayes batch correction (parametric and nonparametric, via sva) supplying alternative corrected creatinine intensities)

## Examples

```
m <- lm(CKD_EPI ~ creatinine + age + sex, data = skpogh_corrected); p.adjust(summary(m)$coefficients[, "Pr(>|t|)"], method = "bonferroni")
```

## Evaluation signals

- Raw-data regression of CKD-EPI on creatinine must be non-significant, while the corrected-data regression becomes significant (Bonferroni-corrected p < 0.05)
- Pearson r between LC-MS and clinical creatinine must progress from ~0.18 (raw) to ~0.5 (lowess) to ~0.61 (statistical-model-corrected)
- ber- and parametric ComBat-corrected data should show maximum residual batch variability < 0.01 (1%), versus up to 0.78 for lowess and 0.60 for nonparametric ComBat
- Exactly the 934 participants with available CKD-EPI enter the regression
- The comparison table contains Pearson r, Fisher's z with 95% CI, and coefficients/p-values for each correction model (raw, lowess, ber, bagging ber, parametric ComBat, nonparametric ComBat)

## Limitations

- Lowess- and nonparametric ComBat-corrected data retain residual association with batch and residual maximum variation up to 0.78 and 0.60 respectively, weakening conclusions drawn from those versions
- Input must be log2-transformed .csv with batch level in the first column; untransformed or wrongly formatted data can obscure high-abundance feature heterogeneity
- dbnorm's Visodbnorm and dbnormSCORE functions are suggested for fewer than 2000 features only

## Evidence

- [results] a multiple regression model was applied to the raw, lowess- and ber-corrected data for 934 participants: "a multiple regression model was applied to the raw, lowess- and ber-corrected data for 934 participants"
- [abstract] some residual association was observed in lowess- and nonparametric Com Bat-corrected data, while no association was present in parametric Com Bat-, ber- and bagging ber-corrected data.: [evidence span withheld — anchor status: not_found]
- [discussion] supports the necessity of data cleaning to remove unwanted technical variation: "supports the necessity of data cleaning to remove unwanted technical variation"
- [readme] The input data must be in .csv format with the independent experiments in the rows and the features (variables) in the columns: "The input data must be in **.csv** format with the independent experiments in the rows and the features (variables) in the columns"
