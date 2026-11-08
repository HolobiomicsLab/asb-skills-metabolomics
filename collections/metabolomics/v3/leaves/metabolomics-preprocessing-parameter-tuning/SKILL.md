---
name: metabolomics-preprocessing-parameter-tuning
description: Use when configuring or optimizing PeakMeister preprocessing of targeted
  full-scan MSI-CE-MS mzML files and benchmarking parameter choices against manual
  vendor-software peak picking and integration.
when_to_use_negative:
- Sample matrices other than serum (e.g., urine, saliva, stool) — PeakMeister has
  not been validated for these
- Untargeted/feature-based metabolomic preprocessing where no targeted mass list or
  reference internal standards exist
- Data requiring dynamic time warping or electrophoretic mobility conversion — PeakMeister
  deliberately avoids these alignment approaches
edam_operation: http://edamontology.org/operation_3215
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: PeakMeister
  role: R script that batch-processes mzML files using the tuned parameter set to
    annotate, integrate, and record peak areas
  repo: github.com/PBM-Group/PeakMeister
- name: R
  role: Statistical environment in which the PeakMeister 'code' script is executed
    and in which ksmooth smoothing is applied
- name: mzML
  role: Open-source file format that converted MSI-CE-MS data must be provided in
provenance:
  source_task_ids:
  - task_002
  - task_008
  source_papers:
  - doi: 10.1021/acs.analchem.4c03513
    title: A Software Tool for Rapid and Automated Preprocessing of Large-Scale Serum
      Metabolomic Data by Multisegment Injection-Capillary Electrophoresis-Mass Spectrometry
  source_content_hash: sha256:f024bf850cf5586adf68386354adb2b9ff7596471eb9251c625084e161cc897f
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/PBM-Group/PeakMeister
  tools:
  - MSI-CE-MS
  - PeakMeister
  - R
  - mzML
  edam_operation: http://edamontology.org/operation_3215
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# metabolomics-preprocessing-parameter-tuning

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Tuning the PeakMeister 'Mass List and Parameters.xlsx' preprocessing parameters (extraction windows, migration/annotation windows, noise and SNR filters, smoothing kernel) for automated targeted MSI-CE-MS peak annotation and integration in R.

## When to use

Use when configuring or optimizing PeakMeister preprocessing of targeted full-scan MSI-CE-MS mzML files and benchmarking parameter choices against manual vendor-software peak picking and integration.

## When NOT to use

- Sample matrices other than serum (e.g., urine, saliva, stool) — PeakMeister has not been validated for these
- Untargeted/feature-based metabolomic preprocessing where no targeted mass list or reference internal standards exist
- Data requiring dynamic time warping or electrophoretic mobility conversion — PeakMeister deliberately avoids these alignment approaches

## Inputs

- Open-source mzML files of multiplexed MSI-CE-MS runs placed in the 'mzML Files' folder
- Targeted analyte mass list (names, m/z values) with per-analyte migration times taken from a single run
- Reference internal-standard list (panel of 11 internal standards with class 'Reference')
- 'Mass List and Parameters.xlsx' parameter template (Sheets 1-3)

## Outputs

- Table of migration indexes / relative migration times used for annotation
- Copy of the preprocessing parameters used
- Extracted ion electropherogram (EIE) plots for reviewing annotation and integration
- Table of peak migration times or expected peak positions
- Peak-area table with peaks below snr.threshold recorded as '<LOD'

## How to apply

Populate the 'Mass List and Parameters.xlsx' template: Sheet 1 lists unique analyte names, target m/z, and extraction.window.ppm (a minimum mass window of ~30-35 ppm is required depending on analyte mass and instrument), plus minimum.peak.width.seconds to filter noise peaks, migration.window.seconds defining the +/- search window around predicted migration time, peak.space.tolerance.percent for how much inter-peak spacing may deviate from the median before reanalysis, snr.threshold so low-signal peaks are recorded as '<LOD', and ksmooth smoothing.kernal/smoothing.strength. Sheet 2 marks reference internal standards with class 'Reference' plus min.mt.min/max.mt.min cutoffs (reference peaks must be reliable and the most intense in their EIE); Sheet 3 sets number.of.injections and lock-mass correction parameters (ref.mass.one, ref.mass.two, ref.mass.window.ppm, ref.mass.counts). Take all peak migration times from a single run, then execute the R script 'code' in the project to batch-process mzML files. Validate tuned parameters by comparing PeakMeister annotations and peak areas against manual vendor-software preprocessing, targeting >99.9% median annotation accuracy and ~-2.1% mean integration bias.

## Related tools

- **PeakMeister** (R script that batch-processes mzML files using the tuned parameter set to annotate, integrate, and record peak areas) — github.com/PBM-Group/PeakMeister
- **R** (Statistical environment in which the PeakMeister 'code' script is executed and in which ksmooth smoothing is applied)
- **mzML** (Open-source file format that converted MSI-CE-MS data must be provided in)

## Examples

```
Use the project file to open the R script titled "code" and execute the script to begin pre-processing your data
```

## Evaluation signals

- Median peak-annotation accuracy >99.9% versus manual vendor-software preprocessing
- Mean peak-integration bias of approximately -2.1% for 47 serum metabolites
- Median error rate of ~0.05% for peak picking (14 metabolites with perfect 0% error)
- Intermediate precision consistent with the reported median CV = 16.0%
- Saved EIE plots confirm correct annotation/integration and peaks below snr.threshold are recorded as '<LOD'

## Limitations

- Not validated for sample matrices other than serum (e.g., urine, saliva, stool)
- Late migrating serum metabolites lacking two flanking or a stable-isotope internal standard are prone to peak misannotation
- Low abundance metabolites with signals near the method detection limit are prone to peak misannotation
- Extraction.window.ppm minimum of ~30-35 ppm is instrument-dependent and must be adjusted to the mass spectrometer used
- No user-friendly software interface yet; requires editing the Excel template and running the R script

## Evidence
