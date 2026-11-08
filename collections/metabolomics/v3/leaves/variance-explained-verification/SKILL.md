---
name: variance-explained-verification
description: Use when you have original (pre-correction) UK Biobank NMR biomarker
  data plus sample QC fields and need to quantify which technical covariates drive
  biomarker variance, e.g. to justify or verify QC correction.
when_to_use_negative:
- When the goal is to correct/remove technical variation rather than quantify it —
  use ukbnmr::remove_technical_variation() instead
- When the input biomarker data has already been QC-corrected by ukbnmr — variance
  explained by technical covariates should be assessed on the original data only
- When the data are not UK Biobank NMR metabolomics fields with the Nightingale sample
  QC structure (plates, spectrometers, degradation time)
edam_topics:
- http://edamontology.org/topic_2269
tools:
- name: ukbnmr
  role: Extracts biomarker concentrations and sample QC flags from the raw exported
    UK Biobank field data into analysis-ready data.frames
  repo: https://github.com/sritchie73/ukbnmr
- name: data.table
  role: Reads the exported field data with fread() and writes the biomarker x covariate
    R-squared table with fwrite()
- name: MASS
  role: Robust linear regression support when fitting biomarker ~ technical covariate
    models
- name: R (stats::lm)
  role: Fits one linear model per biomarker per technical covariate to compute R-squared
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41597-023-01949-y
    title: Quality control and removal of technical variation of NMR metabolic biomarker
      data in ~120,000 UK Biobank participants
  source_content_hash: sha256:759d4c20011eebd1b7961da9bfa29fb8f011e101a5d545581f0a8a5167b71d23
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/sritchie73/ukbnmr
  tools:
  - MASS
  - R
  - R (stats::lm)
  - data.table
  - ukbnmr
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# variance-explained-verification

## Summary

Quantifies how much variance in each of the 249 original UK Biobank NMR biomarker concentrations is explained by recorded technical covariates (plate well, batch, spectrometer, drift, degradation time) via per-covariate linear models, reproducing the Ritchie et al. 2023 variance-explained analysis that motivated the ukbnmr QC procedure.

## When to use

Use when you have original (pre-correction) UK Biobank NMR biomarker data plus sample QC fields and need to quantify which technical covariates drive biomarker variance, e.g. to justify or verify QC correction.

## When NOT to use

- When the goal is to correct/remove technical variation rather than quantify it — use ukbnmr::remove_technical_variation() instead
- When the input biomarker data has already been QC-corrected by ukbnmr — variance explained by technical covariates should be assessed on the original data only
- When the data are not UK Biobank NMR metabolomics fields with the Nightingale sample QC structure (plates, spectrometers, degradation time)

## Inputs

- UK Biobank exported phenotype field data (CSV/TSV from Table Exporter or ukbconv) containing NMR biomarker fields and sample processing fields
- ukbnmr::nmr_info data.frame of biomarker-to-field mappings
- ukbnmr::sample_qc_info data.frame describing sample processing flags (fields #23651-23655 and related)

## Outputs

- variance_explained_r2.csv — tidy biomarker x covariate table of linear model R-squared values
- Summary statistics: median maximum R-squared across biomarkers, count of biomarkers with R-squared >= 5%, and specific R-squared values for alanine (drift) and histidine (degradation time)

## How to apply

Load the exported UK Biobank phenotype field data with data.table::fread and extract biomarker concentrations and sample processing flags using ukbnmr::extract_biomarkers() and ukbnmr::extract_sample_qc_flags(). Assemble a technical covariate matrix per sample from the sample QC fields: 96-well plate row (A-H) and column (1-12), shipment/processing batch, spectrometer of measurement, sample preparation-to-measurement degradation time, and binned measurement timestamps. For each of the 249 biomarkers, fit one linear model (stats::lm; MASS is available for robust regression) per technical covariate and record the model R-squared into a tidy biomarker x covariate table written with data.table::fwrite. Summarize by computing the median of the maximum R-squared across biomarkers, the count of biomarkers with R-squared >= 5%, and the R-squared for alanine against within-spectrometer drift and histidine against degradation time.

## Related tools

- **ukbnmr** (Extracts biomarker concentrations and sample QC flags from the raw exported UK Biobank field data into analysis-ready data.frames) — https://github.com/sritchie73/ukbnmr
- **data.table** (Reads the exported field data with fread() and writes the biomarker x covariate R-squared table with fwrite())
- **MASS** (Robust linear regression support when fitting biomarker ~ technical covariate models)
- **R (stats::lm)** (Fits one linear model per biomarker per technical covariate to compute R-squared)

## Examples

```
library(ukbnmr); exported <- fread("exported_ukbiobank_phenotype_data.csv"); nmr <- extract_biomarkers(exported); qc <- extract_sample_qc_flags(exported)
```

## Evaluation signals

- Median of the maximum R-squared across the 249 biomarkers should be ~1.5% variance explained by the most correlated technical factor
- Count of biomarkers with R-squared >= 5% should be 22 out of 249
- R-squared for alanine against within-spectrometer drift should be ~33%
- R-squared for histidine against sample degradation time should be ~15%
- The R-squared table should have dimensions 249 biomarkers x 6 technical covariates and be a valid tidy CSV readable with fread()

## Limitations

- Other sources of technical variation not captured by the examined covariates (sample preparation time, plate well, spectrometer, drift, plates) may remain
- Regressing out technical covariates can remove biological variation of interest; this skill only quantifies, and does not judge, biological impact
- Filtering to non-derived biomarkers does not mitigate collinearity challenges among the 107 non-derived biomarkers, which complicates multivariable modelling

## Evidence

- [discussion] robust majority of biomarkers: "the vast majority of biomarkers are relatively robust to technical variation, but that a small subset are significantly impacted by inter-spectrometer variation, within-spectrometer drift over time,"
- [readme] extraction functions: "The `extract_sample_qc_flags()` function similarly returns a `data.frame` with one column for each of the [NMR sample processing flags and quality control flags]"
- [methods] package utilities: "This package provides utilities for working with the [UK Biobank NMR metabolomics data]"
- [methods] libraries: "library(data.table) # for fast reading and writing of csv files using fread() and fwrite()"
- [methods] robust regression: "library(MASS) # Robust linear regression"
