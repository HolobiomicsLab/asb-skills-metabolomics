---
name: nmr-biomarker-log-transformation-with-zero-offset
description: Use when preparing the 107 non-derived UK Biobank NMR biomarker concentrations
  for robust regression-based removal of technical variation and some biomarkers contain
  exact 0 values.
when_to_use_negative:
- Biomarker data that are already log-transformed or already processed via remove_technical_variation()
  — applying the offset twice would distort concentrations
- Analysis of derived biomarkers and ratios — the procedure is recommended only for
  the 107 non-derived biomarkers, with composites re-derived afterwards via recompute_derived_biomarkers()
- Datasets with samples or biomarker columns pre-filtered before calling remove_technical_variation(),
  since the algorithms expect all data present
edam_operation: http://edamontology.org/operation_3435
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: ukbnmr
  role: R package implementing remove_technical_variation(), which performs the log
    transformation with offset, exports the per-biomarker log_offset, and back-transforms
    residuals
  repo: https://github.com/sritchie73/ukbnmr
- name: R
  role: Runtime environment for the ukbnmr package and its QC workflow
- name: data.table
  role: fread()/fwrite() for reading exported UK Biobank field data and writing nmr_biomarker_data.csv
    and nmr_biomarker_log_offset.csv
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41597-023-01949-y
    title: Quality control and removal of technical variation of NMR metabolic biomarker
      data in ~120,000 UK Biobank participants
  source_content_hash: sha256:8857af0f8fb6f86cb91a90542960be9b176972ed9b4d2b82f06b676c3bf8d27b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/sritchie73/ukbnmr
  tools:
  - MASS
  - R
  - data.table
  - ukbnmr
  edam_operation: http://edamontology.org/operation_3435
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# nmr-biomarker-log-transformation-with-zero-offset

## Summary

Log-transforms absolute NMR biomarker concentrations from UK Biobank Nightingale measurements, adding a small offset (half the minimum non-zero value) to biomarkers that contain zeros so the log transform is defined prior to sequential regression of technical covariates.

## When to use

Use when preparing the 107 non-derived UK Biobank NMR biomarker concentrations for robust regression-based removal of technical variation and some biomarkers contain exact 0 values.

## When NOT to use

- Biomarker data that are already log-transformed or already processed via remove_technical_variation() — applying the offset twice would distort concentrations
- Analysis of derived biomarkers and ratios — the procedure is recommended only for the 107 non-derived biomarkers, with composites re-derived afterwards via recompute_derived_biomarkers()
- Datasets with samples or biomarker columns pre-filtered before calling remove_technical_variation(), since the algorithms expect all data present

## Inputs

- data.frame of extracted NMR biomarker concentrations (one column per biomarker, rows keyed by eid and visit_index) from ukbnmr::extract_biomarkers()
- Bundled ukbnmr::test_data for trial runs

## Outputs

- Log-transformed biomarker matrix (with offset applied to zero-containing biomarkers) used as input to robust linear regression
- processed$log_offset data.frame recording the offset added per biomarker (written to nmr_biomarker_log_offset.csv)
- Back-transformed absolute concentration data.frame processed$biomarkers written to nmr_biomarker_data.csv

## How to apply

After filtering the extracted UK Biobank NMR field data to the 107 biomarkers that cannot be derived from combinations of others, log transform each biomarker's absolute concentrations. For biomarkers whose value distributions include 0, first add a small offset equal to half the minimum non-zero concentration so the log remains defined; biomarkers without zeros are transformed without offset. This transformation is a prerequisite to the sequential robust linear regressions (MASS rlm) on log sample preparation-to-measurement time, 96-well plate row (A-H), plate column (1-12), and spectrometer measurement-date bins (10 bins per each of six spectrometers). After regression, residuals are transformed back to absolute concentrations by removing the log offset and re-adding the original log-scale mean. The offset used is exported by ukbnmr as processed$log_offset for reproducibility.

## Related tools

- **ukbnmr** (R package implementing remove_technical_variation(), which performs the log transformation with offset, exports the per-biomarker log_offset, and back-transforms residuals) — https://github.com/sritchie73/ukbnmr
- **R** (Runtime environment for the ukbnmr package and its QC workflow)
- **data.table** (fread()/fwrite() for reading exported UK Biobank field data and writing nmr_biomarker_data.csv and nmr_biomarker_log_offset.csv)

## Examples

```
library(ukbnmr); exported <- fread("exported_ukbiobank_phenotype_data.csv"); processed <- remove_technical_variation(exported); fwrite(processed$biomarkers, "nmr_biomarker_data.csv"); fwrite(processed$log_offset, "nmr_biomarker_log_offset.csv")
```

## Evaluation signals

- nmr_biomarker_data.csv exists with 'eid' and 'visit_index' identifier columns plus one column per non-derived NMR biomarker
- All returned concentration values in processed$biomarkers are non-negative after residual rescaling and back-transformation
- processed$log_offset records a non-zero offset only for biomarkers whose raw concentrations include zeros; zero-free biomarkers have no offset
- Absolute concentrations are comparable in scale to the original raw measurements after the inverse transform (offset removed, log-scale mean re-added)

## Limitations

- Biomarker adjustment can introduce new unwanted variation into ratios, percentages, or composite biomarkers, so composites should be recomputed from the 107 adjusted non-derived biomarkers
- Regressing on technical covariates may remove biological variation of interest, and other sources of technical variation are not captured by the covariates examined
- Many biomarkers have extreme outliers and extremely non-normal distributions requiring careful treatment in downstream statistical modelling

## Evidence

- [other] log-transform absolute concentrations with a small offset (half the minimum non-zero value) applied to biomarkers containing zeros: [evidence span withheld — anchor status: not_found]
- [methods] Absolute concentrations are log transformed, with a small offset applied to biomarkers with concentrations of 0.: "Absolute concentrations are log transformed, with a small offset applied to biomarkers with concentrations of 0."
- [methods] filtering to the 107 biomarkers that cannot be derived: "First biomarker data is filtered to the 107 biomarkers that cannot be derived from any combination of other biomarkers."
- [readme] Regression residuals after the sequential adjustments are transformed back to absolute concentrations.: "Regression residuals after the sequential adjustments are transformed back to absolute concentrations."
- [discussion] no filtering or treatment of outlier samples is performed by the ukbnmr R package: "no filtering or treatment of outlier samples is performed by the ukbnmr R package when removing technical variation."
