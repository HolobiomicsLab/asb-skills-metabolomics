---
name: internal-standard-based-migration-time-prediction
description: Use when pre-processing targeted full-scan MSI-CE-MS multiplexed data
  (mzML) where high migration time variability from nonlinear electroosmotic flow
  changes prevents peak alignment, and analyte migration times must be predicted per
  run from internal-standard landmarks.
when_to_use_negative:
- Data is not MSI-CE-MS multiplexed full-scan data (e.g. untargeted LC-MS feature
  tables) — the MTI scheme depends on serially injected segments with consistent inter-peak
  spacing
- 'The matrix is not serum-like: PeakMeister was not yet validated for urine, saliva,
  or stool matrices'
- The analyte lacks two flanking internal standards or a stable-isotope internal standard
  and is late migrating or near the detection limit — these are prone to peak misannotation
edam_operation: http://edamontology.org/operation_3214
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: PeakMeister
  role: R software implementing the MTI algorithm, reference-standard landmark annotation,
    peak-space consistency checks, and sample position assignment (1-13)
  repo: github.com/PBM-Group/PeakMeister
- name: R
  role: Statistical environment in which the 'code' script is executed to batch-process
    the mzML files
- name: MSI-CE-MS
  role: Data-acquisition platform producing the multiplexed electropherograms (<4
    min/sample) that the MTI prediction operates on
provenance:
  source_task_ids:
  - task_002
  - task_005
  source_papers:
  - doi: 10.1021/acs.analchem.4c03513
    title: A Software Tool for Rapid and Automated Preprocessing of Large-Scale Serum
      Metabolomic Data by Multisegment Injection-Capillary Electrophoresis-Mass Spectrometry
  source_content_hash: sha256:761f4e3744b5e853d580c7f3c2e9ce1c66860c6ea481cdc19fcd327b4cb858e0
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
  edam_operation: http://edamontology.org/operation_3214
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# internal-standard-based-migration-time-prediction

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Predicts metabolite migration times in multiplexed MSI-CE-MS runs using migration time indexes (MTI) computed from a panel of 11 internal standards, avoiding dynamic time warping or electrophoretic mobility conversion, so that analyte peaks can be annotated and integrated despite large migration time drift across batches.

## When to use

Use when pre-processing targeted full-scan MSI-CE-MS multiplexed data (mzML) where high migration time variability from nonlinear electroosmotic flow changes prevents peak alignment, and analyte migration times must be predicted per run from internal-standard landmarks.

## When NOT to use

- Data is not MSI-CE-MS multiplexed full-scan data (e.g. untargeted LC-MS feature tables) — the MTI scheme depends on serially injected segments with consistent inter-peak spacing
- The matrix is not serum-like: PeakMeister was not yet validated for urine, saliva, or stool matrices
- The analyte lacks two flanking internal standards or a stable-isotope internal standard and is late migrating or near the detection limit — these are prone to peak misannotation

## Inputs

- Open-source mzML data files placed in the 'mzML Files' folder
- Mass List and Parameters.xlsx template: Sheet 1 analyte m/z list and parameters, Sheet 2 Reference Mass List with 11 'Reference' internal standards and min.mt.min/max.mt.min cutoffs, Sheet 3 lock-mass and smoothing parameters
- Migration times of all analytes and internal standards taken from a single run

## Outputs

- Table of migration indexes (MTIs) or relative migration times used for analyte annotation
- Table of predicted/observed migration times of peaks or expected peak positions
- Table of peak areas per analyte per sample position (1-13 within each multiplexed run)
- Plots of each extracted ion electropherogram (EIE) for annotation/integration review
- Copy of the processing parameters used

## How to apply

Build a Mass List and Parameters.xlsx entry per analyte (name, mz, extraction.window.ppm ~30-35, migration.window.seconds, minimum.peak.width.seconds, snr.threshold, peak.space.tolerance.percent) plus a Reference Mass List sheet with 11 internal standards classed 'Reference' and min.mt.min/max.mt.min cutoffs. Extract EIEs from the mzML files and annotate each reference peak with a separate landmark algorithm that requires reference signals to be reliable and the most intense in their EIE within the MT cutoffs. Compute MTIs from the internal-standard migration times, then predict each analyte's migration time from its two flanking standards and search within expected MT +/- migration.window.seconds. Confirm/adjust annotations using consistency of the serial-injection segment spacing (deviations beyond peak.space.tolerance.percent from the median spacing trigger reanalysis), record signals below snr.threshold as '<LOD' and no-peak positions as 'NPD'. This deliberately replaces DTW or effective-mobility conversion with a computationally simpler index scheme that remains reliable across batches of runs.

## Related tools

- **PeakMeister** (R software implementing the MTI algorithm, reference-standard landmark annotation, peak-space consistency checks, and sample position assignment (1-13)) — github.com/PBM-Group/PeakMeister
- **R** (Statistical environment in which the 'code' script is executed to batch-process the mzML files)
- **MSI-CE-MS** (Data-acquisition platform producing the multiplexed electropherograms (<4 min/sample) that the MTI prediction operates on)

## Examples

```
Rscript code  # open the project's R script 'code' in R with mzML files in 'mzML Files' and the completed Mass List and Parameters.xlsx template
```

## Evaluation signals

- Median peak annotation accuracy >99.9% versus manual vendor-software preprocessing (14 metabolites had perfect 0% error, median error rate 0.05%)
- Mean peak-integration bias of -2.1% for 47 serum metabolites against manual processing
- Intermediate precision median CV = 16.0%; mutual agreement bias of -1.3% for 16 NIST SRM-1950 plasma metabolites
- Reference internal-standard peaks are the most intense reliable signals in their EIEs and fall within min.mt.min/max.mt.min cutoffs
- Inter-peak spacing of serially injected segments deviates less than peak.space.tolerance.percent from the median spacing after annotation

## Limitations

- Late-migrating serum metabolites lacking two flanking or a stable-isotope internal standard are prone to peak misannotation
- Low-abundance metabolites with signals near the method detection limit are prone to misannotation
- Validated only for serum/plasma (ENANI-2019, NIST SRM-1950); not yet validated for urine, saliva, or stool matrices
- Requires a minimum mass extraction window of ~30-35 ppm depending on analyte mass and instrument

## Evidence
