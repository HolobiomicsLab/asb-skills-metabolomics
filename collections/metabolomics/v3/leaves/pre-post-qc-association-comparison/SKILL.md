---
name: pre-post-qc-association-comparison
description: Use when you have both original and ukbnmr::remove_technical_variation-processed
  NMR biomarker concentrations plus incident disease endpoints and need to test whether
  QC changed or strengthened biomarker–disease associations.
when_to_use_negative:
- You only have one version of the biomarker data (either original or post-QC) — the
  comparison requires both the extract_biomarkers and remove_technical_variation outputs
- Your goal is the QC itself rather than evaluating its impact — use ukbnmr::remove_technical_variation
  directly, with no prefiltering of samples or columns beforehand
- The outcome is not time-to-event — Cox models are inappropriate for cross-sectional
  or binary case-control designs
edam_operation: null
edam_topics:
- http://edamontology.org/topic_3360
- http://edamontology.org/topic_3416
tools:
- name: ukbnmr
  role: Supplies extract_biomarkers() to load original NMR concentrations and remove_technical_variation()
    to produce the QC-corrected concentrations compared in the scan
  repo: https://github.com/sritchie73/ukbnmr
- name: data.table
  role: Fast reading of exported UK Biobank field data with fread() and writing of
    the combined association results table with fwrite()
- name: R (survival analysis)
  role: Fits Cox proportional hazards models per SD of concentration adjusted for
    age and sex for each biomarker, endpoint, and data version
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41597-023-01949-y
    title: Quality control and removal of technical variation of NMR metabolic biomarker
      data in ~120,000 UK Biobank participants
  source_content_hash: sha256:81bff3930c57ef8d92571436cb1c882665aa266e9fb691fbd441f3fac0429f6b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/sritchie73/ukbnmr
  tools:
  - R
  - R (survival analysis)
  - data.table
  - ukbnmr
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3360
  - http://edamontology.org/topic_3416
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# pre-post-qc-association-comparison

## Summary

Runs an epidemiological association scan (Cox proportional hazards models for incident CAD and stroke) in parallel on the original and post-QC UK Biobank NMR biomarker data to quantify whether removal of technical variation impacts or improves disease-association signal.

## When to use

Use when you have both original and ukbnmr::remove_technical_variation-processed NMR biomarker concentrations plus incident disease endpoints and need to test whether QC changed or strengthened biomarker–disease associations.

## When NOT to use

- You only have one version of the biomarker data (either original or post-QC) — the comparison requires both the extract_biomarkers and remove_technical_variation outputs
- Your goal is the QC itself rather than evaluating its impact — use ukbnmr::remove_technical_variation directly, with no prefiltering of samples or columns beforehand
- The outcome is not time-to-event — Cox models are inappropriate for cross-sectional or binary case-control designs

## Inputs

- Original UK Biobank NMR biomarker data extracted with ukbnmr::extract_biomarkers (CSV/data.frame, one column per biomarker, keyed by eid and visit_index)
- Post-QC NMR biomarker data produced by ukbnmr::remove_technical_variation
- Incident coronary artery disease and stroke endpoints with follow-up time (truncated at 1 February 2020)
- Prevalent disease status and lipid-lowering medication use for cohort exclusions
- Age and sex covariates

## Outputs

- Single results table (CSV via data.table::fwrite) of hazard ratios, 95% confidence intervals, and P-values for every biomarker-endpoint-version (original vs post-QC) combination
- Ranked association lists per endpoint and data version identifying top biomarker hits

## How to apply

Load the original NMR biomarker data (via ukbnmr::extract_biomarkers) and the post-QC biomarker data from ukbnmr::remove_technical_variation using data.table in R. Assemble the analytic cohort per endpoint by excluding participants with prevalent CAD or stroke and users of lipid-lowering medication, attaching age and sex as covariates, and truncating follow-up at 1 February 2020 (N=95,790 CAD samples with 2,717 incident cases; N=95,623 stroke samples with 1,366 cases). For each of the 249 biomarkers plus derived ratios, fit a Cox proportional hazards model per SD of concentration adjusted for age and sex, separately for original and post-QC versions, and collect hazard ratios, 95% CIs, and P-values into a single table written with data.table::fwrite. Rank associations within each endpoint and version to compare signal, verifying e.g. that the percentage of LDL lipids composed of cholesterol is the strongest post-QC stroke association (HR 0.89, 95% CI 0.86-0.91, P = 2e-15).

## Related tools

- **ukbnmr** (Supplies extract_biomarkers() to load original NMR concentrations and remove_technical_variation() to produce the QC-corrected concentrations compared in the scan) — https://github.com/sritchie73/ukbnmr
- **data.table** (Fast reading of exported UK Biobank field data with fread() and writing of the combined association results table with fwrite())
- **R (survival analysis)** (Fits Cox proportional hazards models per SD of concentration adjusted for age and sex for each biomarker, endpoint, and data version)

## Examples

```
library(ukbnmr); library(data.table); nmr <- fread("nmr_biomarker_data.csv"); qc <- remove_technical_variation(nmr); fwrite(qc$biomarkers, "nmr_post_qc.csv")
```

## Evaluation signals

- Output table contains results for all biomarker-endpoint-version combinations (249 biomarkers plus derived ratios × 2 endpoints × 2 data versions)
- Post-QC stroke ranking reproduces expected top hit: percentage of LDL lipids composed of cholesterol, HR 0.89, 95% CI 0.86-0.91, P = 2e-15
- Albumin is confirmed as the strongest stroke association in the post-QC ranking as stated in the source card validation
- Cohort sizes match expected counts: N=95,790 CAD samples with 2,717 incident cases and N=95,623 stroke samples with 1,366 incident cases
- Follow-up time is truncated at 1 February 2020 for all analysed samples

## Limitations

- Regressing out technical covariates during QC may inadvertently remove biological variation of interest, so pre-vs-post differences should be interpreted cautiously
- Many biomarkers have extreme outliers and extremely non-normal distributions requiring careful treatment in statistical modelling
- Strong inter-correlation among biomarkers impacts choice of multiple testing strategies and challenges multivariable modelling
- Biomarker adjustment can introduce new unwanted variation into ratios, percentages, or composite biomarkers

## Evidence

- [readme] The remove_technical_variation() function removes additional technical variation present in the UK Biobank NMR data: "The `remove_technical_variation()` function removes additional technical variation present in the UK Biobank NMR data"
- [methods] First biomarker data is filtered to the 107 biomarkers that cannot be derived from any combination of other biomarkers.: "First biomarker data is filtered to the 107 biomarkers that cannot be derived from any combination of other biomarkers."
- [discussion] Comparison of key results on original vs QC data is recommended: "analysts utilizing the ukbnmr package may wish to compare their key results on the original data to check for and investigate any discrepancies."
- [abstract] QC procedure increases signal for genetic and epidemiological studies: "and demonstrate that it increases signal for genetic and epidemiological studies of the NMR metabolic biomarker data in UK Biobank"
- [methods] library(data.table) # for fast reading and writing of csv files using fread() and fwrite(): "library(data.table) # for fast reading and writing of csv files using fread() and fwrite()"
