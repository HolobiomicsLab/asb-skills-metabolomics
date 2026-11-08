---
name: fdr-multiple-testing-correction
description: Use when running GetFeatistics significance functions (gentab_P.t.test,
  one/two-way ANOVA, gentab_lm_long) over many metabolites, where per-feature p-values
  must be FDR-adjusted before reporting or building volcano plots.
when_to_use_negative:
- Input is raw feature intensities that have not yet been transformed and tested (transf_data,
  then the univariate/regression functions, must run first)
- A single pre-specified hypothesis test where no multiplicity correction is needed
- The table already contains an FDR_Pvalue column from a prior run — re-running would
  double-adjust
edam_operation: http://edamontology.org/operation_0492
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_3173
tools:
- name: GetFeatistics
  role: R-package whose univariate and regression functions (gentab_P.t.test, ANOVA
    tables, gentab_lm_long) return p-value tables that are FDR-corrected within the
    same output table
  repo: https://github.com/FrigerioGianfranco/GetFeatistics
- name: R
  role: Language in which the FDR correction and surrounding statistical functions
    are implemented and run
- name: ggplot
  role: Used to visualise the FDR-corrected tables (e.g. Volcano_lm, Volcano_ttest_FC)
    as customisable plots
provenance:
  source_task_ids:
  - task_008
  - task_006
  - task_005
  source_papers:
  - doi: 10.1515/jib-2025-0047
    title: 'Streamlining feature elaboration and statistics analysis in metabolomics:
      the GetFeatistics R-package'
  source_content_hash: sha256:7ba655e8cca3310c085f0e24cafd113227cc24e91f27b93d0163909d13dbeafd
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/FrigerioGianfranco/GetFeatistics
  tools:
  - AER
  - GetFeatistics
  - R
  - ggplot
  - lmerTest
  edam_operation: http://edamontology.org/operation_0492
  edam_topics:
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_3173
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 13
---

# fdr-multiple-testing-correction

## Summary

False discovery rate (FDR) correction of p-value tables produced across the metabolomics-wide tests implemented in the GetFeatistics R-package (t-tests, ANOVA with post-hoc Tukey HSD, and per-metabolite regression via gentab_lm_long), so that reported p-values are adjusted for the large number of simultaneous metabolite–covariate tests.

## When to use

Use when running GetFeatistics significance functions (gentab_P.t.test, one/two-way ANOVA, gentab_lm_long) over many metabolites, where per-feature p-values must be FDR-adjusted before reporting or building volcano plots.

## When NOT to use

- Input is raw feature intensities that have not yet been transformed and tested (transf_data, then the univariate/regression functions, must run first)
- A single pre-specified hypothesis test where no multiplicity correction is needed
- The table already contains an FDR_Pvalue column from a prior run — re-running would double-adjust

## Inputs

- Table of raw p-values from gentab_P.t.test, gentab_P.1wayANOVA_posthocTukeyHSD, gentab_P.2wayANOVA_posthocTukeyHSD, or gentab_lm_long
- gentab_lm_long tidy result table with columns dependent/independent, beta, 95% CI, SE, adjusted R-squared, P-value, variation_perc per metabolite–covariate pair

## Outputs

- Tidy table with an FDR_Pvalue (adjusted p-value) column aligned to each metabolite–covariate or metabolite–comparison test
- FDR-corrected table usable as input for Volcano_lm / Volcano_ttest_FC ggplot visualisations

## How to apply

After fitting the underlying tests — pairwise t-tests, one- or two-way ANOVA with post-hoc Tukey HSD, or one regression model per metabolite (lm for fixed effects, lmerTest for mixed-effects longitudinal data, AER's tobit for values censored at limits of detection) — collect the raw P-values in the function's output table and apply FDR correction across the metabolite–covariate (or metabolite–comparison) family, appending an FDR_Pvalue column. In gentab_lm_long the table already carries beta, 95% CI, SE, adjusted R-squared, P-value, and variation_perc per pair; the FDR step completes the documented schema (dependent/independent, beta, 95% CI, SE, adjusted R-squared, P-value, FDR_Pvalue, variation_perc). The corrected table then feeds downstream reporting, e.g. the Volcano_lm ggplot visualisation. Rationale: metabolomics workflows test hundreds of features simultaneously, so uncorrected p-values would inflate false positives in the concise, informative output tables the package produces.

## Related tools

- **GetFeatistics** (R-package whose univariate and regression functions (gentab_P.t.test, ANOVA tables, gentab_lm_long) return p-value tables that are FDR-corrected within the same output table) — https://github.com/FrigerioGianfranco/GetFeatistics
- **R** (Language in which the FDR correction and surrounding statistical functions are implemented and run)
- **ggplot** (Used to visualise the FDR-corrected tables (e.g. Volcano_lm, Volcano_ttest_FC) as customisable plots)

## Evaluation signals

- The output table contains exactly the documented schema columns: dependent/independent, beta, 95% CI, SE, adjusted R-squared, P-value, FDR_Pvalue, variation_perc
- Every FDR_Pvalue is >= its corresponding raw P-value and all corrected values lie in [0, 1]
- FDR_Pvalue is present and non-NA for every metabolite–covariate pair in the table
- The script executes without error for all modes (lm, lmer, tobit) and the corrected table renders into a Volcano_lm plot
- Downstream pairwise tables (gentab_P.t.test with FDR, ANOVA post-hoc Tukey HSD tables) show the same number of rows before and after correction

## Limitations

- The Tobit mode is specifically for censoring values exceeding the limits of detection; applying FDR over tables mixing incompatible model families should be avoided
- GetFeatistics requires R >= 4.3.1 and Java matching the R architecture before any of the statistical functions can be run
- FDR correction is applied across the family of tests within a table; the package does not prescribe cross-table correction, so combining results from separate runs needs care

## Evidence

- [abstract] streamlined regression modelling with fixed effects, mixed-effects models for longitudinal data, and Tobit regression for censoring values exceeding the limits of detection: "streamlined regression modelling with fixed effects, mixed-effects models for longitudinal data, and Tobit regression for censoring values exceeding the limits of detection"
- [abstract] Output tables are concise and informative, facilitating interpretation and reporting, while output visualisations are fully customisable via the ggplot grammar.: "Output tables are concise and informative, facilitating interpretation and reporting, while output visualisations are fully customisable via the ggplot grammar."
- [other] gentab_lm_long fits one model per metabolite (lm, lmer, or Tobit via AER) and outputs a table with effect estimates, confidence intervals, and p-values (with FDR correction) for each metabolite–covariate pair.: [evidence span withheld — anchor status: not_found]
- [other] Apply false discovery rate (FDR) correction across the P-values to obtain adjusted P-values in the table.: [evidence span withheld — anchor status: not_found]
- [other] verify each column of the result table (dependent/independent, beta, 95% CI, SE, adjusted R-squared, P-value, FDR_Pvalue, variation_perc) against the schema documented: [evidence span withheld — anchor status: not_found]
