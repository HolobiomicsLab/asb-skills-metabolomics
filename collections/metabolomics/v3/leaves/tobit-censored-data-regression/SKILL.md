---
name: tobit-censored-data-regression
description: Use when metabolomics intensities contain values censored at detection
  limits (requiring Tobit) or repeated measures requiring mixed effects, and per-metabolite
  effect estimates with FDR-corrected p-values are needed.
when_to_use_negative:
- The input is already a processed statistical result table rather than raw metabolite
  intensities with sample metadata
- The outcome is not censored and there are no repeated measures, so a plain fixed-effects
  lm suffices without the Tobit/mixed-effect machinery
- The goal is untargeted feature extraction or pooled QC filtration from raw mass
  spectrometry files rather than statistical regression on an existing feature table
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_3172
tools:
- name: GetFeatistics
  role: R-package providing the regression mechanism (lm, lmer, Tobit) and tidy-table
    output for per-metabolite modelling
  repo: https://github.com/FrigerioGianfranco/GetFeatistics
- name: AER
  role: R package used as the engine for fitting Tobit models when metabolite values
    are censored at detection limits
- name: R
  role: Language in which the GetFeatistics regression modelling is implemented (requires
    R ≥ 4.3.1)
  repo: https://github.com/FrigerioGianfranco/GetFeatistics
- name: ggplot
  role: Grammar used for customisable output visualisations such as the Volcano_lm
    plot built from the regression table
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1515/jib-2025-0047
    title: 'Streamlining feature elaboration and statistics analysis in metabolomics:
      the GetFeatistics R-package'
  source_content_hash: sha256:17af1760db5225326f4497eb11eb87980d87179b51490ece1dd492f76c9ddd45
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/FrigerioGianfranco/GetFeatistics
  tools:
  - AER
  - GetFeatistics
  - R
  - ggplot
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# tobit-censored-data-regression

## Summary

Fits one regression model per metabolite from a long-format metabolomics feature table using the GetFeatistics regression mechanism, selecting lm for fixed effects, lmer for mixed-effects longitudinal data, or Tobit regression (via AER) when values are censored at limits of detection.

## When to use

Use when metabolomics intensities contain values censored at detection limits (requiring Tobit) or repeated measures requiring mixed effects, and per-metabolite effect estimates with FDR-corrected p-values are needed.

## When NOT to use

- The input is already a processed statistical result table rather than raw metabolite intensities with sample metadata
- The outcome is not censored and there are no repeated measures, so a plain fixed-effects lm suffices without the Tobit/mixed-effect machinery
- The goal is untargeted feature extraction or pooled QC filtration from raw mass spectrometry files rather than statistical regression on an existing feature table

## Inputs

- Long-format metabolomics feature table with per-sample metabolite intensities
- Sample metadata (covariates, grouping, time points for longitudinal designs)
- User-specified regression model formula
- Limits of detection for metabolites with censored values

## Outputs

- Tidy table of effect estimates (beta, 95% CI, standard error, adjusted R-squared, variation percentage) per metabolite–covariate pair
- FDR-adjusted p-values appended to the table
- Volcano_lm customisable ggplot visualisation derived from the table

## How to apply

Load the long-format metabolomics feature table with intensities, sample metadata, and the user-specified model formula into R with the GetFeatistics package. For each dependent variable (metabolite), fit the regression model through the GetFeatistics regression mechanism, choosing the lm engine for fixed-effects designs, lmer for mixed-effects longitudinal data, or a Tobit model via the AER package when censored values exceed the limits of detection. Extract beta, 95% CI, standard error, adjusted R-squared, p-value, and variation percentage from each fitted model and assemble them into a concise, informative tidy table. Apply false discovery rate (FDR) correction across the p-values to obtain adjusted p-values, then feed the tidy table into downstream visualisations such as the Volcano_lm ggplot plot. The package was built precisely because multiple regression models with mixed-effect and Tobit models are a key statistical need in metabolomics pipelines.

## Related tools

- **GetFeatistics** (R-package providing the regression mechanism (lm, lmer, Tobit) and tidy-table output for per-metabolite modelling) — https://github.com/FrigerioGianfranco/GetFeatistics
- **AER** (R package used as the engine for fitting Tobit models when metabolite values are censored at detection limits)
- **R** (Language in which the GetFeatistics regression modelling is implemented (requires R ≥ 4.3.1)) — https://github.com/FrigerioGianfranco/GetFeatistics
- **ggplot** (Grammar used for customisable output visualisations such as the Volcano_lm plot built from the regression table)

## Evaluation signals

- The output table contains one row per metabolite–covariate pair with beta, 95% CI, standard error, adjusted R-squared, p-value, and variation percentage columns
- P-values in the final table carry FDR adjustment across the multiple metabolite models
- Tobit engine was selected only for metabolites with values censored at limits of detection; lm or lmer otherwise
- The tidy table can be passed directly to Volcano_lm to produce a valid customisable ggplot
- Mixed-effect (lmer) fits are used for longitudinal/repeated-measures data and include the specified random-effects structure

## Limitations

- Tobit regression is only appropriate when censoring is at the limits of detection; applying it to uncensored intensities is not justified
- FDR correction is applied across the panel of metabolite models, so interpretation of individual metabolites depends on the multiplicity burden
- The article documents the regression capability at a high level; concrete function signatures are documented in the package reference/vignette (https://frigeriogianfranco.github.io/GetFeatistics/reference/)

## Evidence

- [abstract] Tobit regression for censoring values exceeding the limits of detection: "streamlined regression modelling with fixed effects, mixed-effects models for longitudinal data, and Tobit regression for censoring values exceeding the limits of detection"
- [discussion] Efficient construction of mixed-effect and Tobit models: "the efficient construction of multiple regression models with mixed-effect and Tobit models"
- [readme] README description of advanced statistics including multiple regression with mixed effects: "advanced statistics such as multiple regression linear models with mixed effects"
