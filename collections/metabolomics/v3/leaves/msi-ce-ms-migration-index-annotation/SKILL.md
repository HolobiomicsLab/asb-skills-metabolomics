---
name: msi-ce-ms-migration-index-annotation
description: Use when pre-processing targeted full-scan MSI-CE-MS mzML files with
  high migration-time variability, where peak annotation must be automated across
  multiplexed injections and thousands of serum samples.
when_to_use_negative:
- The sample matrix is not serum (e.g., urine, saliva, stool) — PeakMeister has not
  been validated for matrices other than serum
- Data are from serial-injection isocratic LC-MS rather than MSI-CE-MS — extension
  not yet validated
- Late migrating serum metabolites lacking two flanking or a stable-isotope internal
  standard, or low-abundance metabolites near the method detection limit — these are
  prone to peak misannotation
edam_operation: http://edamontology.org/operation_3694
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: PeakMeister
  role: R software tool that computes migration time indexes from reference internal
    standards and performs automated peak annotation, integration, and sample position
    assignment on mzML MSI-CE-MS data
  repo: https://github.com/PBM-Group/PeakMeister
- name: R
  role: Statistical environment in which PeakMeister is executed; ksmooth from the
    stats package performs EIE smoothing
- name: MSI-CE-MS
  role: High-throughput separation platform producing the multiplexed electropherogram
    data being annotated (<4 min/sample via serial injections)
- name: mzML
  role: Open-source file format that all PeakMeister pre-processing is directly performed
    on
provenance:
  source_task_ids:
  - task_002
  - task_008
  source_papers:
  - doi: 10.1021/acs.analchem.4c03513
    title: A Software Tool for Rapid and Automated Preprocessing of Large-Scale Serum
      Metabolomic Data by Multisegment Injection-Capillary Electrophoresis-Mass Spectrometry
  source_content_hash: sha256:150c3a0b1fa6e49dae4820b4fc82fdd770a62fa5aed0921d5032ff435d07d199
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
  edam_operation: http://edamontology.org/operation_3694
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8125
n_claims: 16
---

# msi-ce-ms-migration-index-annotation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Annotate and integrate peaks in multiplexed MSI-CE-MS metabolomic data using migration time indexes (MTI) computed from a panel of 11 internal standards, implemented in the PeakMeister R tool, instead of dynamic time warping or electrophoretic mobility conversion.

## When to use

Use when pre-processing targeted full-scan MSI-CE-MS mzML files with high migration-time variability, where peak annotation must be automated across multiplexed injections and thousands of serum samples.

## When NOT to use

- The sample matrix is not serum (e.g., urine, saliva, stool) — PeakMeister has not been validated for matrices other than serum
- Data are from serial-injection isocratic LC-MS rather than MSI-CE-MS — extension not yet validated
- Late migrating serum metabolites lacking two flanking or a stable-isotope internal standard, or low-abundance metabolites near the method detection limit — these are prone to peak misannotation

## Inputs

- Open-source mzML data files placed in the 'mzML Files' folder
- 'Mass List and Parameters.xlsx' template (Sheet 1: mass list; Sheet 2: Reference internal standards; Sheet 3: parameters)
- Peak migration times for all analytes taken from a single run
- Panel of 11 internal standards (class 'Reference') for MTI computation

## Outputs

- Table of migration indexes or relative migration times used for analyte annotation
- Copy of the parameters used to process the data
- Plots of each extracted ion electropherogram (EIE) for checking annotation and integration
- Table of migration times of peaks or expected peak positions
- Table of peak areas
- '<LOD' records for peaks failing the snr.threshold

## How to apply

Convert vendor data to open-source mzML files and place them in the 'mzML Files' folder, then fill the 'Mass List and Parameters.xlsx' template: Sheet 1 with unique analyte names, target m/z values, extraction.window.ppm (minimum ~30-35 ppm), migration.window.seconds, minimum.peak.width.seconds, peak.space.tolerance.percent, snr.threshold, and ksmooth kernel/bandwidth; Sheet 2 with class 'Reference' internal standards plus min.mt.min/max.mt.min cutoffs; Sheet 3 with number.of.injections and lock-mass parameters (ref.mass.one, ref.mass.two, ref.mass.window.ppm, ref.mass.counts). PeakMeister computes migration time indexes from the 11 internal standards to predict analyte migration times in each multiplexed run, without dynamic time warping or effective mobility conversion. Peaks are annotated at the predicted migration time +/- migration.window.seconds, confirmed or adjusted using inter-peak spacing consistency (peak.space.tolerance.percent), and flagged for overlaps via interference.comigration.threshold; peaks below snr.threshold are recorded as '<LOD'. Open the project's R script titled 'code' and execute it to batch-process the mzML files.

## Related tools

- **PeakMeister** (R software tool that computes migration time indexes from reference internal standards and performs automated peak annotation, integration, and sample position assignment on mzML MSI-CE-MS data) — https://github.com/PBM-Group/PeakMeister
- **R** (Statistical environment in which PeakMeister is executed; ksmooth from the stats package performs EIE smoothing)
- **MSI-CE-MS** (High-throughput separation platform producing the multiplexed electropherogram data being annotated (<4 min/sample via serial injections))
- **mzML** (Open-source file format that all PeakMeister pre-processing is directly performed on)

## Examples

```
Convert data to mzML into 'mzML Files', fill 'Mass List and Parameters.xlsx', then open the project's R script titled 'code' in R and execute it: Rscript code.R
```

## Evaluation signals

- Median peak annotation accuracy >99.9% compared against manual vendor-software preprocessing (14 of 47 serum metabolites had perfect 0% picking error; median error rate 0.05%)
- Mean peak integration bias of -2.1% for 47 serum metabolites relative to manual processing
- Intermediate precision with median CV = 16.0% consistent with reported benchmarks
- Review of saved EIE plots confirms correct annotation and integration of each extracted ion electropherogram
- Mutual agreement when quantifying 16 plasma metabolites from NIST SRM-1950 (mean bias = -1.3%)

## Limitations

- PeakMeister has not been validated for sample matrices other than serum (e.g., urine, saliva, stool)
- Certain late migrating serum metabolites lacking two flanking or a stable-isotope internal standard were prone to peak misannotation
- Low abundance metabolites with signals near the method detection limit were prone to peak misannotation
- A more user-friendly software interface and 'real-time' data preprocessing output capability are not yet available (future work)
- Extraction.window.ppm minimum of ~30-35 ppm is analyte-mass and mass-spectrometer dependent

## Evidence
