---
name: tobit-censored-regression-with-detection-limits
description: Use when metabolite intensity values are censored because they fall below/above
  limits of detection and you need regression associations that correctly handle censoring
  rather than discarding or imputing those values.
when_to_use_negative:
- Intensity data contain no censored values (all measurements within the linear detection
  range) — the fixed-effects linear-model mode of gentab_lm_long is the simpler correct
  choice
- The goal is feature extraction or QC-based feature filtration from raw MS data (e.g.
  with XCMS or patRoon import) rather than statistical association modelling
- Input is a raw, unprocessed feature table — pre-processing (missing values, normality)
  must precede the regression step
edam_operation: http://edamontology.org/operation_0365
edam_topics:
- http://edamontology.org/topic_2269
tools:
- name: GetFeatistics
  role: R-package providing gentab_lm_long, which wraps AER's tobit regression for
    censored values and also offers fixed-effects and lmerTest-based mixed-effects
    modes
  repo: https://github.com/FrigerioGianfranco/GetFeatistics
- name: R
  role: Language in which GetFeatistics is implemented and executed (version >= 4.3.1)
  repo: https://github.com/FrigerioGianfranco/GetFeatistics
- name: lmerTest
  role: Wrapped by gentab_lm_long for the mixed-effects longitudinal regression mode
    (comparison mode to Tobit)
- name: AER
  role: Provides the tobit regression that gentab_lm_long calls in Tobit mode with
    left/right censoring limits
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1515/jib-2025-0047
    title: 'Streamlining feature elaboration and statistics analysis in metabolomics:
      the GetFeatistics R-package'
  source_content_hash: sha256:1efbe4c84b094a1ed082144ecc1d9ca5c5ebbe8fb468d5678f20adfedc4f1833
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/FrigerioGianfranco/GetFeatistics
  tools:
  - AER
  - GetFeatistics
  - R
  - lmerTest
  edam_operation: http://edamontology.org/operation_0365
  edam_topics:
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# Tobit censored regression with detection limits

## Summary

Fit Tobit regression models for metabolomics intensities that are censored at detection limits, using the GetFeatistics package (which wraps AER's tobit) alongside its fixed-effects and mixed-effects regression modes, returning effect estimates, CIs, and p-values per molecule–covariate pair.

## When to use

Use when metabolite intensity values are censored because they fall below/above limits of detection and you need regression associations that correctly handle censoring rather than discarding or imputing those values.

## When NOT to use

- Intensity data contain no censored values (all measurements within the linear detection range) — the fixed-effects linear-model mode of gentab_lm_long is the simpler correct choice
- The goal is feature extraction or QC-based feature filtration from raw MS data (e.g. with XCMS or patRoon import) rather than statistical association modelling
- Input is a raw, unprocessed feature table — pre-processing (missing values, normality) must precede the regression step

## Inputs

- Cleaned and transformed metabolomics feature-intensity table from the GetFeatistics vignette
- Detection-limit (censoring) values for the targeted molecules, as specified in the vignette
- Study-design variables (predictors/covariates; study terms for fixed and random effects)

## Outputs

- Per-molecule-covariate regression table with columns: dependent/independent, beta, 95% CI, SE, adjusted R-squared, P-value, FDR_Pvalue, variation_perc
- gentab_lm_long_results.csv combining the fixed-effects, mixed-effects, and Tobit mode outputs

## How to apply

Install GetFeatistics from GitHub (requires R >= 4.3.1, matching-architecture Java, and git) via devtools::install_github with dependencies = TRUE. Load the vignette's cleaned and transformed metabolomics table, then run gentab_lm_long in Tobit mode, wrapping AER's tobit regression with left and right censoring limits set to the detection-limit values specified in the vignette for the targeted molecules. For comparison, the same function also supports fixed-effects linear models and mixed-effects longitudinal models via lmerTest. Combine the mode outputs and verify the result table against the documented schema (dependent/independent, beta, 95% CI, SE, adjusted R-squared, P-value, FDR_Pvalue, variation_perc) before exporting to CSV.

## Related tools

- **GetFeatistics** (R-package providing gentab_lm_long, which wraps AER's tobit regression for censored values and also offers fixed-effects and lmerTest-based mixed-effects modes) — https://github.com/FrigerioGianfranco/GetFeatistics
- **R** (Language in which GetFeatistics is implemented and executed (version >= 4.3.1)) — https://github.com/FrigerioGianfranco/GetFeatistics
- **lmerTest** (Wrapped by gentab_lm_long for the mixed-effects longitudinal regression mode (comparison mode to Tobit))
- **AER** (Provides the tobit regression that gentab_lm_long calls in Tobit mode with left/right censoring limits)

## Examples

```
devtools::install_github("FrigerioGianfranco/GetFeatistics", dependencies = TRUE); library(GetFeatistics)
```

## Evaluation signals

- The output table contains exactly the documented columns: dependent/independent, beta, 95% CI, SE, adjusted R-squared, P-value, FDR_Pvalue, variation_perc
- The script executes without error across all three modes (fixed effects, mixed effects, Tobit) — script_runs check
- Field-present checks confirm every documented schema column is populated in gentab_lm_long_results.csv
- Censoring limits passed to the Tobit mode match the detection-limit values specified in the vignette for the targeted molecules
- Each metabolite-covariate pair yields effect estimate, confidence interval, and p-value as documented

## Limitations

- Tobit mode requires explicit detection-limit values from the vignette; if censoring limits are unknown or misspecified, censored-value handling is invalid
- The function's output is oriented to metabolomics regression reporting; columns like variation_perc and FDR_Pvalue assume a multiple-testing metabolomics context
- Java architecture must match R's (64-bit or 32-bit) for installation to succeed; installing with dependencies = TRUE can take considerable time

## Evidence

- [abstract] streamlined regression modelling with fixed effects, mixed-effects models for longitudinal data, and Tobit regression for censoring values exceeding the limits of detection: "streamlined regression modelling with fixed effects, mixed-effects models for longitudinal data, and Tobit regression for censoring values exceeding the limits of detection"
- [discussion] the efficient construction of multiple regression models with mixed-effect and Tobit models: "the efficient construction of multiple regression models with mixed-effect and Tobit models"
- [readme] advanced statistics such as multiple regression linear models with mixed effects: "advanced statistics such as multiple regression linear models with mixed effects"
- [abstract] GetFeatistics is publicly available on GitHub: "GetFeatistics is publicly available on GitHub"
