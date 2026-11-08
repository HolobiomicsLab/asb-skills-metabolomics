---
name: metabolomics-regression-modelling
description: Use when a metabolomics intensity table plus sample metadata and a model
  formula require per-metabolite regression — fixed-effects lm, longitudinal mixed-effects
  (lmer), or Tobit for censored detection-limit values.
when_to_use_negative:
- Input is raw MS data needing feature extraction or pooled-QC feature filtration
  — gentab_lm_long operates downstream of XCMS/patRoon-style extraction and QC filtering
- Task is metabolite annotation or chemical-property retrieval (PubChem, PubChemLite,
  ClassyFire) — this skill covers only the regression-model construction stage
- No regression design is intended (no predictors/covariates and no censoring at LOD)
  — a descriptive or univariate screening step is the wrong framing here
edam_topics:
- http://edamontology.org/topic_2269
- http://edamontology.org/topic_3383
tools:
- name: GetFeatistics
  role: Provides gentab_lm_long, the function that fits lm/lmer/tobit models per metabolite
    and assembles the regression results table
  repo: github.com/FrigerioGianfranco/GetFeatistics
- name: R
  role: Language/runtime in which gentab_lm_long and the dependency stack (devtools,
    ggplot) execute
- name: lmerTest
  role: Engine wrapped by gentab_lm_long for mixed-effects longitudinal regression
    with fixed and random effects
- name: AER
  role: Engine wrapped by gentab_lm_long for Tobit regression on censored values exceeding
    the limits of detection
- name: ggplot
  role: Grammar behind Volcano_lm, the customisable visualisation built from the regression
    table
provenance:
  source_task_ids:
  - task_006
  - task_008
  source_papers:
  - doi: 10.1515/jib-2025-0047
    title: 'Streamlining feature elaboration and statistics analysis in metabolomics:
      the GetFeatistics R-package'
  source_content_hash: sha256:657a34c08bebf5e96e4975c52e39b66f1641150519b04c8752db0f2347a13397
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - AER
  - GetFeatistics
  - R
  - ggplot
  - lmerTest
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_2269
  - http://edamontology.org/topic_3383
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# metabolomics-regression-modelling

## Summary

Fits one regression model per metabolite on a long-format metabolomics feature table using GetFeatistics' gentab_lm_long, covering fixed-effects linear models (lm), mixed-effects longitudinal models (lmerTest), and Tobit regression (AER) for values censored at limits of detection, and emits a tidy results table plus a Volcano_lm ggplot visualisation.

## When to use

Use when a metabolomics intensity table plus sample metadata and a model formula require per-metabolite regression — fixed-effects lm, longitudinal mixed-effects (lmer), or Tobit for censored detection-limit values.

## When NOT to use

- Input is raw MS data needing feature extraction or pooled-QC feature filtration — gentab_lm_long operates downstream of XCMS/patRoon-style extraction and QC filtering
- Task is metabolite annotation or chemical-property retrieval (PubChem, PubChemLite, ClassyFire) — this skill covers only the regression-model construction stage
- No regression design is intended (no predictors/covariates and no censoring at LOD) — a descriptive or univariate screening step is the wrong framing here

## Inputs

- Long-format metabolomics feature table with metabolite intensities and sample metadata (cleaned/transformed per the GF vignette)
- Regression model formula/specification listing predictors, random-effect terms (for longitudinal designs), and covariates
- Left/right censoring limit values (detection limits) for targeted molecules when Tobit mode is needed

## Outputs

- Tidy regression table with columns: dependent/independent, beta, 95% CI, SE, adjusted R-squared, P-value, FDR_Pvalue, variation_perc
- gentab_lm_long_results.csv file
- Volcano_lm ggplot visualisation built from the tidy table

## How to apply

Load the vignette's cleaned, transformed long-format metabolomics table (intensities plus sample metadata) via GetFeatistics after installing it from GitHub with devtools. For each dependent molecule, run gentab_lm_long selecting the engine to match the design: lm for fixed-effects predictors, lmer (wrapping lmerTest) with the study design terms as fixed and random effects for longitudinal data, or tobit (via AER) with left/right censoring limits set to the vignette's detection-limit values when observations hit LOD. Extract beta, 95% CI, SE, adjusted R-squared, P-value, and variation_perc per metabolite–covariate pair, then apply FDR correction across the P-values to obtain the FDR_Pvalue column. Assemble the concise tidy table, write it to gentab_lm_long_results.csv, and optionally feed it to Volcano_lm, a customisable ggplot visualisation.

## Related tools

- **GetFeatistics** (Provides gentab_lm_long, the function that fits lm/lmer/tobit models per metabolite and assembles the regression results table) — github.com/FrigerioGianfranco/GetFeatistics
- **R** (Language/runtime in which gentab_lm_long and the dependency stack (devtools, ggplot) execute)
- **lmerTest** (Engine wrapped by gentab_lm_long for mixed-effects longitudinal regression with fixed and random effects)
- **AER** (Engine wrapped by gentab_lm_long for Tobit regression on censored values exceeding the limits of detection)
- **ggplot** (Grammar behind Volcano_lm, the customisable visualisation built from the regression table)

## Examples

```
devtools::install_github("FrigerioGianfranco/GetFeatistics", dependencies = TRUE); library(GetFeatistics)
```

## Evaluation signals

- Output table contains exactly the documented columns: dependent/independent, beta, 95% CI, SE, adjusted R-squared, P-value, FDR_Pvalue, variation_perc (field_present schema check)
- All three modes (lm fixed-effects, lmerTest mixed-effects, AER Tobit) execute without error (script_runs check)
- Tobit censoring limits match the detection-limit values specified in the GF vignette for the targeted molecules
- FDR-adjusted P-values present alongside raw P-values for every metabolite–covariate pair
- Table written to gentab_lm_long_results.csv and a Volcano_lm ggplot can be constructed from the tidy output

## Limitations

- Column schema of gentab_lm_long is fixed to the documented fields; downstream consumers must expect this exact schema
- Tobit mode is only appropriate when censoring values exceed the limits of detection, and requires LOD limits supplied from the vignette or design
- Package installation requires R >= 4.3.1, Java matching R's architecture, and git; dependencies may take substantial time to install
- Results depend on upstream pre-processing (missing-value handling and intensity-normality transformation) and pooled-QC filtration of features

## Evidence

- [abstract] streamlined regression modelling with fixed effects, mixed-effects models for longitudinal data, and Tobit regression for censoring values exceeding the limits of detection: "streamlined regression modelling with fixed effects, mixed-effects models for longitudinal data, and Tobit regression for censoring values exceeding the limits of detection"
- [abstract] Output tables are concise and informative, facilitating interpretation and reporting, while output visualisations are fully customisable via the ggplot grammar.: "Output tables are concise and informative, facilitating interpretation and reporting, while output visualisations are fully customisable via the ggplot grammar."
- [readme] advanced statistics such as multiple regression linear models with mixed effects: "advanced statistics such as multiple regression linear models with mixed effects"
- [discussion] the efficient construction of multiple regression models with mixed-effect and Tobit models: "the efficient construction of multiple regression models with mixed-effect and Tobit models"
