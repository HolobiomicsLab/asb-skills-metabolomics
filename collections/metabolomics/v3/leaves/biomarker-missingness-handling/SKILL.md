---
name: biomarker-missingness-handling
description: Use when post-QC, log-scaled UK Biobank NMR biomarker concentrations
  still contain plate-level shifts attributable to unexplained technical causes, and
  outlier shipment plates must be detected and their samples set to missing.
when_to_use_negative:
- 'No prefiltering of samples or columns should be performed before remove_technical_variation():
  the algorithms expect all the data to be present, so plate masking must come last,
  not first'
- 'Handling of individual outlier samples or biomarker measurement QC flags — the
  ukbnmr package performs no filtering or treatment of outlier samples; sample-level
  QC decisions (UK Biobank fields #23651–23655, #23700–23948) are a separate sensitivity-analysis
  concern'
- 'Derived biomarkers and ratios: only the 107 non-derived biomarkers are processed;
  composites and ratios must be recomputed afterwards with recompute_derived_biomarkers()'
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3577
- http://edamontology.org/topic_0203
tools:
- name: ukbnmr
  role: Provides remove_technical_variation(), whose final step identifies shipment
    plates that are outliers of non-biological origin and sets their samples to missing,
    returning the flagged plate table
  repo: https://github.com/sritchie73/ukbnmr
- name: R
  role: Analysis environment in which the plate-median normal modelling and +/-3.3744
    SD flagging are implemented and run
- name: data.table
  role: Used for fast reading and writing of the biomarker CSVs (fread/fwrite), including
    outlier_plate_info.csv
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41597-023-01949-y
    title: Quality control and removal of technical variation of NMR metabolic biomarker
      data in ~120,000 UK Biobank participants
  source_content_hash: sha256:77eb61c028a76ee2755ac2a07cf7f372f5d835a4614bab95c9a04f81b7d1f32d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/sritchie73/ukbnmr
  tools:
  - R
  - data.table
  - ukbnmr
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3577
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# biomarker-missingness-handling

## Summary

Identifies shipment plates that are outliers of non-biological origin in UK Biobank NMR biomarker data and sets their samples to missing, as the final step of the ukbnmr technical-variation removal pipeline. This prevents unexplained plate-level technical artefacts from contaminating downstream analyses.

## When to use

Use when post-QC, log-scaled UK Biobank NMR biomarker concentrations still contain plate-level shifts attributable to unexplained technical causes, and outlier shipment plates must be detected and their samples set to missing.

## When NOT to use

- No prefiltering of samples or columns should be performed before remove_technical_variation(): the algorithms expect all the data to be present, so plate masking must come last, not first
- Handling of individual outlier samples or biomarker measurement QC flags — the ukbnmr package performs no filtering or treatment of outlier samples; sample-level QC decisions (UK Biobank fields #23651–23655, #23700–23948) are a separate sensitivity-analysis concern
- Derived biomarkers and ratios: only the 107 non-derived biomarkers are processed; composites and ratios must be recomputed afterwards with recompute_derived_biomarkers()

## Inputs

- Post-QC log-scale NMR biomarker concentration table (107 non-derived biomarkers) as CSV, readable by data.table::fread
- Shipment plate identifiers for each of the 1,352 96-well plates mapping samples to plates

## Outputs

- Biomarker matrix with samples on flagged outlier plates set to missing for the affected biomarker(s)
- outlier_plate_info.csv table of flagged plates per biomarker (via processed$outlier_plate_detection)
- Per-biomarker summary of number/percentage of plates removed

## How to apply

Load the post-QC log-scale concentrations and plate identifiers with data.table::fread, then compute the median concentration of each of the 107 non-derived biomarkers within each of the 1,352 shipment plates. For each biomarker, model the vector of plate medians as a normal distribution (estimate mean and standard deviation), and flag any plate whose median lies beyond +/-3.3744 standard deviations of the fitted normal as an outlier of non-biological origin. Set the samples on flagged plates to missing for the affected biomarker(s) only — this is a per-biomarker, plate-level masking, not whole-sample deletion. Summarise plates removed per biomarker and validate against the reported aggregates: median of 9 plates (0.66% of plates/samples) removed per biomarker, with a maximum of 20 plates (1.5%) for albumin and phosphoglycerides. In the ukbnmr package this step is implemented inside remove_technical_variation() and the flagged plate table is written to outlier_plate_info.csv.

## Related tools

- **ukbnmr** (Provides remove_technical_variation(), whose final step identifies shipment plates that are outliers of non-biological origin and sets their samples to missing, returning the flagged plate table) — https://github.com/sritchie73/ukbnmr
- **R** (Analysis environment in which the plate-median normal modelling and +/-3.3744 SD flagging are implemented and run)
- **data.table** (Used for fast reading and writing of the biomarker CSVs (fread/fwrite), including outlier_plate_info.csv)

## Examples

```
library(ukbnmr); library(data.table); processed <- remove_technical_variation(exported); fwrite(processed$outlier_plate_detection, file="outlier_plate_info.csv")
```

## Evaluation signals

- Per-biomarker plate-removal counts reproduce the published aggregates: median of 9 plates (0.66% of plates/samples) and a maximum of 20 plates (1.5%) for albumin and phosphoglycerides
- Flagged plate table is written to outlier_plate_info.csv and contains plate IDs flagged beyond +/-3.3744 SD of the fitted plate-median normal distribution
- Only the 107 non-derived biomarkers are processed; derived biomarkers and ratios are absent from the masking output
- Samples on flagged plates are set to missing (NA) rather than deleted, preserving the original sample rows

## Limitations

- Outlier plates are 'unexplained outliers of non-biological origin'; a small number of such plates exist across a swathe of biomarkers but the underlying cause is unknown
- Setting plate samples to missing reduces available data (median 0.66%, up to 1.5% of samples per biomarker) rather than correcting the values
- The UK Biobank NMR metabolomics data release analysed covered 121,695 participants / 1,352 plates; later full ~500,000-participant releases use updated algorithm versions, so thresholds may differ
- Setting samples missing is irreversible within the pipeline; sensitivity analysis to inclusion/exclusion of affected samples and measurements is recommended

## Evidence

- [methods] Outlier plates were detected from plate-level median concentrations and their samples set to missing; across biomarkers this removed a median of 9 plates (0.66% of plates/samples): "Samples belonging to shipment plates that are outliers of non-biological origin are identified and set to missing."
- [readme] Flagged plate table written via the ukbnmr package: "fwrite(processed$outlier_plate_detection, file="path/to/outlier_plate_info.csv")"
- [discussion] No sample-level outlier filtering is performed by the package: "no filtering or treatment of outlier samples is performed by the ukbnmr R package when removing technical variation."
- [discussion] Plate outliers are of unexplained, non-biological cause: "a small number of plates are unexplained outliers of non-biological origin across a swathe of biomarkers."
- [readme] All data must be present before running the removal algorithm: "no prefiltering of samples or columns should be performed prior to running this function: the algorithms used for removing technical variation expect all the data to be present."
