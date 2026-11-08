---
name: covariate-adjustment-confounder-control
description: Use when scanning ~249 NMR metabolic biomarkers (plus derived ratios)
  for associations with incident CAD or stroke in UK Biobank, and results must be
  adjusted for age and sex with prevalent cases and lipid-lowering medication users
  excluded.
when_to_use_negative:
- When estimating effects of lipid-lowering medication itself — the skill excludes
  medication users as a confounder-control strategy rather than modeling medication
  as an exposure.
- When running remove_technical_variation() itself — the README warns that no prefiltering
  of samples or columns should be performed before that function; exclusion steps
  belong to downstream association analysis.
- For association models that require multivariable adjustment across many correlated
  biomarkers — the article notes strong inter-correlation structure challenges multivariable
  modelling, and this skill only adjusts per-biomarker models for age and sex.
tools:
- name: ukbnmr
  role: Supplies extract_biomarkers() for original concentrations, remove_technical_variation()
    for post-QC concentrations, and recompute_derived_biomarkers() so ratios/percentages
    analysed after covariate adjustment are recomputed from adjusted non-derived biomarkers.
  repo: https://github.com/sritchie73/ukbnmr
- name: R
  role: Language in which the per-biomarker Cox proportional hazards models (survival
    analysis) and covariate-adjusted association scan are implemented.
- name: data.table
  role: Used with fread()/fwrite() to load exported UK Biobank field data and write
    the biomarker-endpoint-version association results table.
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41597-023-01949-y
    title: Quality control and removal of technical variation of NMR metabolic biomarker
      data in ~120,000 UK Biobank participants
  source_content_hash: sha256:0caaa44122310a52f72ad71639ab5b42f59d9bb7b1497426ca02455a1d989ff3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/sritchie73/ukbnmr
  tools:
  - R
  - data.table
  - ukbnmr
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# covariate-adjustment-confounder-control

## Summary

Adjusts biomarker-disease association analyses in the UK Biobank NMR metabolomics data for core demographic confounders (age, sex) and removes the major systematic biological influence of lipid-lowering medication use by exclusion, when scanning biomarker associations with incident coronary artery disease and stroke via Cox proportional hazards models.

## When to use

Use when scanning ~249 NMR metabolic biomarkers (plus derived ratios) for associations with incident CAD or stroke in UK Biobank, and results must be adjusted for age and sex with prevalent cases and lipid-lowering medication users excluded.

## When NOT to use

- When estimating effects of lipid-lowering medication itself — the skill excludes medication users as a confounder-control strategy rather than modeling medication as an exposure.
- When running remove_technical_variation() itself — the README warns that no prefiltering of samples or columns should be performed before that function; exclusion steps belong to downstream association analysis.
- For association models that require multivariable adjustment across many correlated biomarkers — the article notes strong inter-correlation structure challenges multivariable modelling, and this skill only adjusts per-biomarker models for age and sex.

## Inputs

- Original UK Biobank NMR biomarker data extracted with ukbnmr::extract_biomarkers (data.frame, one row per eid x visit_index)
- Post-QC NMR biomarker data from ukbnmr::remove_technical_variation
- Incident CAD and stroke event records with follow-up time (truncated at 1 February 2020)
- Covariates: age and sex; lipid-lowering medication use; prevalent CAD/stroke status

## Outputs

- Per-biomarker Cox model results table (hazard ratios, 95% confidence intervals, P-values) for every biomarker-endpoint-version combination, written via data.table::fwrite
- Within-endpoint, within-version rankings of biomarker associations
- Analytic cohort definitions (exclusions applied, N per endpoint, case counts)

## How to apply

Assemble the analytic cohort per endpoint: exclude participants with prevalent CAD or stroke and users of lipid-lowering medication, truncate follow-up at 1 February 2020 (N=95,790 CAD samples with 2,717 incident cases; N=95,623 stroke samples with 1,366 cases), and attach age and sex as covariates. For each of the 249 biomarkers plus derived ratios, fit a Cox proportional hazards model per SD of concentration adjusted for age and sex, separately on the original (via ukbnmr::extract_biomarkers) and post-QC (via ukbnmr::remove_technical_variation) concentrations, using survival analysis in R. Adjusted biomarkers can carry new unwanted variation into ratios, percentages, and composites, so derived biomarkers should be recomputed from adjusted non-derived parts (ukbnmr::recompute_derived_biomarkers). Rationale: sex, body mass index, and lipid-lowering medication are major systematic biological sources of variation in the NMR data, so medication users are excluded and age/sex are entered as model covariates. Collect hazard ratios, 95% CIs and P-values per biomarker-endpoint-version combination, then rank within endpoint and version to compare pre- vs post-QC signal (post-QC: albumin strongest for stroke; pre-QC removal: percentage of LDL lipids composed of cholesterol HR 0.89, 95% CI 0.86-0.91, P = 2 × 10−15).

## Related tools

- **ukbnmr** (Supplies extract_biomarkers() for original concentrations, remove_technical_variation() for post-QC concentrations, and recompute_derived_biomarkers() so ratios/percentages analysed after covariate adjustment are recomputed from adjusted non-derived biomarkers.) — https://github.com/sritchie73/ukbnmr
- **R** (Language in which the per-biomarker Cox proportional hazards models (survival analysis) and covariate-adjusted association scan are implemented.)
- **data.table** (Used with fread()/fwrite() to load exported UK Biobank field data and write the biomarker-endpoint-version association results table.)

## Examples

```
coxph(Surv(follow_up_time, incident_stroke) ~ biomarker_z + age + sex, data = cohort); fwrite(results, "cox_results.csv")
```

## Evaluation signals

- Cohort sizes reproduce the expected analytic sets: N=95,790 CAD samples with 2,717 incident cases and N=95,623 stroke samples with 1,366 cases after exclusions and follow-up truncation at 1 February 2020.
- Output table contains results for all biomarker-endpoint-version combinations (249 biomarkers plus derived ratios × CAD/stroke × original/post-QC).
- Post-QC stroke ranking confirms albumin as the strongest association, while the LDL cholesterol percentage (HR 0.89, 95% CI 0.86-0.91, P = 2e-15) is recovered as expected.
- Hazard ratios are reported per SD of biomarker concentration, with all models adjusted for both age and sex.
- Key results on original data are compared against post-QC results to check for discrepancies, as recommended by the authors.

## Limitations

- Filtering to non-derived biomarkers does not mitigate challenges introduced by collinearity, as many non-derived biomarkers have extremely strong inter-correlations, which impacts multiple testing strategy choices and multivariable modelling.
- Biomarker adjustment can introduce new unwanted variation into biomarkers that are ratios, percentages, or otherwise composed of multiple other biomarkers; the authors recommend filtering to the 107 non-derived biomarkers and re-deriving the rest post-adjustment.
- Regression on covariates may remove biological variation of interest, and assessing whether a QC procedure also removed biological variation is challenging and potentially intractable for a dataset as deeply phenotyped as UK Biobank.

## Evidence

- [discussion] Major systematic biological sources of variation include participant sex, body mass index, and use of lipid lowering medication: "major differences in most biomarker concentrations depending on participant sex, body mass index, and use of lipid lowering medication"
- [discussion] Recommendation to compare adjusted vs original results: "analysts utilizing the ukbnmr package may wish to compare their key results on the original data to check for and investigate any discrepancies."
- [discussion] Derived biomarkers can inherit adjustment artefacts: "biomarker adjustment can introduce new unwanted variation into biomarkers which are ratios, percentages, or otherwise composed of multiple other biomarkers"
- [readme] No prefiltering before remove_technical_variation: "the no prefiltering of samples or columns should be performed prior to running this function: the algorithms used for removing technical variation expect all the data to be present."
