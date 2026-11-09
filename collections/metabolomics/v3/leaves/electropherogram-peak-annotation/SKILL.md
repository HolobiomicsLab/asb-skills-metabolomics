---
name: electropherogram-peak-annotation
description: Use when pre-processing targeted full-scan MSI-CE-MS metabolomic data
  with high migration-time variability (e.g. ~105 s drift), where peak alignment or
  time-warping tools fail on multiplexed runs and mzML files must be annotated against
  a targeted mass list.
when_to_use_negative:
- Input data are untargeted molecular features rather than a targeted mass list —
  PeakMeister is designed for targeted full-scan MSI-CE-MS data
- Sample matrix is not serum — PeakMeister is not yet validated for urine, saliva,
  or stool
- Data are conventional single-injection CE-MS without multiplexed segments — the
  peak-space-based confirmation step assumes equally spaced peaks from serial injections
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: PeakMeister
  role: Open-source R software that computes migration indexes, predicts analyte migration
    times, and automates peak annotation, integration, and sample position assignment
    for MSI-CE-MS data
  repo: https://github.com/PBM-Group/PeakMeister
- name: R
  role: Statistical environment in which PeakMeister runs; its stats package 'ksmooth'
    function performs EIE smoothing
- name: mzML
  role: Open-source file format that raw MSI-CE-MS data must be converted into before
    processing
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/acs.analchem.4c03513
    title: A Software Tool for Rapid and Automated Preprocessing of Large-Scale Serum
      Metabolomic Data by Multisegment Injection-Capillary Electrophoresis-Mass Spectrometry
  source_content_hash: sha256:a811a47637cf8d2fc82dfc6d6602513e56ad0a1dd81ca1c9f1e04168623cb562
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/PBM-Group/PeakMeister
  repo_url: https://github.com/PBM-Group/PeakMeister
  tools:
  - MSI-CE-MS
  - PeakMeister
  - R
  - mzML
  edam_operation: http://edamontology.org/operation_3646
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
  techniques:
  - CE-MS
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9285714285714286
n_claims: 14
---

# electropherogram-peak-annotation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Automated peak annotation and integration of multiplexed MSI-CE-MS extracted ion electropherograms using migration time indices (MTIs) computed from internal-standard landmark peaks, implemented in the open-source R tool PeakMeister.

## When to use

Use when pre-processing targeted full-scan MSI-CE-MS metabolomic data with high migration-time variability (e.g. ~105 s drift), where peak alignment or time-warping tools fail on multiplexed runs and mzML files must be annotated against a targeted mass list.

## When NOT to use

- Input data are untargeted molecular features rather than a targeted mass list — PeakMeister is designed for targeted full-scan MSI-CE-MS data
- Sample matrix is not serum — PeakMeister is not yet validated for urine, saliva, or stool
- Data are conventional single-injection CE-MS without multiplexed segments — the peak-space-based confirmation step assumes equally spaced peaks from serial injections

## Inputs

- Raw MSI-CE-MS data files converted to mzML format (in 'mzML Files' folder)
- Mass List and Parameters.xlsx template: analyte names, m/z values, extraction.window.ppm, migration.window.seconds, snr.threshold, peak.space.tolerance.percent, minimum.peak.width.seconds
- Reference Mass List sheet with internal standards (class = 'Reference') and their reference migration times from a single analytical run
- Parameters sheet settings: number.of.injections, lock-mass values (ref.mass.one, ref.mass.two), ref.mass.window.ppm, ref.mass.counts, apply.mass.correction, apply.smoothing

## Outputs

- Table of migration indexes or relative migration times used for analyte annotation
- Table of peak migration times or expected peak positions
- Table of peak areas
- Plots of each extracted ion electropherogram for checking annotation and integration
- Copy of the parameters used to process the data

## How to apply

Convert raw MSI-CE-MS files to mzML and place them in the 'mzML Files' folder; populate the 'Mass List and Parameters.xlsx' template with analyte names, m/z values, extraction.window.ppm (minimum ~30-35 ppm), and reference migration times all taken from the same run. Mark the 11 internal standards as class = 'Reference' in the Reference Mass List sheet, applying min.mt.min/max.mt.min cutoffs to exclude candidate standard peaks outside thresholds. Execute the 'code' R script: PeakMeister extracts ion electropherograms (optionally smoothed with the stats package 'ksmooth' function), computes migration indexes for each analyte from internal-standard migration times rather than performing alignment or mobility conversion, then searches for analyte peaks at predicted migration time +/- migration.window.seconds. Use peak.space.tolerance.percent to confirm/adjust annotations of the multiplexed segments by exploiting the approximately equal spacing between analytical peaks, and set snr.threshold so weak signals are recorded as '<LOD'. Export the MTI table, predicted/expected migration time table, peak area table, and per-analyte EIE plots for review.

## Related tools

- **PeakMeister** (Open-source R software that computes migration indexes, predicts analyte migration times, and automates peak annotation, integration, and sample position assignment for MSI-CE-MS data) — https://github.com/PBM-Group/PeakMeister
- **R** (Statistical environment in which PeakMeister runs; its stats package 'ksmooth' function performs EIE smoothing)
- **mzML** (Open-source file format that raw MSI-CE-MS data must be converted into before processing)

## Examples

```
Convert raw files to mzML in 'mzML Files', fill 'Mass List and Parameters.xlsx', then open the R project and run: source('code')  # executes PeakMeister pre-processing and writes MTI, peak-time, and peak-area tables
```

## Evaluation signals

- Median absolute error between predicted and observed migration times across metabolites should be ~2.9 s versus ~105 s uncorrected (36-fold improvement)
- Median peak annotation accuracy should exceed 99.9%
- Mean bias of automated peak integration versus manual processing should be near -2.1%
- Intermediate precision (median CV) should be around 16.0% across batches
- Extracted ion electropherogram plots should show peaks annotated at predicted positions with segment spacing within peak.space.tolerance.percent

## Limitations

- Certain late-migrating serum metabolites lacking two flanking or a stable-isotope internal standard were prone to peak misannotation
- Low-abundance metabolites with signals near the method detection limit were prone to peak misannotation
- Reference migration times must all come from the same analytical run; minimum extraction window of ~30-35 ppm is required depending on analyte mass and instrument
- Not yet validated for matrices other than serum (urine, saliva, stool); no user-friendly GUI yet — script-based with an Excel parameter sheet

## Evidence
