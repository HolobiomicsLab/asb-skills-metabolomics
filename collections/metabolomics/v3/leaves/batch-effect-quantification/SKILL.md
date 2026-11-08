---
name: batch-effect-quantification
description: 'Use when raw NMR biomarker data (e.g. UK Biobank fields #23651-23948)
  plus sample processing QC flags are available and you need to rank technical covariates
  by per-biomarker variance explained before deciding on correction steps.'
when_to_use_negative:
- Your goal is to remove technical variation rather than quantify it — use ukbnmr::remove_technical_variation()
  instead of per-covariate R-squared models
- You are working with the post-QC, already-corrected biomarker data — variance explained
  by technical covariates should be assessed on the original data only
- 'You need to treat outlier samples: the ukbnmr package performs no filtering of
  outlier samples, so outlier handling must be designed separately, not inferred from
  this quantification'
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_2269
- http://edamontology.org/topic_0593
tools:
- name: ukbnmr
  role: extract_biomarkers() and extract_sample_qc_flags() pull the 249 NMR biomarker
    concentrations and sample processing/QC flags from the exported UK Biobank field
    data into analysis-ready data.frames
  repo: sritchie73/ukbnmr
- name: data.table
  role: fread() reads the exported field data and fwrite() writes the variance_explained_r2.csv
    output table
- name: MASS
  role: robust linear regression option when fitting biomarker ~ technical covariate
    models
- name: R (stats::lm)
  role: fits one linear model per biomarker per technical covariate and supplies the
    R-squared used as the variance-explained metric
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41597-023-01949-y
    title: Quality control and removal of technical variation of NMR metabolic biomarker
      data in ~120,000 UK Biobank participants
  source_content_hash: sha256:74b16e770911b5e4fd7c907267185b8780f2a3521e2242c13b097ea5bc9e8c0c
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
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_2269
  - http://edamontology.org/topic_0593
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# batch-effect-quantification

## Summary

Quantifies how much variance in each of the 249 original UK Biobank NMR biomarker concentrations is explained by each recorded technical covariate (96-well plate row/column, shipment/processing batch, spectrometer, within-spectrometer drift, sample degradation time, measurement timestamps) by fitting per-covariate linear models and recording R-squared.

## When to use

Use when raw NMR biomarker data (e.g. UK Biobank fields #23651-23948) plus sample processing QC flags are available and you need to rank technical covariates by per-biomarker variance explained before deciding on correction steps.

## When NOT to use

- Your goal is to remove technical variation rather than quantify it — use ukbnmr::remove_technical_variation() instead of per-covariate R-squared models
- You are working with the post-QC, already-corrected biomarker data — variance explained by technical covariates should be assessed on the original data only
- You need to treat outlier samples: the ukbnmr package performs no filtering of outlier samples, so outlier handling must be designed separately, not inferred from this quantification

## Inputs

- Exported UK Biobank phenotype field data (CSV/TSV from Table Exporter or ukbconv) containing NMR biomarker fields #23700-23948 and sample QC fields #23651-23655
- ukbnmr::nmr_info data.frame of biomarker-to-field mappings
- ukbnmr::sample_qc_info data.frame of sample processing field mappings

## Outputs

- Tidy biomarker x technical covariate R-squared table written to variance_explained_r2.csv via data.table::fwrite
- Summary statistics: median of the maximum R-squared across biomarkers, count of biomarkers with R-squared >= 5%, and specific R-squared values for alanine vs within-spectrometer drift and histidine vs degradation time

## How to apply

Load the exported UK Biobank phenotype field data with data.table::fread and extract biomarker concentrations via ukbnmr::extract_biomarkers() and sample processing flags via ukbnmr::extract_sample_qc_flags(). Assemble a technical covariate matrix per sample covering 96-well plate row (A-H) and column (1-12), shipment/processing batch, spectrometer of measurement, sample preparation-to-measurement degradation time, and binned measurement timestamps (samples grouped into 10 bins within each of the six spectrometers by plate measurement date, mirroring the ukbnmr drift bins). For each of the 249 biomarkers, fit one linear model (R stats::lm, optionally MASS robust regression) per technical covariate and record the model R-squared, writing the tidy biomarker x covariate table with data.table::fwrite (e.g. variance_explained_r2.csv). Judge success by reproducing the paper's summary statistics: median ~1.5% variance explained by the most correlated technical factor, 22 of 249 biomarkers with >=5% variance explained, ~33% of alanine variance from within-spectrometer drift, and ~15% of histidine variance from sample degradation time.

## Related tools

- **ukbnmr** (extract_biomarkers() and extract_sample_qc_flags() pull the 249 NMR biomarker concentrations and sample processing/QC flags from the exported UK Biobank field data into analysis-ready data.frames) — sritchie73/ukbnmr
- **data.table** (fread() reads the exported field data and fwrite() writes the variance_explained_r2.csv output table)
- **MASS** (robust linear regression option when fitting biomarker ~ technical covariate models)
- **R (stats::lm)** (fits one linear model per biomarker per technical covariate and supplies the R-squared used as the variance-explained metric)

## Examples

```
nmr <- ukbnmr::extract_biomarkers(fread("exported.csv")); sapply(split(nmr, nmr$spectrometer), function(d) summary(lm(Ala ~ measure_time_bin, data=d))$r.squared)
```

## Evaluation signals

- Median of the maximum per-biomarker R-squared across the 249 biomarkers is ~1.5% (the red dashed 1% line in the paper's variance-explained boxplots)
- Exactly 22 of 249 biomarkers show >= 5% of variance explained by one or more technical factors
- R-squared for alanine regressed on within-spectrometer drift over time is ~33%
- R-squared for histidine regressed on sample degradation time (preparation-to-measurement hours) is ~15%
- Output CSV has one row per biomarker x covariate pair (249 biomarkers) with a numeric r2 column and no missing covariate levels

## Limitations

- There may be other sources of technical variation not captured by the technical covariates examined
- Regressing on technical covariates may remove biological variation of interest, and the analysis relies on metadata regression rather than calibration to quality control samples
- Strong inter-correlations among biomarkers impact multiple testing strategies and challenge multivariable modelling; filtering to the 107 non-derived biomarkers does not mitigate collinearity
- Many biomarkers have extreme outliers and extremely non-normal distributions requiring careful treatment before linear modelling

## Evidence

- [abstract] technical covariate list: "These included sample preparation time, shipping plate well, spectrometer batch effects, drift over time within spectrometer, and outlier shipping plates."
- [discussion] expected finding: "the vast majority of biomarkers are relatively robust to technical variation, but that a small subset are significantly impacted by inter-spectrometer variation, within-spectrometer drift over time,"
- [readme] data extraction: "The `extract_sample_qc_flags()` function similarly returns a `data.frame` with one column for each of the [NMR sample processing flags and quality control flags]"
- [methods] covariate binning: "Each biomarker is adjusted for drift over time within each of the six spectrometers. To do so, samples are grouped into 10
     bins, within each spectrometer, by the date the majority of samples on"
- [methods] regression tooling: "library(MASS) # Robust linear regression"
