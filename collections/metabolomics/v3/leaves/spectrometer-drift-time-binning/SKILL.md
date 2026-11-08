---
name: spectrometer-drift-time-binning
description: Use when UK Biobank NMR biomarker concentrations still show within-spectrometer
  drift over time after correction for sample degradation time, plate row (A-H), and
  plate column (1-12), and must be adjusted via date-binned robust regression.
when_to_use_negative:
- The exported field data has been prefiltered to a subset of samples or biomarkers
  — the README states no prefiltering should be performed prior to remove_technical_variation()
  because the algorithms expect all data to be present
- You only need the sample-degradation, plate-row, or plate-column corrections — this
  skill covers the final drift step applied after those sequential adjustments
- Working with derived/composite biomarkers — adjustment is performed only on the
  107 biomarkers that cannot be derived; composites are recomputed afterwards
tools:
- name: ukbnmr
  role: Provides remove_technical_variation(), which implements the sequential regression
    including within-spectrometer drift adjustment via date-bin grouping
  repo: sritchie73/ukbnmr
- name: MASS
  role: Supplies robust linear regression used to fit the date-bin drift adjustment
- name: data.table
  role: fread()/fwrite() used to load exported field data and write the corrected
    nmr_biomarker_data.csv
- name: R
  role: Runtime environment for the ukbnmr QC workflow
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41597-023-01949-y
    title: Quality control and removal of technical variation of NMR metabolic biomarker
      data in ~120,000 UK Biobank participants
  source_content_hash: sha256:1b85b93d327505541b2a120157526dc0e992880cc51eb8267eeb676f3c4e5a74
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
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# spectrometer-drift-time-binning

## Summary

The final regression step of the ukbnmr remove_technical_variation() QC pipeline: log-scale biomarker residuals are regressed on measurement-date bins (10 bins per each of the six Nightingale spectrometers) using robust linear regression, removing within-spectrometer drift over time.

## When to use

Use when UK Biobank NMR biomarker concentrations still show within-spectrometer drift over time after correction for sample degradation time, plate row (A-H), and plate column (1-12), and must be adjusted via date-binned robust regression.

## When NOT to use

- The exported field data has been prefiltered to a subset of samples or biomarkers — the README states no prefiltering should be performed prior to remove_technical_variation() because the algorithms expect all data to be present
- You only need the sample-degradation, plate-row, or plate-column corrections — this skill covers the final drift step applied after those sequential adjustments
- Working with derived/composite biomarkers — adjustment is performed only on the 107 biomarkers that cannot be derived; composites are recomputed afterwards

## Inputs

- Raw UK Biobank exported phenotype field data (CSV/TSV from Table Exporter or ukbconv), read with data.table::fread
- Log-scale residuals of the 107 non-derived NMR biomarkers after adjustment for sample preparation-to-measurement time, plate row (A-H), and plate column (1-12)
- Measurement-date and spectrometer metadata for each 96-well shipment plate

## Outputs

- Within-spectrometer drift-adjusted log-scale biomarker residuals, back-transformed to absolute concentrations
- processed$biomarkers data.frame (eid, visit_index, and one column per non-derived NMR biomarker), written to nmr_biomarker_data.csv

## How to apply

After ukbnmr's remove_technical_variation() has filtered to the 107 non-derived biomarkers, log-transformed concentrations with a small offset (half the minimum non-zero value) for biomarkers containing zeros, and sequentially regressed each log-biomarker on log sample preparation-to-measurement time, plate row, and plate column, this step regresses the residuals on measurement-date bins: samples are grouped into 10 bins within each of the six spectrometers by the date the majority of samples on their respective 96-well plates were measured. Robust linear regression from the MASS package is used because date-bin effects can be driven by outliers. The resulting residuals are transformed back to absolute concentrations (re-adding the original log-scale mean and removing the log offset) before being written to nmr_biomarker_data.csv. Order matters: this drift adjustment is applied last, using residuals from the earlier adjustments.

## Related tools

- **ukbnmr** (Provides remove_technical_variation(), which implements the sequential regression including within-spectrometer drift adjustment via date-bin grouping) — sritchie73/ukbnmr
- **MASS** (Supplies robust linear regression used to fit the date-bin drift adjustment)
- **data.table** (fread()/fwrite() used to load exported field data and write the corrected nmr_biomarker_data.csv)
- **R** (Runtime environment for the ukbnmr QC workflow)

## Examples

```
library(ukbnmr); exported <- fread("exported_ukbiobank_phenotype_data.csv"); processed <- remove_technical_variation(exported)
```

## Evaluation signals

- Output nmr_biomarker_data.csv contains the 'eid' and 'visit_index' identifier columns plus one column per non-derived NMR biomarker (107 total)
- All returned concentration values are non-negative after residual back-transformation
- Drift adjustment used exactly 10 measurement-date bins within each of the six spectrometers, and was fitted after (not before) the time, row, and column adjustments
- Reproducibility of blind duplicate samples improves after correction (CV% decreasing, R2 increasing), per the article's QC evaluation

## Limitations

- The pipeline relies on regressing out technical covariates from metadata labels rather than calibration to quality control samples, so other sources of technical variation may not be captured by the covariates examined
- Regressing on these technical covariates may remove biological variation of interest, and assessing whether QC removed biological variation is challenging for a dataset as large as UK Biobank
- Algorithm versions differ: version 2 and the default version 3 made changes to earlier steps relative to version 1 described in Ritchie et al. 2023, so exact behavior depends on the package version used
- The function takes a little over an hour to run and requires at least 32 GB of RAM

## Evidence

- [abstract] Drift step is one of the identified technical variation sources: "These included sample preparation time, shipping plate well, spectrometer batch effects, drift over time within spectrometer, and outlier shipping plates."
- [methods] Article methods description of the drift adjustment step: "Each biomarker is adjusted for drift over time within each of the six spectrometers."
- [readme] README detail on 10-bin grouping by plate measurement date: "samples are grouped into 10 bins, within each spectrometer, by the date the majority of samples on their respective 96-well plates were measured."
- [readme] Back-transformation of residuals after sequential adjustments: "Regression residuals after the sequential adjustments are transformed back to absolute concentrations."
