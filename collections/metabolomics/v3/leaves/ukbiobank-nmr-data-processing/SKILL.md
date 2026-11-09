---
name: ukbiobank-nmr-data-processing
description: 'Use when processing raw UK Biobank NMR metabolomics field exports (fields
  #23651-23948) to remove plate-, spectrometer-, and time-driven technical variation,
  or to detect non-biological outlier 96-well shipment plates before genetic/epidemiological
  analyses.'
when_to_use_negative:
- Data has been pre-filtered to samples or biomarker columns before adjustment — remove_technical_variation()
  expects all the data to be present and no prefiltering should be performed
- Input is already QC-adjusted NMR biomarker data with technical covariates removed;
  re-running would regress out signal a second time
- Goal is to analyze composite biomarkers/ratios directly — those (61 composites,
  81 ratios) must be recomputed from the 107 adjusted non-derived biomarkers post-adjustment
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: ukbnmr
  role: R package implementing remove_technical_variation(), extract_biomarkers(),
    and outlier plate identification for UK Biobank NMR data
  repo: https://github.com/sritchie73/ukbnmr
- name: data.table
  role: fread()/fwrite() for fast reading of exported field data and writing of nmr_biomarker_data.csv
    and outlier_plate_info.csv
- name: MASS
  role: Robust linear regression used to fit the sequential technical-covariate adjustment
    models
- name: ukbtools
  role: Optional processing of pre-RAP UK Biobank field exports into a format compatible
    with ukbnmr
provenance:
  source_task_ids:
  - task_003
  - task_002
  source_papers:
  - doi: 10.1038/s41597-023-01949-y
    title: Quality control and removal of technical variation of NMR metabolic biomarker
      data in ~120,000 UK Biobank participants
  source_content_hash: sha256:23b45847df2997066ff4ad127bb61a2f548831b1dc5777c87fac96a407a5d984
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/sritchie73/ukbnmr
  tools:
  - MASS
  - R
  - data.table
  - ukbnmr
  - ukbtools
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# ukbiobank-nmr-data-processing

## Summary

End-to-end R workflow using the ukbnmr package to quality-control UK Biobank NMR metabolic biomarker data: extracting exported field data, removing technical variation via sequential robust regressions, and identifying/removing outlier shipping plates of non-biological origin.

## When to use

Use when processing raw UK Biobank NMR metabolomics field exports (fields #23651-23948) to remove plate-, spectrometer-, and time-driven technical variation, or to detect non-biological outlier 96-well shipment plates before genetic/epidemiological analyses.

## When NOT to use

- Data has been pre-filtered to samples or biomarker columns before adjustment — remove_technical_variation() expects all the data to be present and no prefiltering should be performed
- Input is already QC-adjusted NMR biomarker data with technical covariates removed; re-running would regress out signal a second time
- Goal is to analyze composite biomarkers/ratios directly — those (61 composites, 81 ratios) must be recomputed from the 107 adjusted non-derived biomarkers post-adjustment

## Inputs

- UK Biobank exported phenotype field data (CSV/TSV saved by Table Exporter on the RAP, or via ukbconv/ukbtools), containing NMR metabolomics fields #23651-23948
- Post-QC log-scale biomarker concentrations with plate identifiers (for outlier plate detection reconstruction)
- ukbnmr::test_data bundled dataset for testing the pipeline

## Outputs

- nmr_biomarker_data.csv: corrected biomarker data.frame with 'eid' and 'visit_index' columns plus one column per non-derived NMR biomarker
- outlier_plate_info.csv: table of flagged outlier plates per biomarker
- nmr_biomarker_qc_flags.csv, nmr_sample_qc_flags.csv, and nmr_biomarker_log_offset.csv from processed$ lists

## How to apply

In R, load exported UK Biobank field data with data.table::fread and call ukbnmr::remove_technical_variation(), which filters to the 107 non-derived biomarkers, log-transforms absolute concentrations with a small offset (half the minimum non-zero value) for biomarkers containing zeros, then sequentially fits robust linear regressions (MASS::rlm) of each log-biomarker on log sample preparation-to-measurement time, plate row (A-H), plate column (1-12), and measurement-date bins (10 bins per each of six spectrometers). Residuals are transformed back to absolute concentrations by re-adding the log-scale mean and removing the log offset. As the final step, compute per-biomarker median concentrations across each of the 1,352 shipment plates, model the plate medians as normal, and flag plates whose median lies beyond +/-3.3744 standard deviations as outliers of non-biological origin; set samples on flagged plates to missing for the affected biomarker(s). Write processed$biomarkers to nmr_biomarker_data.csv and processed$outlier_plate_detection to outlier_plate_info.csv with data.table::fwrite. Validate by reproducing the reported aggregates: a median of 9 plates (0.66%) removed per biomarker and a maximum of 20 plates (1.5%) for albumin and phosphoglycerides, and confirming all returned concentrations are non-negative.

## Related tools

- **ukbnmr** (R package implementing remove_technical_variation(), extract_biomarkers(), and outlier plate identification for UK Biobank NMR data) — https://github.com/sritchie73/ukbnmr
- **data.table** (fread()/fwrite() for fast reading of exported field data and writing of nmr_biomarker_data.csv and outlier_plate_info.csv)
- **MASS** (Robust linear regression used to fit the sequential technical-covariate adjustment models)
- **ukbtools** (Optional processing of pre-RAP UK Biobank field exports into a format compatible with ukbnmr)

## Examples

```
library(ukbnmr); library(data.table); exported <- fread("exported_ukbiobank_phenotype_data.csv"); processed <- remove_technical_variation(exported); fwrite(processed$biomarkers, "nmr_biomarker_data.csv"); fwrite(processed$outlier_plate_detection, "outlier_plate_info.csv")
```

## Evaluation signals

- Outlier-plate summary reproduces reported statistics: median of 9 plates (0.66%) removed per biomarker, maximum 20 plates (1.5%) for albumin and phosphoglycerides
- nmr_biomarker_data.csv exists with 'eid' and 'visit_index' identifier columns plus one column per non-derived NMR biomarker
- All returned concentration values are non-negative after residual rescaling
- Flagged plate table is written to outlier_plate_info.csv
- Reproducibility of blind duplicate samples improves (CV% decreases, R2 increases) after applying the procedure

## Limitations

- No filtering or treatment of outlier samples is performed by the ukbnmr package when removing technical variation; analysts should consider UK Biobank QC-flag fields in sensitivity analyses
- The procedure relies on regressing out technical covariates from metadata labels rather than calibration to quality control samples, and may remove biological variation of interest; other technical sources are not captured by the covariates examined
- remove_technical_variation() takes a little over an hour to run and requires at least 32 GB of RAM; biomarker adjustment can introduce new unwanted variation into ratios, percentages, or composite biomarkers
- Later algorithm versions (v2, v3) modify steps for expanded data releases covering ~150,000 additional and all ~500,000 participants

## Evidence

- [methods] Each biomarker is adjusted for the time between sample preparation and sample measurement (hours) on a log scale.: "Each biomarker is adjusted for the time between sample preparation and sample measurement (hours) on a log scale."
- [methods] Samples belonging to shipment plates that are outliers of non-biological origin are identified and set to missing.: "Samples belonging to shipment plates that are outliers of non-biological origin are identified and set to missing."
- [discussion] no filtering or treatment of outlier samples is performed by the ukbnmr R package when removing technical variation: "no filtering or treatment of outlier samples is performed by the ukbnmr R package when removing technical variation."
- [readme] remove_technical_variation() takes a little over an hour to run, and requires at least 32 GB of RAM: [evidence span withheld — anchor status: external_doc]
- [readme] the algorithms used for removing technical variation expect all the data to be present: "Note that the no prefiltering of samples or columns should be performed prior to running this function: the algorithms used for removing technical variation expect all the data to be present."
