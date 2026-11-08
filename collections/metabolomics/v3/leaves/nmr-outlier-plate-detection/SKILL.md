---
name: nmr-outlier-plate-detection
description: Use when processing UK Biobank NMR biomarker data after regression-based
  technical-covariate adjustment, when a small number of 96-well shipment plates show
  unexplained plate-level shifts across a swathe of biomarkers and their samples should
  be set to missing.
when_to_use_negative:
- 'No prefiltering of samples or columns should be performed prior to remove_technical_variation():
  the algorithm expects all data present, so do not apply plate outlier removal to
  hand-filtered subsets'
- Do not use this to remove biologically extreme participant samples — the ukbnmr
  package performs no filtering or treatment of outlier samples, only outlier plates
  of non-biological origin
- Not applicable once the 61 composite biomarkers and 81 ratios have been recomputed
  — outlier detection targets the 107 non-derived biomarkers before re-derivation
tools:
- name: ukbnmr
  role: R package implementing the QC pipeline; outlier plate detection is step 8
    of remove_technical_variation(), returned as processed$outlier_plate_detection
  repo: https://github.com/sritchie73/ukbnmr
- name: data.table
  role: Used via fread()/fwrite() to load the post-QC log-scale biomarker data and
    write the outlier plate table to outlier_plate_info.csv
- name: R
  role: Analysis environment in which plate medians are computed and plate medians
    are modelled as normal distributions
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41597-023-01949-y
    title: Quality control and removal of technical variation of NMR metabolic biomarker
      data in ~120,000 UK Biobank participants
  source_content_hash: sha256:1e1be6519cc4c992491dff4f0ee4a9e4bd98ba07277c0fc6ae9a6708983baa87
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/sritchie73/ukbnmr
  tools:
  - R
  - data.table
  - ukbnmr
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 14
---

# nmr-outlier-plate-detection

## Summary

Final step of the UK Biobank NMR QC pipeline: identify shipment plates whose median biomarker concentrations are outliers of non-biological origin and set the affected samples to missing. This removes residual plate-level technical variation left after row, column, and spectrometer-drift adjustments.

## When to use

Use when processing UK Biobank NMR biomarker data after regression-based technical-covariate adjustment, when a small number of 96-well shipment plates show unexplained plate-level shifts across a swathe of biomarkers and their samples should be set to missing.

## When NOT to use

- No prefiltering of samples or columns should be performed prior to remove_technical_variation(): the algorithm expects all data present, so do not apply plate outlier removal to hand-filtered subsets
- Do not use this to remove biologically extreme participant samples — the ukbnmr package performs no filtering or treatment of outlier samples, only outlier plates of non-biological origin
- Not applicable once the 61 composite biomarkers and 81 ratios have been recomputed — outlier detection targets the 107 non-derived biomarkers before re-derivation

## Inputs

- Post-QC log-transformed NMR biomarker concentration table for the 107 non-derived biomarkers (one row per UK Biobank participant sample)
- Shipment plate identifiers mapping each sample to its 96-well shipment plate (1,352 plates)
- Exported UK Biobank NMR field data (CSV/TSV from Table Exporter) when using the ukbnmr remove_technical_variation() wrapper

## Outputs

- Biomarker concentration table with samples on outlier plates set to missing for the affected biomarker(s)
- Outlier plate table listing flagged plates per biomarker (written to outlier_plate_info.csv)
- Per-biomarker summary of the number of plates removed

## How to apply

Load the post-QC log-scale biomarker concentrations with plate identifiers using data.table::fread in R. For each of the 107 non-derived biomarkers, compute the median concentration within each of the 1,352 shipment plates, then model the vector of 1,352 plate medians as a normal distribution and estimate its mean and standard deviation. Flag any plate whose median lies beyond +/-3.3744 standard deviations of the fitted normal distribution as an outlier of non-biological origin, per biomarker. Set the samples on flagged plates to missing for the affected biomarker(s), then summarize plates removed per biomarker. This step is implemented within the remove_technical_variation() function of the ukbnmr package, whose output includes an outlier plate table to be written to outlier_plate_info.csv. The rationale is that a small number of plates are unexplained outliers of non-biological origin across a swathe of biomarkers and cannot be corrected by covariate regression.

## Related tools

- **ukbnmr** (R package implementing the QC pipeline; outlier plate detection is step 8 of remove_technical_variation(), returned as processed$outlier_plate_detection) — https://github.com/sritchie73/ukbnmr
- **data.table** (Used via fread()/fwrite() to load the post-QC log-scale biomarker data and write the outlier plate table to outlier_plate_info.csv)
- **R** (Analysis environment in which plate medians are computed and plate medians are modelled as normal distributions)

## Examples

```
processed <- remove_technical_variation(exported); fwrite(processed$outlier_plate_detection, file="path/to/outlier_plate_info.csv")
```

## Evaluation signals

- Outlier-plate summary reproduces the reported aggregates: median of 9 plates (0.66% of plates/samples) removed per biomarker across the 107 biomarkers
- Maximum removal per biomarker is 20 plates (1.5%), observed for albumin and phosphoglycerides
- The flagged plate table is written and populated as outlier_plate_info.csv
- Flagged plates per biomarker all have plate medians beyond +/-3.3744 standard deviations of the fitted normal distribution over the 1,352 plate medians
- Post-removal, samples on flagged plates are set to missing only for the biomarker(s) whose plate median triggered the flag, mirroring the albumin example

## Limitations

- The procedure relies on regressing out technical covariates from metadata labels rather than calibration to quality control samples, so other uncharacterized plate-level technical sources may remain
- Regressing on technical covariates may remove biological variation of interest, and assessing whether QC removed biological variation is challenging for UK Biobank
- The +/-3.3744 SD threshold and plate counts were calibrated on the first ~120,000 participant release (1,352 plates, algorithm version 1); later versions were tweaked for expanded releases covering ~150,000 and ~500,000 participants

## Evidence

- [methods] Outlier plate detection step of the QC pipeline: "Samples belonging to shipment plates that are outliers of non-biological origin are identified and set to missing."
- [discussion] Motivation for the step: "a small number of plates are unexplained outliers of non-biological origin across a swathe of biomarkers."
- [readme] Output artifact for this step: "fwrite(processed$outlier_plate_detection, file="path/to/outlier_plate_info.csv")"
- [discussion] No sample-level outlier filtering: "no filtering or treatment of outlier samples is performed by the ukbnmr R package when removing technical variation."
