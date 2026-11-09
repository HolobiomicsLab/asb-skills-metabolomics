---
name: plate-median-normal-modelling
description: Use when post-QC, log-transformed UK Biobank NMR biomarker concentrations
  still contain plate-level batch artefacts and shipment plates of non-biological
  origin must be flagged and their samples set to missing.
when_to_use_negative:
- 'No prefiltering of samples or columns should be performed before remove_technical_variation():
  the algorithm expects all data to be present, so do not apply this outlier-plate
  step to a subsetted dataset'
- 'Not for outlier samples flagged by Nightingale measurement/sample QC flags (UK
  Biobank fields #23651-23655 and #23700-23948) — the package performs no filtering
  or treatment of outlier samples beyond plate-level outliers'
- Not for single-participant or small-cohort NMR data lacking the plate structure
  (1,352 shipment plates) needed to fit plate-median distributions
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3577
tools:
- name: ukbnmr
  role: R package implementing remove_technical_variation(), whose final step identifies
    and removes outlier shipment plates and returns processed$outlier_plate_detection
  repo: https://github.com/sritchie73/ukbnmr
- name: R
  role: Language in which the plate-median normal modelling and sample-removal are
    implemented
- name: data.table
  role: Provides fread()/fwrite() for fast reading of post-QC biomarker data and writing
    outlier_plate_info.csv
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41597-023-01949-y
    title: Quality control and removal of technical variation of NMR metabolic biomarker
      data in ~120,000 UK Biobank participants
  source_content_hash: sha256:bc9bcadf7a507f237bd2a3d1cd6b388ea30d843b78a664a9a6ee9c7048ca9585
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/sritchie73/ukbnmr
  tools:
  - R
  - data.table
  - ukbnmr
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3577
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 4
---

# plate-median-normal-modelling

## Summary

Final QC step for UK Biobank NMR biomarker data: detect shipment plates of unexplained, non-biological origin by modelling per-plate median concentrations of the 107 non-derived biomarkers as normal distributions and removing samples from plates whose medians deviate beyond a fixed SD threshold.

## When to use

Use when post-QC, log-transformed UK Biobank NMR biomarker concentrations still contain plate-level batch artefacts and shipment plates of non-biological origin must be flagged and their samples set to missing.

## When NOT to use

- No prefiltering of samples or columns should be performed before remove_technical_variation(): the algorithm expects all data to be present, so do not apply this outlier-plate step to a subsetted dataset
- Not for outlier samples flagged by Nightingale measurement/sample QC flags (UK Biobank fields #23651-23655 and #23700-23948) — the package performs no filtering or treatment of outlier samples beyond plate-level outliers
- Not for single-participant or small-cohort NMR data lacking the plate structure (1,352 shipment plates) needed to fit plate-median distributions

## Inputs

- Post-QC log-transformed NMR biomarker concentrations for the 107 non-derived biomarkers (data.frame / CSV)
- Shipment plate identifiers per sample (e.g. acquires from sample_qc_info / sample processing information)
- Output of prior QC steps in remove_technical_variation() (time, plate row A-H, column 1-12, and within-spectrometer drift adjustments)

## Outputs

- Corrected biomarker data with samples from outlier plates set to missing for the affected biomarker(s)
- Flagged-plate table (processed$outlier_plate_detection) written to outlier_plate_info.csv
- Per-biomarker summary of the number (and percentage) of plates removed

## How to apply

Load the post-QC log-scale biomarker concentrations plus plate identifiers with data.table::fread in R. Compute the median concentration of each of the 107 non-derived biomarkers within each of the 1,352 shipment plates. For each biomarker, model the vector of 1,352 plate medians as a normal distribution, estimate its mean and standard deviation, and flag any plate whose median lies beyond ±3.3744 standard deviations of the fitted distribution. Set all samples on flagged plates to missing for the affected biomarker(s), then summarise plates removed per biomarker. This runs as the last step of the remove_technical_variation() pipeline (algorithm step 8), after adjustment for preparation time, plate row/column effects, and within-spectrometer drift. In the ukbnmr package the flagged-plate table is exported via processed$outlier_plate_detection to outlier_plate_info.csv.

## Related tools

- **ukbnmr** (R package implementing remove_technical_variation(), whose final step identifies and removes outlier shipment plates and returns processed$outlier_plate_detection) — https://github.com/sritchie73/ukbnmr
- **R** (Language in which the plate-median normal modelling and sample-removal are implemented)
- **data.table** (Provides fread()/fwrite() for fast reading of post-QC biomarker data and writing outlier_plate_info.csv)

## Examples

```
library(ukbnmr); processed <- remove_technical_variation(exported); fwrite(processed$outlier_plate_detection, file="outlier_plate_info.csv")
```

## Evaluation signals

- Across the 107 biomarkers, the number of plates removed must reproduce the reported aggregates: median of 9 plates (0.66% of plates/samples) per biomarker
- Maximum plates removed must be 20 plates (1.5%), occurring for albumin and phosphoglycerides
- All samples belonging to any flagged plate must be set to missing for the affected biomarker(s), mirroring the albumin example in Fig. 5
- Flagged-plate table must be written to outlier_plate_info.csv via processed$outlier_plate_detection
- Plate medians must be computed over exactly the 1,352 shipment plates for each of the 107 non-derived biomarkers before normal fitting

## Limitations

- The threshold (±3.3744 SD) was calibrated on the first ~120,000 participants; later algorithm versions (v2, v3) tweaked the pipeline for subsequent data releases covering up to all ~500,000 participants
- Plates are flagged as outliers of 'non-biological origin' but the source of the variation is unexplained ('a small number of plates are unexplained outliers of non-biological origin across a swathe of biomarkers')
- Setting flagged plates to missing discards data; sensitivity analysis to inclusion/exclusion of affected samples is recommended

## Evidence

- [methods] Outlier plate detection sets samples to missing: "Samples belonging to shipment plates that are outliers of non-biological origin are identified and set to missing."
- [readme] README export of flagged plates: "fwrite(processed$outlier_plate_detection, file="path/to/outlier_plate_info.csv")"
- [readme] Algorithm ordering within remove_technical_variation: "7. Regression residuals after the sequential adjustments are transformed back to absolute concentrations."
- [discussion] Unexplained non-biological plate outliers: "a small number of plates are unexplained outliers of non-biological origin across a swathe of biomarkers."
- [readme] No prefiltering allowed: "Note that the no prefiltering of samples or columns should be performed prior to running this function"
