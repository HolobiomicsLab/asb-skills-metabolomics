---
name: msi-ce-ms-peak-annotation
description: Use when pre-processing targeted full-scan multiplexed MSI-CE-MS mzML
  runs where large migration-time variability makes standard alignment tools unsuitable
  and metabolite peaks must be annotated, integrated, and assigned to serial injection
  positions.
when_to_use_negative:
- Input is untargeted feature-table data or data from non-multiplexed CE-MS runs without
  serially injected segments, since the MTI prediction and peak-space consistency
  checks depend on multiplexed MSI-CE-MS structure
- Metabolites lacking two flanking or stable-isotope internal standards (e.g., certain
  late-migrating serum metabolites) or low-abundance metabolites near the method detection
  limit, which are prone to peak misannotation
- Sample matrices other than serum/plasma (e.g., urine, saliva, stool), for which
  PeakMeister has not yet been validated
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3172
tools:
- name: PeakMeister
  role: R-based software that executes the MTI-based migration time prediction, peak
    annotation, integration, and sample position assignment on mzML files
  repo: github.com/PBM-Group/PeakMeister
- name: R
  role: Statistical environment in which the PeakMeister script ('code') is executed
    for pre-processing
- name: MSI-CE-MS
  role: Multiplexed separation platform (<4 min/sample) whose serial-injection data
    structure the annotation algorithm exploits
provenance:
  source_task_ids:
  - task_004
  - task_005
  source_papers:
  - doi: 10.1021/acs.analchem.4c03513
    title: A Software Tool for Rapid and Automated Preprocessing of Large-Scale Serum
      Metabolomic Data by Multisegment Injection-Capillary Electrophoresis-Mass Spectrometry
  source_content_hash: sha256:e34f791f084f94c7fd7422642f82e9ed1574700b9028ad64a669f3dd0fdac49e
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
  - NIST SRM-1950
  - PeakMeister
  - R
  edam_operation: http://edamontology.org/operation_3646
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# msi-ce-ms-peak-annotation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Automated annotation, integration, and sample-position assignment of metabolite peaks in multiplexed MSI-CE-MS data using PeakMeister's migration time index (MTI) algorithm, which predicts analyte migration times from 11 internal-standard landmark peaks instead of dynamic time warping or mobility transformations.

## When to use

Use when pre-processing targeted full-scan multiplexed MSI-CE-MS mzML runs where large migration-time variability makes standard alignment tools unsuitable and metabolite peaks must be annotated, integrated, and assigned to serial injection positions.

## When NOT to use

- Input is untargeted feature-table data or data from non-multiplexed CE-MS runs without serially injected segments, since the MTI prediction and peak-space consistency checks depend on multiplexed MSI-CE-MS structure
- Metabolites lacking two flanking or stable-isotope internal standards (e.g., certain late-migrating serum metabolites) or low-abundance metabolites near the method detection limit, which are prone to peak misannotation
- Sample matrices other than serum/plasma (e.g., urine, saliva, stool), for which PeakMeister has not yet been validated

## Inputs

- Targeted full-scan MSI-CE-MS mzML files (placed in 'mzML Files' folder)
- 'Mass List and Parameters.xlsx' template with analyte names, m/z values, and preprocessing parameters
- Reference Mass List sheet (class='Reference' internal standards with min.mt.min/max.mt.min cutoffs)
- Parameters sheet (number.of.injections, lock-mass ref.mass.one/two, ref.mass.window.ppm, ref.mass.counts, apply.mass.correction, apply.smoothing)

## Outputs

- Table of migration indexes / relative migration times used to annotate analytes
- Table of peak migration times or expected peak positions
- Table of peak areas
- Plots of each extracted ion electropherogram (EIE) for annotation/integration QC
- Copy of the parameters used to process the data

## How to apply

Convert all acquisition files to mzML, place them in the 'mzML Files' folder, and fill the 'Mass List and Parameters.xlsx' template with analyte names, m/z values, and parameters (extraction.window.ppm of at least ~30-35 ppm, minimum.peak.width.seconds, migration.window.seconds, peak.space.tolerance.percent, snr.threshold, smoothing.kernal/strength). Extract extracted ion electropherograms (EIEs) for all targeted m/z values, then annotate the 11 internal-standard 'Reference' peaks with the landmark algorithm using min.mt.min/max.mt.min cutoffs, requiring each reference peak to be the most intense reliable signal in its EIE. Compute migration time indexes (MTI) from the internal-standard migration times and predict each analyte's MT from its two flanking standards, searching within expected position +/- migration.window.seconds; confirm annotations using the consistency of spaces between serially injected segments, flagging peaks deviating beyond peak.space.tolerance.percent from the median spacing for reanalysis. Integrate peak areas at annotated positions, record signals below snr.threshold as '<LOD', flag 'Interfered' peaks via interference/interference.comigration.threshold, and record 'NPD' where no peak is detected; assign sample positions 1-13 within each multiplexed run and write the migration-time table, peak-area table, and per-analyte EIE plots.

## Related tools

- **PeakMeister** (R-based software that executes the MTI-based migration time prediction, peak annotation, integration, and sample position assignment on mzML files) — github.com/PBM-Group/PeakMeister
- **R** (Statistical environment in which the PeakMeister script ('code') is executed for pre-processing)
- **MSI-CE-MS** (Multiplexed separation platform (<4 min/sample) whose serial-injection data structure the annotation algorithm exploits)

## Examples

```
Rscript code  # open the PeakMeister project file and execute the 'code' R script after populating 'Mass List and Parameters.xlsx' and placing mzML files in 'mzML Files'
```

## Evaluation signals

- Median peak annotation accuracy >99.9% versus manual preprocessing by an experienced analyst
- Mean peak integration bias of -2.1% relative to manual integration
- Median CV = 16.0% for intermediate precision across 3 batches over an eight-month period
- EIE plots confirm each 'Reference' peak is the most intense reliable signal in its EIE and analyte annotations fall within expected positions
- Flagged peaks deviate no more than peak.space.tolerance.percent from median segment spacing after reanalysis

## Limitations

- Late-migrating metabolites lacking two flanking or a stable-isotope internal standard are prone to peak misannotation
- Low-abundance metabolites with signals near the method detection limit are prone to peak misannotation
- Validated for serum/plasma only; not yet validated for urine, saliva, stool, or HILIC-based separations beyond what is mentioned
- No user-friendly interface yet; users must edit the Excel parameter sheets and run the R script

## Evidence
