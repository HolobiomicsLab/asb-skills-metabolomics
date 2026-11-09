---
name: nmr-biomarker-hazard-ratio-estimation
description: Use when UK Biobank NMR biomarker concentrations (original or ukbnmr
  remove_technical_variation output) must be tested for association with incident
  disease endpoints via per-SD Cox models adjusted for age and sex.
when_to_use_negative:
- The data have not yet passed through remove_technical_variation and the goal is
  QC itself — no prefiltering or downstream modelling should precede that function
- The analysis targets genetic associations (GWAS) rather than epidemiological incident-disease
  Cox models
- The endpoint is a continuous trait rather than incident CAD/stroke requiring survival
  analysis
edam_operation: null
edam_topics:
- http://edamontology.org/topic_3335
- http://edamontology.org/topic_2269
tools:
- name: ukbnmr
  role: Supplies extract_biomarkers() to load the original NMR biomarker concentrations
    and remove_technical_variation() to produce the QC-corrected concentrations whose
    associations are compared
  repo: https://github.com/sritchie73/ukbnmr
- name: data.table
  role: Used in R for fast reading and writing of the biomarker tables and the final
    hazard ratio results with fread() and fwrite()
- name: R
  role: Language in which the Cox proportional hazards survival models per biomarker
    are fit
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41597-023-01949-y
    title: Quality control and removal of technical variation of NMR metabolic biomarker
      data in ~120,000 UK Biobank participants
  source_content_hash: sha256:46c77b3e62824173c7e1403e409916c36b26a337b0f767802eba103fdcff6345
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/sritchie73/ukbnmr
  tools:
  - R
  - data.table
  - ukbnmr
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3335
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# nmr-biomarker-hazard-ratio-estimation

## Summary

A skill for running epidemiological biomarker association scans: per-biomarker Cox proportional hazards models over the 249 UK Biobank NMR metabolic biomarkers for incident coronary artery disease and stroke, computed on both original and ukbnmr QC-corrected concentrations to quantify how removing technical variation changes hazard ratios.

## When to use

Use when UK Biobank NMR biomarker concentrations (original or ukbnmr remove_technical_variation output) must be tested for association with incident disease endpoints via per-SD Cox models adjusted for age and sex.

## When NOT to use

- The data have not yet passed through remove_technical_variation and the goal is QC itself — no prefiltering or downstream modelling should precede that function
- The analysis targets genetic associations (GWAS) rather than epidemiological incident-disease Cox models
- The endpoint is a continuous trait rather than incident CAD/stroke requiring survival analysis

## Inputs

- Original UK Biobank NMR biomarker data extracted with ukbnmr::extract_biomarkers (data.frame, one row per eid/visit_index)
- Post-QC NMR biomarker data output by ukbnmr::remove_technical_variation (processed$biomarkers data.frame)
- Incident CAD and stroke endpoints with follow-up time derived from UK Biobank hospital and death records
- Age, sex, and lipid-lowering medication covariates

## Outputs

- Table of hazard ratios, 95% confidence intervals, and P-values for every biomarker-endpoint-version combination (written with data.table::fwrite)
- Ranked association lists per endpoint and data version identifying the strongest biomarker-disease associations

## How to apply

Load the original NMR biomarker data (ukbnmr::extract_biomarkers) and the post-QC data from ukbnmr::remove_technical_variation into R with data.table. Assemble the analytic cohort for each endpoint by excluding prevalent CAD or stroke cases and users of lipid-lowering medication, attaching age and sex as covariates, and truncating follow-up at 1 February 2020 (yielding N=95,790 CAD samples with 2,717 incident cases and N=95,623 stroke samples with 1,366 cases). For each of the 249 biomarkers plus derived ratios, fit a Cox proportional hazards model per SD of concentration adjusted for age and sex, separately for original and post-QC versions, using survival analysis in R. Collect hazard ratios, 95% confidence intervals, and P-values for every biomarker-endpoint-version combination into a single table written with data.table::fwrite, then rank associations within each endpoint and version to identify the strongest hits. The rationale is that the QC procedure increases signal for epidemiological studies, so running both versions side-by-side demonstrates the impact of removing technical variation (e.g. albumin strongest post-QC for stroke vs. percentage of LDL lipids composed of cholesterol, HR 0.89, 95% CI 0.86-0.91, P = 2e-15).

## Related tools

- **ukbnmr** (Supplies extract_biomarkers() to load the original NMR biomarker concentrations and remove_technical_variation() to produce the QC-corrected concentrations whose associations are compared) — https://github.com/sritchie73/ukbnmr
- **data.table** (Used in R for fast reading and writing of the biomarker tables and the final hazard ratio results with fread() and fwrite())
- **R** (Language in which the Cox proportional hazards survival models per biomarker are fit)

## Examples

```
library(ukbnmr); processed <- remove_technical_variation(exported); fwrite(processed$biomarkers, file="path/to/nmr_biomarker_data.csv")
```

## Evaluation signals

- The output table contains results for all biomarker-endpoint-version combinations (249+ biomarkers × CAD/stroke × original/post-QC)
- Post-QC stroke ranking reproduces albumin as the strongest association after QC
- Percentage of LDL lipids composed of cholesterol reproduces as the strongest stroke association in the expected scan (HR 0.89, 95% CI 0.86-0.91, P = 2e-15)
- Cohort sizes match the expected N=95,790 CAD samples (2,717 incident cases) and N=95,623 stroke samples (1,366 cases) after exclusions and follow-up truncation at 1 February 2020

## Limitations

- Regression on technical covariates during QC may remove biological variation of interest, which is challenging to assess even in deeply phenotyped data like UK Biobank
- Biomarker adjustment can introduce new unwanted variation into ratios, percentages, or composite biomarkers, so post-QC derived measures need care
- Many biomarkers have extreme outliers and extremely non-normal distributions requiring careful treatment in statistical modelling, and strong inter-correlations complicate multiple testing and multivariable modelling

## Evidence

- [other] After removal of technical variation, the percentage of LDL lipids composed of cholesterol was the biomarker with the strongest association with incident stroke: [evidence span withheld — anchor status: not_found]
- [readme] README remove_technical_variation: "The `remove_technical_variation()` function removes additional technical variation present in the UK Biobank NMR data"
- [methods] Article methods: derived biomarker filtering: "First biomarker data is filtered to the 107 biomarkers that cannot be derived from any combination of other biomarkers."
- [abstract] Article abstract: QC increases signal: "and demonstrate that it increases signal for genetic and epidemiological studies of the NMR metabolic biomarker data in UK Biobank"
- [readme] README: extract_biomarkers output structure: "Each row of the extracted `data.frame` corresponds to a single observation for a UK Biobank participant at either baseline assessment (2006-2010)"
