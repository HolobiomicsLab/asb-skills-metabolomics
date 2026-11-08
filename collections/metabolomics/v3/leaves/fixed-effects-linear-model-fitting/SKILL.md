---
name: fixed-effects-linear-model-fitting
description: Use when the input is a cleaned/transformed metabolomics feature table
  of targeted molecule intensities and the goal is regressing each metabolite on specified
  covariates with fixed effects only (no random effects, no censored values).
when_to_use_negative:
- The study design is longitudinal and requires random effects — use gentab_lm_long's
  mixed-effects mode (lmerTest) instead.
- Response values are censored at limits of detection — use the Tobit mode (AER::tobit)
  with left/right censoring limits instead.
- The input is raw, unprocessed intensities that have not yet been cleaned/transformed
  per the vignette workflow.
edam_operation: http://edamontology.org/operation_0306
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3520
tools:
- name: GetFeatistics
  role: R-package providing gentab_lm_long for fixed-effects (and mixed-effects/Tobit)
    regression over metabolite-covariate pairs
  repo: https://github.com/FrigerioGianfranco/GetFeatistics
- name: R
  role: runtime language for executing GetFeatistics functions
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1515/jib-2025-0047
    title: 'Streamlining feature elaboration and statistics analysis in metabolomics:
      the GetFeatistics R-package'
  source_content_hash: sha256:e8e09f838e507c8ff9c7031cdf54acc732b93fe2488f0a2b26ef8f58f9b1689f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/FrigerioGianfranco/GetFeatistics
  tools:
  - AER
  - GetFeatistics
  - R
  - lmerTest
  edam_operation: http://edamontology.org/operation_0306
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# fixed-effects-linear-model-fitting

## Summary

Fit fixed-effects linear regression models per metabolite using GetFeatistics' gentab_lm_long on cleaned, transformed targeted-metabolomics intensity tables, producing effect estimates (beta, 95% CI, SE), adjusted R-squared, P-values and FDR-adjusted P-values for each molecule–covariate pair.

## When to use

Use when the input is a cleaned/transformed metabolomics feature table of targeted molecule intensities and the goal is regressing each metabolite on specified covariates with fixed effects only (no random effects, no censored values).

## When NOT to use

- The study design is longitudinal and requires random effects — use gentab_lm_long's mixed-effects mode (lmerTest) instead.
- Response values are censored at limits of detection — use the Tobit mode (AER::tobit) with left/right censoring limits instead.
- The input is raw, unprocessed intensities that have not yet been cleaned/transformed per the vignette workflow.

## Inputs

- cleaned and transformed metabolomics feature/intensity table for targeted molecules (as prepared per the GetFeatistics vignette)
- regression model specification: response molecule intensities and fixed-effect predictor covariates/intercept

## Outputs

- long-format regression results table with columns: dependent/independent, beta, 95% CI, SE, adjusted R-squared, P-value, FDR_Pvalue, variation_perc
- CSV export of verified results (e.g. gentab_lm_long_results.csv)

## How to apply

Install the GetFeatistics R-package (requires R ≥ 4.3.1, Java matching R's architecture, and git) via devtools::install_github, then load it with library(GetFeatistics). Load the vignette's cleaned and transformed example metabolomics table using the package's documented data-loading commands. Run gentab_lm_long in its fixed-effects linear-model mode, regressing each targeted molecule's intensity on the specified predictors and intercept, as distinct from its mixed-effects mode (which wraps lmerTest for longitudinal designs) and its Tobit mode (which uses AER::tobit with left/right censoring limits set to detection-limit values). Verify the output table column schema — dependent/independent, beta, 95% CI, SE, adjusted R-squared, P-value, FDR_Pvalue, variation_perc — against the GetFeatistics documentation before writing results (e.g. gentab_lm_long_results.csv). This mode is one of three regression modelling modes the package offers alongside mixed-effects and Tobit models, part of the package's rationale of efficiently constructing multiple regression models across many metabolites.

## Related tools

- **GetFeatistics** (R-package providing gentab_lm_long for fixed-effects (and mixed-effects/Tobit) regression over metabolite-covariate pairs) — https://github.com/FrigerioGianfranco/GetFeatistics
- **R** (runtime language for executing GetFeatistics functions)

## Examples

```
devtools::install_github("FrigerioGianfranco/GetFeatistics", dependencies = TRUE); library(GetFeatistics)
```

## Evaluation signals

- Output table contains exactly the documented schema columns: dependent/independent, beta, 95% CI, SE, adjusted R-squared, P-value, FDR_Pvalue, variation_perc (field_present check).
- Script executes without error for all three gentab_lm_long modes (script_runs check).
- One row per metabolite-covariate pair with beta, 95% CI and SE populated for each targeted molecule.
- FDR_Pvalue column is present and adjusted relative to the raw P-value column.
- Model ran in fixed-effects mode: no random-effects terms and no censoring limits were specified.

## Limitations

- Fixed-effects mode does not handle longitudinal random effects — mixed-effects modelling requires the lmerTest-backed mode.
- Censoring at limits of detection is not handled in fixed-effects mode; Tobit mode with explicit left/right censoring limits is required.
- Package requires R ≥ 4.3.1 and Java with the same architecture as R (64-bit or 32-bit); installation of all dependencies may take time.

## Evidence

- [abstract] streamlined regression modelling with fixed effects, mixed-effects models for longitudinal data, and Tobit regression for censoring values exceeding the limits of detection: "streamlined regression modelling with fixed effects, mixed-effects models for longitudinal data, and Tobit regression for censoring values exceeding the limits of detection"
- [discussion] the efficient construction of multiple regression models with mixed-effect and Tobit models: "the efficient construction of multiple regression models with mixed-effect and Tobit models"
- [readme] Getting streamlined elaboration of targeted and non-targeted metabolomics data, including elaboration of feature tables, separate QC processing, advanced statistics such as multiple regression linear models with mixed effects: "advanced statistics such as multiple regression linear models with mixed effects"
