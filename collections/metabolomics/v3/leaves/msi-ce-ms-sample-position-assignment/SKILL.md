---
name: msi-ce-ms-sample-position-assignment
description: Use when pre-processing multiplexed MSI-CE-MS runs (multiple serial injections
  per run) and each detected peak must be mapped back to the injected sample position
  it originated from, with migration times that drift batch-to-batch.
when_to_use_negative:
- Untargeted CE-MS feature tables or non-multiplexed single-injection runs, where
  serial-segment spacing is absent and feature-based alignment tools are more appropriate
- Data not convertible to open-source mzML files (PeakMeister pre-processing is performed
  directly on mzML)
- Matrices other than serum (urine, saliva, stool) — PeakMeister is not yet validated
  for these
edam_operation: http://edamontology.org/operation_3694
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0092
tools:
- name: PeakMeister
  role: R software performing MTI-based migration time prediction, peak annotation/integration,
    and sample position (1-13) assignment for multiplexed MSI-CE-MS data
  repo: github.com/PBM-Group/PeakMeister
- name: R
  role: Statistical environment in which the PeakMeister pre-processing script ('code')
    is executed
  repo: https://github.com/PBM-Group/PeakMeister
- name: mzML
  role: Open-source input file format on which all PeakMeister pre-processing is directly
    performed
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1021/acs.analchem.4c03513
    title: A Software Tool for Rapid and Automated Preprocessing of Large-Scale Serum
      Metabolomic Data by Multisegment Injection-Capillary Electrophoresis-Mass Spectrometry
  source_content_hash: sha256:c5c086ec61ed9784dacfdc7ea8460db8b21b40e372e9b8a7a41ba071c067d5e8
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/PBM-Group/PeakMeister
  tools:
  - PeakMeister
  - R
  - mzML
  edam_operation: http://edamontology.org/operation_3694
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6923076923076923
n_claims: 13
---

# msi-ce-ms-sample-position-assignment

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Assigns sample positions (1-13) within each multiplexed MSI-CE-MS analytical run after peak annotation and integration, using migration time indices (MTI) from 11 internal standards and the consistent spacing between serially injected segments.

## When to use

Use when pre-processing multiplexed MSI-CE-MS runs (multiple serial injections per run) and each detected peak must be mapped back to the injected sample position it originated from, with migration times that drift batch-to-batch.

## When NOT to use

- Untargeted CE-MS feature tables or non-multiplexed single-injection runs, where serial-segment spacing is absent and feature-based alignment tools are more appropriate
- Data not convertible to open-source mzML files (PeakMeister pre-processing is performed directly on mzML)
- Matrices other than serum (urine, saliva, stool) — PeakMeister is not yet validated for these

## Inputs

- mzML files of multiplexed MSI-CE-MS runs placed in the 'mzML Files' folder
- 'Mass List and Parameters.xlsx' template with analyte names, m/z values, extraction.window.ppm, migration.window.seconds, peak.space.tolerance.percent, snr.threshold, and migration times from one reference run
- Reference Mass List sheet defining the 11 internal standards (class='Reference') with min.mt.min / max.mt.min cutoffs
- Parameters sheet: number.of.injections, lock-mass settings (ref.mass.one, ref.mass.two, ref.mass.window.ppm, ref.mass.counts), apply.mass.correction, apply.smoothing

## Outputs

- Sample position assignment (positions 1-13) for each analyte within every multiplexed run
- Table of migration times (or expected peak positions) for annotated peaks
- Table of peak areas
- Table of migration time indices / relative migration times used for annotation
- Copy of the processing parameters
- Per-analyte extracted ion electropherogram plots for annotation/integration QC

## How to apply

Extract extracted ion electropherograms (EIEs) for all targeted m/z values from mzML files using the extraction.window.ppm window from the 'Mass List and Parameters.xlsx' template. Annotate the 11 reference internal standards with the landmark algorithm (min.mt.min / max.mt.min cutoffs, minimum.peak.width.seconds filter), verify each reference peak is the most intense reliable signal in its EIE, and compute MTIs from their migration times to predict each analyte's MT from its two flanking standards, searching within +/- migration.window.seconds. Confirm and adjust annotations using the consistency of spaces between serially injected segments: flag peaks deviating beyond peak.space.tolerance.percent from the median spacing for reanalysis. Integrate peak areas, apply snr.threshold to record '<LOD', flag 'Interfered' peaks via interference.comigration.threshold, record 'NPD' when no peak is found, then assign positions 1-13 within each run and write the migration time table, peak area table, and per-analyte EIE plots.

## Related tools

- **PeakMeister** (R software performing MTI-based migration time prediction, peak annotation/integration, and sample position (1-13) assignment for multiplexed MSI-CE-MS data) — github.com/PBM-Group/PeakMeister
- **R** (Statistical environment in which the PeakMeister pre-processing script ('code') is executed) — https://github.com/PBM-Group/PeakMeister
- **mzML** (Open-source input file format on which all PeakMeister pre-processing is directly performed)

## Examples

```
Rscript code.R  # run from the PeakMeister project after filling 'Mass List and Parameters.xlsx' and placing mzML files in 'mzML Files'
```

## Evaluation signals

- Peak annotation accuracy vs manually preprocessed reference should reach median >99.9%
- Peak integration bias vs manual processing should show mean bias of -2.1%
- Recorded '<LOD', 'Interfered', and 'NPD' flags align with snr.threshold, interference.comigration.threshold, and absence of detected peaks respectively
- All annotated peaks fall within the median serial-segment spacing +/- peak.space.tolerance.percent (or were flagged and reanalyzed)
- Each reference standard peak is the most intense reliable signal in its EIE and passes min.mt.min / max.mt.min cutoffs

## Limitations

- Late migrating serum metabolites lacking two flanking or a stable-isotope internal standard were prone to peak misannotation
- Low abundance metabolites with signals near the method detection limit were prone to peak misannotation
- Minimum mass extraction window of ~30-35 ppm is typically required; the ideal extraction.window.ppm depends on the mass spectrometer used
- No user-friendly software interface yet; real-time (as opposed to retrospective) pre-processing output is still under development

## Evidence
