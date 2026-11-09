---
name: technical-covariate-regression-analysis
description: Use when you have UK Biobank NMR biomarker data (249 biomarkers plus
  sample QC fields) and need to quantify, per biomarker, the percentage of concentration
  variance attributable to each technical covariate to decide which measurements need
  technical-variation removal.
when_to_use_negative:
- The goal is to actually remove technical variation from the 107 non-derived biomarkers
  rather than quantify it — use the ukbnmr remove_technical_variation() robust-regression
  pipeline instead
- The data are already corrected concentrations (output of remove_technical_variation())
  and you want to verify residual technical effects — the regression should be run
  on the original data as reported
- Input is biomarker measurement QC flags rather than concentrations — R-squared regression
  on covariates applies to the concentration matrix, not the flag columns
edam_operation: null
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: ukbnmr
  role: Provides extract_biomarkers() and extract_sample_qc_flags() to pull the 249
    NMR biomarker concentrations and sample processing/technical covariate fields
    from the raw exported field data
  repo: https://github.com/sritchie73/ukbnmr
- name: R
  role: Language in which the per-covariate stats::lm models are fitted and R-squared
    values recorded
- name: data.table
  role: fread() to load the exported field data and fwrite() to write the variance_explained_r2.csv
    table
- name: MASS
  role: Robust linear regression (used in the companion removal procedure whose covariates
    this analysis screens)
provenance:
  source_task_ids:
  - task_001
  - task_002
  source_papers:
  - doi: 10.1038/s41597-023-01949-y
    title: Quality control and removal of technical variation of NMR metabolic biomarker
      data in ~120,000 UK Biobank participants
  source_content_hash: sha256:4affc954fc2d8ebe757009ffb6c1ee639e77be0814e4b5037c5e6ffc6ff56d4e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/sritchie73/ukbnmr
  tools:
  - MASS
  - R
  - data.table
  - ukbnmr
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# technical-covariate-regression-analysis

## Summary

Quantifies how much variance in each of the 249 UK Biobank NMR biomarker concentrations is explained by each recorded technical covariate (plate row/column, shipment batch, spectrometer, degradation time, measurement-date bins) by fitting one linear model per biomarker-covariate pair and recording R-squared, to identify biomarkers most affected by technical variation before applying the ukbnmr removal procedure.

## When to use

Use when you have UK Biobank NMR biomarker data (249 biomarkers plus sample QC fields) and need to quantify, per biomarker, the percentage of concentration variance attributable to each technical covariate to decide which measurements need technical-variation removal.

## When NOT to use

- The goal is to actually remove technical variation from the 107 non-derived biomarkers rather than quantify it — use the ukbnmr remove_technical_variation() robust-regression pipeline instead
- The data are already corrected concentrations (output of remove_technical_variation()) and you want to verify residual technical effects — the regression should be run on the original data as reported
- Input is biomarker measurement QC flags rather than concentrations — R-squared regression on covariates applies to the concentration matrix, not the flag columns

## Inputs

- UK Biobank exported phenotype field data (CSV/TSV from Table Exporter or ukbconv) containing NMR biomarker fields
- Extracted biomarker concentration data.frame (one column per NMR biomarker, keyed by eid and visit_index) from ukbnmr::extract_biomarkers()
- Extracted sample QC/processing flags data.frame from ukbnmr::extract_sample_qc_flags() (plate row A-H, column 1-12, shipment batch, spectrometer, degradation time, measurement timestamps)

## Outputs

- variance_explained_r2.csv: tidy biomarker x technical-covariate R-squared table written with data.table::fwrite
- Summary statistics: median of maximum R-squared across biomarkers, count of biomarkers with R-squared >= 5%, and biomarker-specific R-squared values (alanine vs. spectrometer drift, histidine vs. degradation time)

## How to apply

Load the exported UK Biobank phenotype field data with data.table::fread and extract biomarker concentrations and sample QC fields using ukbnmr::extract_biomarkers() and ukbnmr::extract_sample_qc_flags(). Assemble the technical covariate matrix per sample: 96-well plate row (A-H), plate column (1-12), shipment/processing batch, spectrometer of measurement, sample preparation-to-measurement degradation time, and binned measurement timestamps. For each of the 249 biomarkers, fit one linear model per technical covariate with stats::lm and record the model R-squared, yielding a tidy biomarker x covariate R-squared table. Summarize with the median of the maximum R-squared across biomarkers, the count of biomarkers with R-squared >= 5%, and the specific R-squared for alanine against within-spectrometer drift and histidine against sample degradation time; correct application should reproduce the reported values (median ~1.5%, 22/249 biomarkers >=5%, ~33% alanine drift, ~15% histidine degradation time).

## Related tools

- **ukbnmr** (Provides extract_biomarkers() and extract_sample_qc_flags() to pull the 249 NMR biomarker concentrations and sample processing/technical covariate fields from the raw exported field data) — https://github.com/sritchie73/ukbnmr
- **R** (Language in which the per-covariate stats::lm models are fitted and R-squared values recorded)
- **data.table** (fread() to load the exported field data and fwrite() to write the variance_explained_r2.csv table)
- **MASS** (Robust linear regression (used in the companion removal procedure whose covariates this analysis screens))

## Examples

```
library(ukbnmr); exported <- fread("exported_ukbiobank_phenotype_data.csv"); nmr <- extract_biomarkers(exported); sqc <- extract_sample_qc_flags(exported); r2 <- sapply(nmr$alanine, function(x) summary(lm(x ~ sqc$measurement_drift))$r.squared)
```

## Evaluation signals

- The computed summary statistics reproduce the reported values: median ~1.5% variance explained by the most correlated technical factor, 22 of 249 biomarkers with >=5% variance explained, ~33% for alanine vs. within-spectrometer drift, ~15% for histidine vs. sample degradation time
- The tidy R-squared table covers all 249 biomarkers x every assembled technical covariate and is readable as a valid CSV written by data.table::fwrite
- Variance-explained distributions are consistent with the reported boxplots in which a red dashed line marks 1% variance explained
- Each linear model is a simple one-covariate lm per biomarker (not multivariable), so max R-squared per biomarker identifies the single most correlated technical factor

## Limitations

- This analysis quantifies variance explained by covariates examined; other sources of technical variation are not captured by these technical covariates
- Regressing on technical covariates may remove biological variation of interest, and assessing whether a QC procedure removed biological variation is challenging and potentially intractable for UK Biobank
- The R-squared screen does not itself correct the data; removal of variation requires the separate multi-step robust regression procedure, and no filtering of outlier samples is performed by the ukbnmr package when removing technical variation

## Evidence

- [discussion] Most biomarkers were robust to technical variation, but a small subset are significantly impacted: "the vast majority of biomarkers are relatively robust to technical variation, but that a small subset are significantly impacted by inter-spectrometer variation, within-spectrometer drift over time,"
- [abstract] Variance explained by technical covariates: "These included sample preparation time, shipping plate well, spectrometer batch effects, drift over time within spectrometer, and outlier shipping plates."
- [readme] Extraction functions: "The `extract_sample_qc_flags()` function similarly returns a `data.frame` with one column for each of the [NMR sample processing flags and quality control flags]"
- [methods] Adjustment steps mirror the covariates screened: "Each biomarker is adjusted for drift over time within each of the six spectrometers."
- [methods] Tool stack: "library(data.table) # for fast reading and writing of csv files using fread() and fwrite()"
