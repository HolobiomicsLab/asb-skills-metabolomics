---
name: metabolite-batch-association-regression
description: Use when you have a batch-corrected log2 metabolomics matrix (batch levels
  in column 1) and need to test whether a specific metabolite — e.g. citrate — still
  shows batch-associated intensity shifts after correction.
when_to_use_negative:
- The input is an unprocessed instrument export (e.g. raw XCMS m/z table) rather than
  a normalized, log2-scaled, batch-labeled .csv — dbnorm's regression assumes pre-processed
  input.
- The goal is choosing among correction models across all metabolites at once rather
  than diagnosing one specific feature; use dbnormSCORE's score plot over the full
  variable matrix instead.
- The dataset has more than 2000 features, where dbnorm's own guidance says its graphical/regression
  functions become computationally slow.
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: R
  role: execution environment for stats::lm regression of metabolite intensity on
    batch factor and one-way ANOVA
- name: dbnorm
  role: R package that supplies the corrected matrices (via Visodbnorm/ProfPlot* functions)
    and whose adj-R2 scoring framework this per-metabolite regression replicates
  repo: https://github.com/NBDZ/dbnorm
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41598-021-84824-3
    title: DBnorm as an R package for the comparison and selection of appropriate
      statistical methods for batch effect correction in metabolomic studies
  source_content_hash: sha256:c672d4306070881582ca8e84e6f5225dbb65591d0f3e75858acf4da0eff2c913
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/NBDZ/dbnorm
  tools:
  - R
  - dbnorm
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 5
---

# metabolite-batch-association-regression

## Summary

Quantifies residual batch dependence of a single metabolite (e.g. citrate) by regressing its corrected intensity on the analytical batch factor, reporting the adjusted R-squared so that incomplete drift correction (as with lowess on SKIPOGH citrate, residual 0.78) can be detected and diagnosed.

## When to use

Use when you have a batch-corrected log2 metabolomics matrix (batch levels in column 1) and need to test whether a specific metabolite — e.g. citrate — still shows batch-associated intensity shifts after correction.

## When NOT to use

- The input is an unprocessed instrument export (e.g. raw XCMS m/z table) rather than a normalized, log2-scaled, batch-labeled .csv — dbnorm's regression assumes pre-processed input.
- The goal is choosing among correction models across all metabolites at once rather than diagnosing one specific feature; use dbnormSCORE's score plot over the full variable matrix instead.
- The dataset has more than 2000 features, where dbnorm's own guidance says its graphical/regression functions become computationally slow.

## Inputs

- batch-corrected metabolomics intensity matrix as .csv, samples in rows, features in columns, batch levels in the first column, log2-scaled
- raw (uncorrected) SKIPOGH-style intensity matrix of the same layout, for pre-correction baseline
- target metabolite feature identifier (e.g. citrate column name)

## Outputs

- adjusted R-squared of the metabolite ~ batch regression (residual batch-association statistic)
- one-way ANOVA confirmation table (citrate ~ batch)
- per-metabolite CSV result file (e.g. batch_association_citrate.csv with metabolite name, adj-R2, batch regression)

## How to apply

Load the corrected SKIPOGH-style matrix as .csv with samples in rows, features in columns, and batch levels in the first column (data must already be normalized and log2-scaled per the dbnorm README). Extract the target metabolite column (e.g. citrate), drop the batch column from the feature matrix, and fit stats::lm(intensity ~ batch_factor) in R; optionally confirm with one-way ANOVA of citrate ~ batch. The adjusted R-squared of the batch term is the residual batch-association metric — dbnorm uses exactly this statistic (adj-R2, range 0 to 1) to score model performance. Compare against the correction model's reported maximum residual variability: ber and parametric ComBat achieved < 0.01 (1%) on SKIPOGH, while lowess left a maximum of 0.78 (78%) and nonparametric ComBat 0.60 (60%), so a citrate adj-R2 near 0.78 flags lowess as the failing model. Write per-metabolite results (metabolite name, adjusted R-squared, batch regression) to CSV for model-selection decisions, e.g. batch_association_citrate.csv.

## Related tools

- **R** (execution environment for stats::lm regression of metabolite intensity on batch factor and one-way ANOVA)
- **dbnorm** (R package that supplies the corrected matrices (via Visodbnorm/ProfPlot* functions) and whose adj-R2 scoring framework this per-metabolite regression replicates) — https://github.com/NBDZ/dbnorm

## Examples

```
Rscript -e 'd <- read.csv("skipogh_lowess.csv", row.names = 1); batch <- factor(d[,1]); fit <- lm(d$citrate ~ batch); summary(fit)$adj.r.squared'
```

## Evaluation signals

- Adjusted R-squared must lie in [0, 1]; a negative value signals a mis-specified lm model (e.g. batch not treated as a factor).
- On SKIPOGH lowess-corrected data, the maximum residual variability for any metabolite was 0.78 — a citrate adj-R2 close to 0.78 corroborates the article's finding; values < 0.01 match ber/parametric ComBat instead.
- Result CSV must contain one row per metabolite with columns: metabolite name, adjusted R-squared, batch regression term.
- The batch factor in the lm formula must correspond exactly to the first column of the input .csv (dbnorm's documented layout).
- ANOVA p-value and lm adj-R2 should be directionally consistent (both near-zero, or both large) for the same metabolite.

## Limitations

- Regression quantifies association but does not itself remove the residual drift; a high adj-R2 only diagnoses the correction model as inadequate for that feature.
- dbnorm's regression/visualization functions are recommended for fewer than 2000 features; performance degrades on larger feature matrices.
- Input must already be log2-scaled and normalized — running the regression on unscaled raw intensities can mask high-abundance-feature heterogeneity.
- Per-metabolite regression must be run feature-by-feature; dbnormSCORE's global score is based on the maximum adj-R2 across variables, not on any single metabolite of interest.

## Evidence

- [results] lowess-corrected SKIPOGH data showed maximum residual variability of 0.78: "the maximum variability detected for a metabolite was 0.78 (78%) for the lowess-corrected dataset (Supplementary Table 2)"
- [results] ber and parametric ComBat leave < 0.01 residual batch variability: "the datasets corrected by ber and parametric Com Bat functions presented a similar performance in this study, with a maximum variability associated with the batch < 0.01 (1%)"
- [results] dbnorm score is based on highest adjusted-R2: [evidence span withheld — anchor status: not_found]
- [abstract] dbnorm assigns adj-R2 (0 to 1) score: [evidence span withheld — anchor status: not_found]
- [readme] README: input must be log2-scaled .csv with batch in first column: "Data to be uploaded must be normalized and scaled on the log2 to account for the high abundance features (variables) by which technical heterogeneity might be overlooked."
- [readme] README: batch levels in first column of .csv: "The input data must be in **.csv** format with the independent experiments in the rows and the features (variables) in the columns, with the `batch` levels considered in the first column."
