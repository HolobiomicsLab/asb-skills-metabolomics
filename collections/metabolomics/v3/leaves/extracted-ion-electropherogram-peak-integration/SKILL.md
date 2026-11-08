---
name: extracted-ion-electropherogram-peak-integration
description: Use when targeted full-scan MSI-CE-MS data in mzML format must be peak-picked
  and integrated across many multiplexed runs with high migration time variability,
  and manual preprocessing would be slow or inconsistent.
when_to_use_negative:
- Data are not multiplexed MSI-CE-MS runs — e.g. data needing dynamic time warping
  or electrophoretic mobility conversion is what PeakMeister explicitly avoids and
  other tools may suit better
- Untargeted molecular-feature discovery where no targeted mass list of analyte m/z
  values exists — PeakMeister requires a targeted mass list in the 'Mass List and
  Parameters.xlsx' template
- Non-serum matrices such as urine, saliva, or stool — PeakMeister has not yet been
  validated for matrices other than serum
edam_operation: http://edamontology.org/operation_3215
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: PeakMeister
  role: R-based software that extracts EIEs from mzML files, predicts analyte migration
    times via MTIs from 11 internal standards, and integrates peaks automatically
  repo: https://github.com/PBM-Group/PeakMeister
- name: R
  role: Statistical environment in which the PeakMeister script titled 'code' is opened
    and executed
- name: mzML
  role: Open-source input file format to which all vendor data must be converted before
    preprocessing
provenance:
  source_task_ids:
  - task_004
  - task_005
  - task_002
  source_papers:
  - doi: 10.1021/acs.analchem.4c03513
    title: A Software Tool for Rapid and Automated Preprocessing of Large-Scale Serum
      Metabolomic Data by Multisegment Injection-Capillary Electrophoresis-Mass Spectrometry
  source_content_hash: sha256:164ca1d9f4846373a2e3cf3682a72f9f5c2fb51dfa7527f48d6b1c99b0d4eb80
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
  - NIST SRM-1950
  - PeakMeister
  - R
  - mzML
  edam_operation: http://edamontology.org/operation_3215
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8947368421052632
n_claims: 19
---

# Extracted-ion electropherogram peak integration

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Automated extraction, annotation, and integration of analyte peaks from multiplexed MSI-CE-MS mzML runs using PeakMeister's migration time index (MTI) algorithm, producing peak area tables, migration time tables, and EIE plots as a validated replacement for manual vendor-software preprocessing.

## When to use

Use when targeted full-scan MSI-CE-MS data in mzML format must be peak-picked and integrated across many multiplexed runs with high migration time variability, and manual preprocessing would be slow or inconsistent.

## When NOT to use

- Data are not multiplexed MSI-CE-MS runs — e.g. data needing dynamic time warping or electrophoretic mobility conversion is what PeakMeister explicitly avoids and other tools may suit better
- Untargeted molecular-feature discovery where no targeted mass list of analyte m/z values exists — PeakMeister requires a targeted mass list in the 'Mass List and Parameters.xlsx' template
- Non-serum matrices such as urine, saliva, or stool — PeakMeister has not yet been validated for matrices other than serum

## Inputs

- Targeted full-scan mzML files from MSI-CE-MS runs (placed in 'mzML Files' folder)
- 'Mass List and Parameters.xlsx' template with analyte names, m/z values, extraction windows, and processing parameters
- Peak migration times of analytes and 11 'Reference' internal standards, all taken from a single run

## Outputs

- Table of peak areas per analyte per run
- Table of migration times of peaks or expected peak positions
- Table of migration indexes / relative migration times used for annotation
- Copy of the parameters used to process the data
- Plots of each extracted ion electropherogram (EIE) for annotation/integration QC
- Sample position assignments (1-13) within each multiplexed run

## How to apply

Convert all acquisitions to open-source mzML files in the 'mzML Files' folder and fill the 'Mass List and Parameters.xlsx' template with analyte names, m/z values, and an extraction.window.ppm of at least ~30-35 ppm. PeakMeister extracts extracted ion electropherograms (EIEs), then predicts each analyte's migration time from migration time indexes computed from a panel of 11 'Reference' internal standards (filtered by min.mt.min/max.mt.min cutoffs), searching for peaks at the expected time +/- migration.window.seconds — no dynamic time warping or electrophoretic mobility conversion is used. Annotations are confirmed or adjusted via inter-peak spacing consistency (peak.space.tolerance.percent, median spacing with tolerance-triggered reanalysis), noise filtered by minimum.peak.width.seconds, and signals below snr.threshold recorded as '<LOD'; overlapping peaks are flagged via interference and interference.comigration.threshold, and absent peaks as 'NPD'. Smoothing with the ksmooth kernel (smoothing.kernal/smoothing.strength) and optional lock-mass correction (ref.mass.one/two, ref.mass.window.ppm, ref.mass.counts) precede integration, and peaks are integrated at annotated positions with sample positions (1-13) assigned within each multiplexed run. Always inspect the saved EIE plots to confirm correct annotation and integration before accepting the peak area table.

## Related tools

- **PeakMeister** (R-based software that extracts EIEs from mzML files, predicts analyte migration times via MTIs from 11 internal standards, and integrates peaks automatically) — https://github.com/PBM-Group/PeakMeister
- **R** (Statistical environment in which the PeakMeister script titled 'code' is opened and executed)
- **mzML** (Open-source input file format to which all vendor data must be converted before preprocessing)

## Examples

```
Rscript code  # open the PeakMeister project's R script titled 'code' after filling 'Mass List and Parameters.xlsx' and placing mzML files in the 'mzML Files' folder
```

## Evaluation signals

- Median peak annotation accuracy >99.9% versus manual preprocessing by an experienced analyst
- Mean peak integration bias of -2.1% against manual vendor-software processing of 47 serum metabolites
- Intermediate precision consistent with reported median CV = 16.0%
- EIE plots show peaks annotated at predicted migration time +/- migration.window.seconds with no misassignments flagged by peak.space.tolerance.percent reanalysis
- For NIST SRM-1950 quantification, mean bias of -1.3% with per-metabolite biases within the reported -28% (alanine) to +25% (cystine) range

## Limitations

- Late migrating serum metabolites lacking two flanking or a stable-isotope internal standard were prone to peak misannotation
- Low abundance metabolites with signals near the method detection limit were prone to peak misannotation
- Not yet validated for sample matrices other than serum (e.g., urine, saliva, stool)
- No user-friendly interface or real-time preprocessing output yet available; preprocessing is script-based and retrospective

## Evidence

- [readme] MTI-based MT prediction with spacing check: [evidence span withheld — anchor status: not_found]
