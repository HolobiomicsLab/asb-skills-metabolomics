---
name: robust-linear-regression-residual-backtransformation
description: Use when raw UK Biobank NMR metabolomics field data shows technical variation
  attributable to sample degradation time, shipping plate well position, or spectrometer
  drift, and corrected biomarker concentrations are needed before genetic or epidemiological
  analysis.
when_to_use_negative:
- Data has been prefiltered (samples or biomarker columns removed) before calling
  remove_technical_variation() — the README states no prefiltering should be performed
  because the algorithms expect all data present
- The goal is to filter or exclude QC-flagged samples or outlier plates — the package
  explicitly performs no such filtering, and sensitivity analysis to inclusion/exclusion
  of flagged samples is left to the analyst
- Working with derived/composite biomarkers or ratios — adjustment is applied only
  to the 107 non-derived biomarkers, and re-deriving composites post-adjustment is
  a separate step (recompute_derived_biomarkers())
edam_operation: http://edamontology.org/operation_3658
edam_topics:
- http://edamontology.org/topic_3577
- http://edamontology.org/topic_2229
tools:
- name: ukbnmr
  role: Provides remove_technical_variation() implementing the sequential robust regression
    and residual back-transformation on the 107 non-derived NMR biomarkers
  repo: https://github.com/sritchie73/ukbnmr
- name: MASS
  role: Supplies the robust linear regression used to regress each log-biomarker on
    the technical covariates
- name: data.table
  role: fread() reads the exported field data and fwrite() writes the corrected nmr_biomarker_data.csv
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41597-023-01949-y
    title: Quality control and removal of technical variation of NMR metabolic biomarker
      data in ~120,000 UK Biobank participants
  source_content_hash: sha256:c75605ffeeea78d5c6d2e0677a1949cc3990211c6cdcd748823622cbc9405873
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/sritchie73/ukbnmr
  tools:
  - MASS
  - R
  - data.table
  - ukbnmr
  edam_operation: http://edamontology.org/operation_3658
  edam_topics:
  - http://edamontology.org/topic_3577
  - http://edamontology.org/topic_2229
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# Robust linear regression residual back-transformation for NMR biomarker QC

## Summary

Sequentially regresses log-transformed UK Biobank NMR biomarker concentrations on technical covariates (sample preparation-to-measurement time, 96-well plate row, plate column, and per-spectrometer measurement-date drift) using MASS robust linear regression, then transforms residuals back to absolute concentrations. This is the core computational step of the ukbnmr remove_technical_variation() QC procedure.

## When to use

Use when raw UK Biobank NMR metabolomics field data shows technical variation attributable to sample degradation time, shipping plate well position, or spectrometer drift, and corrected biomarker concentrations are needed before genetic or epidemiological analysis.

## When NOT to use

- Data has been prefiltered (samples or biomarker columns removed) before calling remove_technical_variation() — the README states no prefiltering should be performed because the algorithms expect all data present
- The goal is to filter or exclude QC-flagged samples or outlier plates — the package explicitly performs no such filtering, and sensitivity analysis to inclusion/exclusion of flagged samples is left to the analyst
- Working with derived/composite biomarkers or ratios — adjustment is applied only to the 107 non-derived biomarkers, and re-deriving composites post-adjustment is a separate step (recompute_derived_biomarkers())

## Inputs

- Raw exported UK Biobank phenotype field data (CSV/TSV saved by the RAP Table Exporter, columns named p<field_id>_i<visit_index>_a<repeat_index>), loaded via data.table::fread
- ukbnmr::test_data bundled data.frame for testing the procedure

## Outputs

- processed$biomarkers: data.frame with 'eid', 'visit_index' identifier columns plus one column per non-derived NMR biomarker, corrected concentrations written to nmr_biomarker_data.csv
- processed$log_offset: per-biomarker log offset values (nmr_biomarker_log_offset.csv)
- Regression residuals implicitly re-expressed as absolute concentrations after re-adding the log-scale mean and removing the offset

## How to apply

Load the exported UK Biobank phenotype field data with data.table::fread (or use the bundled ukbnmr test_data), then call ukbnmr::remove_technical_variation(), which filters to the 107 non-derived biomarkers and log-transforms absolute concentrations with a small offset (half the minimum non-zero value) for biomarkers containing zeros. Each log-biomarker is then sequentially regressed with MASS robust linear regression on: log sample preparation-to-measurement time, plate row (A-H), plate column (1-12), and finally measurement-date bins (10 bins per each of the six spectrometers) to capture within-spectrometer drift. Regression residuals are transformed back to absolute concentrations by re-adding the original log-scale mean and removing the log offset. Write the corrected data.frame processed$biomarkers to nmr_biomarker_data.csv with data.table::fwrite; the function takes over an hour and at least 32 GB RAM on the full dataset, so persist outputs rather than re-running inline.

## Related tools

- **ukbnmr** (Provides remove_technical_variation() implementing the sequential robust regression and residual back-transformation on the 107 non-derived NMR biomarkers) — https://github.com/sritchie73/ukbnmr
- **MASS** (Supplies the robust linear regression used to regress each log-biomarker on the technical covariates)
- **data.table** (fread() reads the exported field data and fwrite() writes the corrected nmr_biomarker_data.csv)

## Examples

```
library(ukbnmr); library(data.table); exported <- fread("exported_ukbiobank_phenotype_data.csv"); processed <- remove_technical_variation(exported); fwrite(processed$biomarkers, "nmr_biomarker_data.csv")
```

## Evaluation signals

- nmr_biomarker_data.csv exists and contains the 'eid' and 'visit_index' identifier columns plus exactly one column per non-derived NMR biomarker
- All returned concentration values are non-negative after residual rescaling back to absolute concentrations
- Rows of the output are uniquely identifiable by the combination of 'eid' and 'visit_index' (baseline visit_index == 0, first repeat assessment visit_index == 1)
- Regression sequence respected the documented covariate order: log sample time, then plate row (A-H), then plate column (1-12), then 10 measurement-date bins within each of the six spectrometers
- Improved reproducibility of blind duplicate samples (CV% decreasing, R2 increasing) with overall correlation structure and associations with key biological covariates (sex, BMI, lipid-lowering medication) unchanged

## Limitations

- The procedure relies on regressing out technical covariates from metadata labels rather than calibration to quality control samples, so other sources of technical variation may remain uncaptured
- Regressing on these covariates may remove biological variation of interest, and assessing whether biological variation was removed is challenging for UK Biobank
- Adjustment can introduce new unwanted variation into ratios, percentages, or composite biomarkers, so the 107 non-derived biomarkers are recommended with re-derivation of the rest post-adjustment
- No filtering or treatment of outlier samples is performed by the package; outlier shipping plate samples are identified and set to missing but analysts are advised to run sensitivity analyses on QC-flagged samples

## Evidence

- [methods] Sequential robust regression covariate order: "Each biomarker is adjusted for the time between sample preparation and sample measurement (hours) on a log scale."
- [methods] Plate and drift adjustments: "Each biomarker is adjusted for remaining systematic differences between columns (1-12) on the 96-well shipment plates."
- [methods] Within-spectrometer drift binning: "Each biomarker is adjusted for drift over time within each of the six spectrometers."
- [methods] Robust regression tooling: "library(MASS) # Robust linear regression"
- [readme] No sample prefiltering allowed: "Note that the no prefiltering of samples or columns should be performed prior to running this function: the algorithms used for removing technical variation expect all the data to be present."
- [discussion] Metadata-based regression caveat: "it relies on regressing out technical covariates from metadata labels, rather than calibration to quality control samples."
- [discussion] Reproducibility improvement evidence: "our procedure improved the reproducibility of biomarker measurements amongst blind duplicate samples; with the coefficient of variation (CV%) decreasing and coefficient of determination (R2)"
