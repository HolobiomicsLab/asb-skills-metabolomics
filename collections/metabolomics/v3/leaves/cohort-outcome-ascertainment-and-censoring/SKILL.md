---
name: cohort-outcome-ascertainment-and-censoring
description: Use when running epidemiological biomarker association scans against
  incident disease endpoints in UK Biobank NMR data and you need a defined at-risk
  cohort with prevalent cases excluded and follow-up truncated to a fixed censoring
  date.
when_to_use_negative:
- Before running ukbnmr::remove_technical_variation — the README states no prefiltering
  of samples or columns should be performed prior to that function
- When the outcome is a single precomputed association (e.g. one biomarker-endpoint
  estimate) rather than a full 249-biomarker scan over both original and post-QC data
- When follow-up must extend past 1 February 2020 or when studying the full ~500,000
  participant release, since cohort sizes and censoring here are specific to the ~120,000-participant
  NMR phase
tools:
- name: ukbnmr
  role: Supplies extract_biomarkers() for original NMR concentrations and remove_technical_variation()
    for post-QC concentrations analyzed side by side
  repo: https://github.com/sritchie73/ukbnmr
- name: R (survival)
  role: Fits Cox proportional hazards models per SD of biomarker concentration adjusted
    for age and sex
- name: data.table
  role: Fast reading of exported field data and writing of the combined biomarker-endpoint-version
    results table via fwrite()
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41597-023-01949-y
    title: Quality control and removal of technical variation of NMR metabolic biomarker
      data in ~120,000 UK Biobank participants
  source_content_hash: sha256:6669c3ba7535c82f4b5fd3016e42c214209bc17bfa21d47eaa9da1e54a2ca4ee
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/sritchie73/ukbnmr
  tools:
  - R
  - R (survival)
  - data.table
  - ukbnmr
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# cohort-outcome-ascertainment-and-censoring

## Summary

Assembles UK Biobank analytic cohorts for incident coronary artery disease and stroke, excluding prevalent cases and lipid-lowering medication users, attaching age/sex covariates, and truncating follow-up at 1 February 2020 before biomarker association scanning.

## When to use

Use when running epidemiological biomarker association scans against incident disease endpoints in UK Biobank NMR data and you need a defined at-risk cohort with prevalent cases excluded and follow-up truncated to a fixed censoring date.

## When NOT to use

- Before running ukbnmr::remove_technical_variation — the README states no prefiltering of samples or columns should be performed prior to that function
- When the outcome is a single precomputed association (e.g. one biomarker-endpoint estimate) rather than a full 249-biomarker scan over both original and post-QC data
- When follow-up must extend past 1 February 2020 or when studying the full ~500,000 participant release, since cohort sizes and censoring here are specific to the ~120,000-participant NMR phase

## Inputs

- NMR biomarker concentrations from ukbnmr::extract_biomarkers (original data.frame with eid, visit_index, one column per biomarker)
- Post-QC biomarker data.frame from ukbnmr::remove_technical_variation ($biomarkers)
- UK Biobank CAD and stroke incidence records with event dates
- Age, sex, and lipid-lowering medication use fields
- Follow-up time derived from assessment date to 1 February 2020 or event

## Outputs

- Analytic cohort tables per endpoint (CAD: N=95,790, 2,717 incident cases; stroke: N=95,623, 1,366 incident cases)
- Single results table of hazard ratios, 95% confidence intervals, and P-values for all biomarker-endpoint-version combinations, written via data.table::fwrite
- Ranking of associations within each endpoint and data version

## How to apply

For each endpoint (incident CAD and incident stroke), exclude participants with prevalent disease and users of lipid-lowering medication, then attach age and sex as mandatory covariates. Truncate follow-up time at 1 February 2020 so that all biomarker versions (original via ukbnmr::extract_biomarkers and post-QC via ukbnmr::remove_technical_variation) are analyzed on identical person-time. Reproduce the published cohort sizes as a checkpoint: N=95,790 CAD samples with 2,717 incident cases and N=95,623 stroke samples with 1,366 cases. For each of the 249 biomarkers plus derived ratios, fit a Cox proportional hazards model (survival package in R) per SD of concentration adjusted for age and sex, separately on original and post-QC concentrations. Collect hazard ratios, 95% CIs, and P-values for every biomarker-endpoint-version combination into one data.table and write it with data.table::fwrite. Note the ordering: cohort assembly and Cox modeling happen after the raw-to-QC data preparation, and no sample prefiltering should be done before remove_technical_variation because its algorithms expect the complete data.

## Related tools

- **ukbnmr** (Supplies extract_biomarkers() for original NMR concentrations and remove_technical_variation() for post-QC concentrations analyzed side by side) — https://github.com/sritchie73/ukbnmr
- **R (survival)** (Fits Cox proportional hazards models per SD of biomarker concentration adjusted for age and sex)
- **data.table** (Fast reading of exported field data and writing of the combined biomarker-endpoint-version results table via fwrite())

## Examples

```
library(survival); fit <- coxph(Surv(followup_time, incident_stroke) ~ biomarker_sd + age + sex, data = cohort)
```

## Evaluation signals

- Cohort sizes match published values: N=95,790 CAD samples with 2,717 incident cases and N=95,623 stroke samples with 1,366 cases after exclusions and 1 February 2020 truncation
- Output table contains a result row for every biomarker-endpoint-version combination (249 biomarkers plus derived ratios × 2 endpoints × 2 data versions)
- Post-QC stroke ranking reproduces the expected top hits: percentage of LDL lipids composed of cholesterol with HR 0.89, 95% CI 0.86–0.91, P = 2×10⁻¹⁵
- Hazard ratios are interpretable as per-SD effects with all models adjusted for age and sex only
- Key results are also computed on the original data so original vs post-QC discrepancies can be checked, as recommended by the article

## Limitations

- The article recommends comparing key results on the original data to check for and investigate any discrepancies after using ukbnmr
- Biomarker adjustment can introduce new unwanted variation into ratios, percentages, or composite biomarkers, so derived biomarkers should be interpreted carefully
- Strong correlation structure among NMR biomarkers impacts multiple testing strategy choices and challenges multivariable modelling beyond the age/sex-adjusted per-biomarker scan

## Evidence

- [other] Cohort assembly excluding prevalent cases and medication users with fixed censoring date: [evidence span withheld — anchor status: not_found]
- [other] Published cohort sizes and case counts: [evidence span withheld — anchor status: not_found]
- [methods] Cox model specification per biomarker version: [evidence span withheld — anchor status: not_found]
- [readme] No prefiltering before remove_technical_variation: "Note that the no prefiltering of samples or columns should be performed prior to running this function: the algorithms used for removing technical variation expect all the data to be present."
- [discussion] Recommendation to compare results on original data: "analysts utilizing the ukbnmr package may wish to compare their key results on the original data to check for and investigate any discrepancies."
