---
name: longitudinal-mixed-effects-regression
description: Use when a metabolomics intensity table must be regressed per metabolite
  on study-design covariates, especially longitudinal/repeated-measures designs needing
  mixed effects, or intensities censored at detection limits.
when_to_use_negative:
- Data are raw intensities that have not been cleaned/transformed or QC-filtered —
  GetFeatistics expects pre-processed feature tables (e.g. pooled-QC filtered)
- The analysis is a single fixed-effect model with no longitudinal structure and no
  censoring — a plain lm outside this multi-mode mechanism is simpler
- The goal is untargeted feature extraction or annotation rather than statistical
  association testing (use patRoon/MS-DIAL import and QC-filtering functions instead)
edam_operation: http://edamontology.org/operation_0306
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: GetFeatistics
  role: Provides gentab_lm_long, the multi-mode per-metabolite regression mechanism
    (lm / lmer / Tobit) and the output table schema
  repo: github.com/FrigerioGianfranco/GetFeatistics
- name: R
  role: Runtime language for the package; README requires R version >= 4.3.1
- name: lmerTest
  role: Wrapped by gentab_lm_long to fit mixed-effects longitudinal models with fixed
    and random design terms
- name: AER
  role: Provides the tobit regression used for censoring values exceeding the limits
    of detection
- name: ggplot
  role: Grammar used for downstream customisable visualisations such as the Volcano_lm
    plot
provenance:
  source_task_ids:
  - task_006
  - task_008
  source_papers:
  - doi: 10.1515/jib-2025-0047
    title: 'Streamlining feature elaboration and statistics analysis in metabolomics:
      the GetFeatistics R-package'
  source_content_hash: sha256:f0fa3c4971feaddbef455eea9a44230818f3e5fe72c6c33e372d7b4bdc55cbc8
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - AER
  - GetFeatistics
  - R
  - ggplot
  - lmerTest
  edam_operation: http://edamontology.org/operation_0306
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8666666666666667
n_claims: 15
---

# longitudinal-mixed-effects-regression

## Summary

Fits regression models across a metabolomics feature table using the GetFeatistics gentab_lm_long mechanism, which supports fixed-effects lm, mixed-effects longitudinal models via lmerTest, and Tobit regression via AER for censoring at limits of detection, returning a tidy per-metabolite result table.

## When to use

Use when a metabolomics intensity table must be regressed per metabolite on study-design covariates, especially longitudinal/repeated-measures designs needing mixed effects, or intensities censored at detection limits.

## When NOT to use

- Data are raw intensities that have not been cleaned/transformed or QC-filtered — GetFeatistics expects pre-processed feature tables (e.g. pooled-QC filtered)
- The analysis is a single fixed-effect model with no longitudinal structure and no censoring — a plain lm outside this multi-mode mechanism is simpler
- The goal is untargeted feature extraction or annotation rather than statistical association testing (use patRoon/MS-DIAL import and QC-filtering functions instead)

## Inputs

- Long-format metabolomics feature table with molecule intensities and sample metadata
- User-specified model formula with fixed and (for mixed-effects mode) random effects terms
- Censoring / detection-limit values for targeted molecules (for Tobit mode)

## Outputs

- Tidy regression table with columns: dependent/independent, beta, 95% CI, SE, adjusted R-squared, P-value, FDR_Pvalue, variation_perc (e.g. gentab_lm_long_results.csv)
- Volcano_lm ggplot visualisation derived from the table

## How to apply

Load the cleaned, transformed feature table (long format: intensities plus sample metadata) and the user-specified model formula. For each dependent variable (molecule), call gentab_lm_long selecting the engine by data structure: lm for fixed effects, lmer (wrapped via lmerTest) when the study design includes random effects such as subject identifiers in longitudinal data, or AER's tobit with left/right censoring limits set to the molecules' limits of detection. Extract beta, 95% CI, SE, adjusted R-squared, P-value and variation percentage per metabolite–covariate pair, and apply FDR correction across P-values to produce FDR_Pvalue. The resulting tidy table can feed downstream ggplot-based reporting such as a Volcano_lm plot. Validate that the output contains exactly the documented columns (dependent/independent, beta, 95% CI, SE, adjusted R-squared, P-value, FDR_Pvalue, variation_perc) and that all three model modes execute without error.

## Related tools

- **GetFeatistics** (Provides gentab_lm_long, the multi-mode per-metabolite regression mechanism (lm / lmer / Tobit) and the output table schema) — github.com/FrigerioGianfranco/GetFeatistics
- **R** (Runtime language for the package; README requires R version >= 4.3.1)
- **lmerTest** (Wrapped by gentab_lm_long to fit mixed-effects longitudinal models with fixed and random design terms)
- **AER** (Provides the tobit regression used for censoring values exceeding the limits of detection)
- **ggplot** (Grammar used for downstream customisable visualisations such as the Volcano_lm plot)

## Examples

```
devtools::install_github("FrigerioGianfranco/GetFeatistics", dependencies = TRUE); library(GetFeatistics)
```

## Evaluation signals

- Output table contains exactly the documented columns: dependent/independent, beta, 95% CI, SE, adjusted R-squared, P-value, FDR_Pvalue, variation_perc
- Script executes without error in all three modes (fixed-effects, mixed-effects, Tobit)
- One row (or model) is produced per metabolite–covariate pair, with one model fitted per dependent variable
- FDR_Pvalue column is populated, confirming false-discovery-rate correction was applied across P-values
- Tobit mode uses censoring limits matching the vignette-specified detection limits for the targeted molecules

## Limitations

- Censored Tobit mode requires explicit left/right detection-limit values; without them censored intensities cannot be modelled correctly
- Mixed-effects mode depends on lmerTest and requires a correctly specified random-effects structure reflecting the longitudinal study design
- The paper describes the mechanism at a high level; per-function parameters should be verified against the package reference documentation and vignette

## Evidence

- [abstract] streamlined regression modelling with fixed effects, mixed-effects models for longitudinal data, and Tobit regression for censoring values exceeding the limits of detection: "streamlined regression modelling with fixed effects, mixed-effects models for longitudinal data, and Tobit regression for censoring values exceeding the limits of detection"
- [discussion] efficient construction of multiple regression models: "the efficient construction of multiple regression models with mixed-effect and Tobit models"
- [readme] advanced statistics such as multiple regression linear models with mixed effects: "advanced statistics such as multiple regression linear models with mixed effects"
- [abstract] Output tables are concise and informative, facilitating interpretation and reporting, while output visualisations are fully customisable via the ggplot grammar.: "Output tables are concise and informative, facilitating interpretation and reporting, while output visualisations are fully customisable via the ggplot grammar."
